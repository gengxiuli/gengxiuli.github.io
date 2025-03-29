---
layout: post
title:  Quagga zebra between kernel interface method
date:   2025-03-29
category: networking
tags:   quagga zebra kernel netlink route socket
---

Quagga开源路由协议的核心功能，就是维护内核中的路由表，这是通过用户态的路由协议和内核交互实现的。那么用户态程序如何与内核进行交互，本文就来探究一下。

在 Quagga 1.2.4的 configure.ac 文件中有下面这一段代码：

```shell
dnl ------------------------------------
dnl Determine routing get and set method
dnl ------------------------------------
AC_MSG_CHECKING(zebra between kernel interface method)
if test x"$opsys" = x"gnu-linux"; then
  AC_MSG_RESULT(netlink)
  RT_METHOD=rt_netlink.o
  AC_DEFINE(HAVE_NETLINK,,netlink)
  netlink=yes
else
  AC_MSG_RESULT(Route socket)
  KERNEL_METHOD="kernel_socket.o"
  RT_METHOD="rt_socket.o"
fi
AC_SUBST(RT_METHOD)
AC_SUBST(KERNEL_METHOD)
AM_CONDITIONAL([HAVE_NETLINK], [test "x$netlink" = "xyes"])
```
通过判断操作系统的类型（opsys），当是 gnu-linux 的时候，RT_METHOD =  rt_netlink.o，否则 RT_METHOD =  kernel_socket.o 且 KERNEL_METHOD = rt_socket.o。RT_METHOD 和 KERNEL_METHOD 在 zebra 目录中的 Makefile.am 引用，用来赋值最终需要编译的目标文件变量，具体如下：

```shell
ipforward = @IPFORWARD@
if_method = @IF_METHOD@
rt_method = @RT_METHOD@
rtread_method = @RTREAD_METHOD@
kernel_method = @KERNEL_METHOD@
ioctl_method = @IOCTL_METHOD@

otherobj = $(ioctl_method) $(ipforward) $(if_method) \
	$(rt_method) $(rtread_method) $(kernel_method)
```

接下来我们看一下具体的代码。在 zebra_rib.c 中 rib_process 最后通过 rib_update_kernel 将路由表安装到内核中， rib_update_kernel 最终通过 kernel_route_rib 将路由实际下发到内核路由表。而kernel_route_rib 有两种不同的实现，分别对应前面提到的 rt_netlink.c 以及 rt_socket.c 和 kernel_socket.c 文件。

首先让我们看看 rt_netlink.c中 的实现。

```c
int
kernel_route_rib (struct prefix *p, struct rib *old, struct rib *new)
{
  if (!old && new)
    return netlink_route_multipath (RTM_NEWROUTE, p, new);
  if (old && !new)
    return netlink_route_multipath (RTM_DELROUTE, p, old);

   /* Replace, can be done atomically if metric does not change;
    * netlink uses [prefix, tos, priority] to identify prefix.
    * Now metric is not sent to kernel, so we can just do atomic replace. */
  return netlink_route_multipath (RTM_NEWROUTE, p, new);
}
```

根据 old 和 new 的状态决定是添加 (RTM_NEWROUTE) 还是删除 (RTM_DELROUTE) 路由, 或者是更新路由。不论哪种情况最终都调用 netlink_route_multipath。因为netlink_route_multipath的实现比较长，我们只选取比较重要的进行分析。


```c
/* Routing table change via netlink interface. */
static int
netlink_route_multipath (int cmd, struct prefix *p, struct rib *rib)
{
  int bytelen;
  struct sockaddr_nl snl;
  struct nexthop *nexthop = NULL, *tnexthop;
  int recursing;
  int nexthop_num;
  int discard;
  int family = PREFIX_FAMILY(p);
  const char *routedesc;

  /*请求消息格式, 其中nlmsghdr结构体成员n封装netlink消息头，rtmsg结构体成员r封装路由消息投，buf封装路由信息净荷*/
  struct
  {
    struct nlmsghdr n;
    struct rtmsg r;
    char buf[NL_PKT_BUF_SIZE];
  } req;

  struct zebra_vrf *zvrf = vrf_info_lookup (rib->vrf_id);

  memset (&req, 0, sizeof req - NL_PKT_BUF_SIZE);

  bytelen = (family == AF_INET ? 4 : 16);

  req.n.nlmsg_len = NLMSG_LENGTH (sizeof (struct rtmsg));
  req.n.nlmsg_flags = NLM_F_CREATE | NLM_F_REPLACE | NLM_F_REQUEST;
  req.n.nlmsg_type = cmd;
  req.r.rtm_family = family;
  req.r.rtm_table = rib->table;
  req.r.rtm_dst_len = p->prefixlen;
  req.r.rtm_protocol = RTPROT_ZEBRA;
  req.r.rtm_scope = RT_SCOPE_LINK;

  /*此处省略根据传入的cmd, p 和 rib 参数决定下发的路由消息处理流程*/

  /* Destination netlink address. */
  memset (&snl, 0, sizeof snl);
  snl.nl_family = AF_NETLINK;

  /*除了参数n的地址外，还通过netlink_talk下发了netlink_cmd，也就是netlink socket,以及zvrf指定的路由表*/

  /* Talk to netlink socket. */
  return netlink_talk (&req.n, &zvrf->netlink_cmd, zvrf);
}

我们再看看netlink_talk的函数实现。

```c
/* sendmsg() to netlink socket then recvmsg(). */
static int
netlink_talk (struct nlmsghdr *n, struct nlsock *nl, struct zebra_vrf *zvrf)
{
  int status;
  struct sockaddr_nl snl;

  /*构建消息缓冲区iovec和消息体msghdr，需要注意的是这里直接使用nlmsghdr传入的地址作为缓冲区，使用nlmsg_len标记缓冲区长度*/
  struct iovec iov = {
    .iov_base = (void *) n,
    .iov_len = n->nlmsg_len
  };
  struct msghdr msg = {
    .msg_name = (void *) &snl,
    .msg_namelen = sizeof snl,
    .msg_iov = &iov,
    .msg_iovlen = 1,
  };
  int save_errno;

  memset (&snl, 0, sizeof snl);
  snl.nl_family = AF_NETLINK;

  n->nlmsg_seq = ++nl->seq;

  /*这里表示需要ACK应答消息*/

  /* Request an acknowledgement by setting NLM_F_ACK */
  n->nlmsg_flags |= NLM_F_ACK;

  if (IS_ZEBRA_DEBUG_KERNEL)
    zlog_debug ("netlink_talk: %s type %s(%u), seq=%u", nl->name,
               lookup (nlmsg_str, n->nlmsg_type), n->nlmsg_type,
               n->nlmsg_seq);

  /*通过sendmsg发送消息给内核*/

  /* Send message to netlink interface. */
  if (zserv_privs.change (ZPRIVS_RAISE))
    zlog (NULL, LOG_ERR, "Can't raise privileges");
  status = sendmsg (nl->sock, &msg, 0);
  save_errno = errno;
  if (zserv_privs.change (ZPRIVS_LOWER))
    zlog (NULL, LOG_ERR, "Can't lower privileges");

  if (status < 0)
    {
      zlog (NULL, LOG_ERR, "netlink_talk sendmsg() error: %s",
            safe_strerror (save_errno));
      return -1;
    }

   /*获取内核响应，并调用netlink_parse_info进行解析*/

  /* 
   * Get reply from netlink socket. 
   * The reply should either be an acknowlegement or an error.
   */
  return netlink_parse_info (netlink_talk_filter, nl, zvrf);
}
```

我们再看看netlink_parse_info的实现，因为代码比较长，所以只选取重点段落分析。

```c
/* Receive message from netlink interface and pass those information
   to the given function. */
static int
netlink_parse_info (int (*filter) (struct sockaddr_nl *, struct nlmsghdr *,
                                   vrf_id_t),
                    struct nlsock *nl, struct zebra_vrf *zvrf)
{
  int status;
  int ret = 0;
  int error;

  while (1)
    {
      /*构建接收缓冲区以及接收消息结构体*/
      struct iovec iov = {
        .iov_base = nl_rcvbuf.p,
        .iov_len = nl_rcvbuf.size,
      };
      struct sockaddr_nl snl;
      struct msghdr msg = {
        .msg_name = (void *) &snl,
        .msg_namelen = sizeof snl,
        .msg_iov = &iov,
        .msg_iovlen = 1
      };
      struct nlmsghdr *h;

      /*通过recvmsg接收内核的响应消息*/
      status = recvmsg (nl->sock, &msg, 0);
      if (status < 0)
        {
          if (errno == EINTR)
            continue;
          if (errno == EWOULDBLOCK || errno == EAGAIN)
            break;
          zlog (NULL, LOG_ERR, "%s recvmsg overrun: %s",
	  	nl->name, safe_strerror(errno));
          continue;
        }

      if (status == 0)
        {
          zlog (NULL, LOG_ERR, "%s EOF", nl->name);
          return -1;
        }

      if (msg.msg_namelen != sizeof snl)
        {
          zlog (NULL, LOG_ERR, "%s sender address length error: length %d",
                nl->name, msg.msg_namelen);
          return -1;
        }
      
      /*循环处理nlmsghdr消息*/
      for (h = (struct nlmsghdr *) nl_rcvbuf.p; 
           NLMSG_OK (h, (unsigned int) status);
           h = NLMSG_NEXT (h, status))
        {
             /*此处省略处理具体消息的流程*/
		}

          /* OK we got netlink message. */
          if (IS_ZEBRA_DEBUG_KERNEL)
            zlog_debug ("netlink_parse_info: %s type %s(%u), seq=%u, pid=%u",
                       nl->name,
                       lookup (nlmsg_str, h->nlmsg_type), h->nlmsg_type,
                       h->nlmsg_seq, h->nlmsg_pid);

        /*此处省略进行一些错误信息处理的流程*/
    }

  return ret;
}
```

经过上面的分析，我们就知道了zebra如何使用netlink和内核交互路由配置信息，接下来我们看看socket方式是如何实现的。

在rt_socket.c中我们找到了老朋友kernel_route_rib函数：

```c
int
kernel_route_rib (struct prefix *p, struct rib *old, struct rib *new)
{
  int route = 0;

  if (zserv_privs.change(ZPRIVS_RAISE))
    zlog (NULL, LOG_ERR, "Can't raise privileges");

  if (old)
    route |= kernel_rtm (RTM_DELETE, p, old);

  if (new)
    route |= kernel_rtm (RTM_ADD, p, new);

  if (zserv_privs.change(ZPRIVS_LOWER))
    zlog (NULL, LOG_ERR, "Can't lower privileges");

  return route;
}
```

可以看到不论是添加还是删除路由，最终都通过kernel_rtm实现的，而kernel_rtm根据Address Family类型AF_INET和AF_INET6，分别调用kernel_rtm_ipv4和kernel_rtm_ipv6。我们以kernel_rtm_ipv4为例看一下具体流程, kernel_rtm_ipv6的实现是类似的。

```c
/* Interface between zebra message and rtm message. */
static int
kernel_rtm_ipv4 (int cmd, struct prefix *p, struct rib *rib)

{
  struct sockaddr_in *mask = NULL;
  struct sockaddr_in sin_dest, sin_mask, sin_gate;
  struct nexthop *nexthop, *tnexthop;
  int recursing;
  int nexthop_num = 0;
  ifindex_t ifindex = 0;
  int gate = 0;
  int error;
  char prefix_buf[PREFIX_STRLEN];

  /*初始化地址簇相关信息*/

  if (IS_ZEBRA_DEBUG_RIB)
    prefix2str (p, prefix_buf, sizeof(prefix_buf));
  memset (&sin_dest, 0, sizeof (struct sockaddr_in));
  sin_dest.sin_family = AF_INET;
#ifdef HAVE_STRUCT_SOCKADDR_IN_SIN_LEN
  sin_dest.sin_len = sizeof (struct sockaddr_in);
#endif /* HAVE_STRUCT_SOCKADDR_IN_SIN_LEN */
  sin_dest.sin_addr = p->u.prefix4;

  memset (&sin_mask, 0, sizeof (struct sockaddr_in));

  memset (&sin_gate, 0, sizeof (struct sockaddr_in));
  sin_gate.sin_family = AF_INET;
#ifdef HAVE_STRUCT_SOCKADDR_IN_SIN_LEN
  sin_gate.sin_len = sizeof (struct sockaddr_in);
#endif /* HAVE_STRUCT_SOCKADDR_IN_SIN_LEN */

  /* Make gateway. */
  for (ALL_NEXTHOPS_RO(rib->nexthop, nexthop, tnexthop, recursing))
    {
      if (CHECK_FLAG (nexthop->flags, NEXTHOP_FLAG_RECURSIVE))
        continue;

      gate = 0;
      char gate_buf[INET_ADDRSTRLEN] = "NULL";

      /*此处省略一些错误流程检查*/

      /*通过rtm_write下发路由信息*/
	  error = rtm_write (cmd,
			     (union sockunion *)&sin_dest, 
			     (union sockunion *)mask, 
			     gate ? (union sockunion *)&sin_gate : NULL,
			     ifindex,
			     rib->flags,
			     rib->metric);

           if (IS_ZEBRA_DEBUG_RIB)
           {
             if (!gate)
             {
               zlog_debug ("%s: %s: attention! gate not found for rib %p",
                 __func__, prefix_buf, rib);
               rib_dump (p, rib);
             }
             else
               inet_ntop (AF_INET, &sin_gate.sin_addr, gate_buf, INET_ADDRSTRLEN);
           }

           /*此处省略一些错误信息处理流程*/
 
     } /* for (ALL_NEXTHOPS_RO(...))*/

  return 0; /*XXX*/
}
```

我们再来看看rtm_write的实现。

```c

/* Interface function for the kernel routing table updates.  Support
 * for RTM_CHANGE will be needed.
 * Exported only for rt_socket.c
 */
int
rtm_write (int message,
	   union sockunion *dest,
	   union sockunion *mask,
	   union sockunion *gate,
	   unsigned int index,
	   int zebra_flags,
	   int metric)
{
  int ret;
  caddr_t pnt;
  struct interface *ifp;

  /* Sequencial number of routing message. */
  static int msg_seq = 0;

  /* Struct of rt_msghdr and buffer for storing socket's data. */
  struct 
  {
    struct rt_msghdr rtm;
    char buf[512];
  } msg;
  
  if (routing_sock < 0)
    return ZEBRA_ERR_EPERM;

  /*构建rt_msghdr结构体信息*/

  /* Clear and set rt_msghdr values */
  memset (&msg, 0, sizeof (struct rt_msghdr));
  msg.rtm.rtm_version = RTM_VERSION;
  msg.rtm.rtm_type = message;
  msg.rtm.rtm_seq = msg_seq++;
  msg.rtm.rtm_addrs = RTA_DST;
  msg.rtm.rtm_addrs |= RTA_GATEWAY;
  msg.rtm.rtm_flags = RTF_UP;
  msg.rtm.rtm_index = index;

  /*各种标志位设置*/
  if (metric != 0)
    {
      msg.rtm.rtm_rmx.rmx_hopcount = metric;
      msg.rtm.rtm_inits |= RTV_HOPCOUNT;
    }

  ifp = if_lookup_by_index (index);

  if (gate && (message == RTM_ADD || message == RTM_CHANGE))
    msg.rtm.rtm_flags |= RTF_GATEWAY;

  if (mask)
    msg.rtm.rtm_addrs |= RTA_NETMASK;
  else if (message == RTM_ADD || message == RTM_CHANGE)
    msg.rtm.rtm_flags |= RTF_HOST;

  /* Tagging route with flags */
  msg.rtm.rtm_flags |= (RTF_PROTO1);

  /* Additional flags. */
  if (zebra_flags & ZEBRA_FLAG_BLACKHOLE)
    msg.rtm.rtm_flags |= RTF_BLACKHOLE;
  if (zebra_flags & ZEBRA_FLAG_REJECT)
    msg.rtm.rtm_flags |= RTF_REJECT;

  /*地址前缀，掩码，下一跳网关地址赋值*/
#define SOCKADDRSET(X,R) \
  if (msg.rtm.rtm_addrs & (R)) \
    { \
      int len = SAROUNDUP (X); \
      memcpy (pnt, (caddr_t)(X), len); \
      pnt += len; \
    }

  pnt = (caddr_t) msg.buf;

  /* Write each socket data into rtm message buffer */
  SOCKADDRSET (dest, RTA_DST);
  SOCKADDRSET (gate, RTA_GATEWAY);
  SOCKADDRSET (mask, RTA_NETMASK);

  msg.rtm.rtm_msglen = pnt - (caddr_t) &msg;

  /*调用write将msg消息下发内核*/
  ret = write (routing_sock, &msg, msg.rtm.rtm_msglen);

  /*返回信息处理*/
  if (ret != msg.rtm.rtm_msglen) 
    {
      if (errno == EEXIST) 
	return ZEBRA_ERR_RTEXIST;
      if (errno == ENETUNREACH)
	return ZEBRA_ERR_RTUNREACH;
      if (errno == ESRCH)
	return ZEBRA_ERR_RTNOEXIST;
      
      zlog_warn ("%s: write : %s (%d)", __func__, safe_strerror (errno), errno);
      return ZEBRA_ERR_KERNEL;
    }
  return ZEBRA_ERR_NOERROR;
}
```

根据上述分析，我们大概知道了netlink消息通过nlmsghdr，rtmsg和buf下发路由配置，而socket通过rt_msghdr和buf来下发路由配置。netlink消息之所以分为nlmsghdr和rtmsg两个部分，是因为nlmsghdr是netlink通用信息头，而rtmsg才是路由相关头信息。不论哪种方式，最终的路由配置记录都存放在后面的buf中，通过前面的头信息中的len字段来标识buf的边界。

虽然上面分析的是 Quagga,但是后面fork出的 Frr 和上面的实现也差不多，这里就不展开对比了。需要注意的一点是，不管是 Quagga 还是 Frr,在使用socket类型将路由信息写入内核的时候，都没有通过read/recv读取返回状态信息，而netlink方式会通过recvmsg读取返回信息。

netlink套接字的初始化代码：

```c
/* Make socket for Linux netlink interface. */
static int
netlink_socket (struct nlsock *nl, unsigned long groups, vrf_id_t vrf_id)
{
  int ret;
  struct sockaddr_nl snl;
  int sock;
  int namelen;
  int save_errno;

  if (zserv_privs.change (ZPRIVS_RAISE))
    {
      zlog (NULL, LOG_ERR, "Can't raise privileges");
      return -1;
    }

  /*使用 AF_NETLINK 地址簇创建，协议类型是NETLINK_ROUTE*/
  sock = vrf_socket (AF_NETLINK, SOCK_RAW, NETLINK_ROUTE, vrf_id);
  if (sock < 0)
    {
      zlog (NULL, LOG_ERR, "Can't open %s socket: %s", nl->name,
            safe_strerror (errno));
      return -1;
    }

  memset (&snl, 0, sizeof snl);
  snl.nl_family = AF_NETLINK;
  snl.nl_groups = groups;

  /* Bind the socket to the netlink structure for anything. */
  ret = bind (sock, (struct sockaddr *) &snl, sizeof snl);
  save_errno = errno;
  if (zserv_privs.change (ZPRIVS_LOWER))
    zlog (NULL, LOG_ERR, "Can't lower privileges");

  if (ret < 0)
    {
      zlog (NULL, LOG_ERR, "Can't bind %s socket to group 0x%x: %s",
            nl->name, snl.nl_groups, safe_strerror (save_errno));
      close (sock);
      return -1;
    }

  /* multiple netlink sockets will have different nl_pid */
  namelen = sizeof snl;
  ret = getsockname (sock, (struct sockaddr *) &snl, (socklen_t *) &namelen);
  if (ret < 0 || namelen != sizeof snl)
    {
      zlog (NULL, LOG_ERR, "Can't get %s socket name: %s", nl->name,
            safe_strerror (errno));
      close (sock);
      return -1;
    }

  nl->snl = snl;
  nl->sock = sock;
  return ret;
}
```

socket套接字的初始化代码：

```c
/* Make routing socket. */
static void
routing_socket (struct zebra_vrf *zvrf)
{
  if (zvrf->vrf_id != VRF_DEFAULT)
    return;

  if ( zserv_privs.change (ZPRIVS_RAISE) )
    zlog_err ("routing_socket: Can't raise privileges");

  /*使用AF_ROUTE地址簇创建，协议类型为0*/
  routing_sock = socket (AF_ROUTE, SOCK_RAW, 0);

  if (routing_sock < 0) 
    {
      if ( zserv_privs.change (ZPRIVS_LOWER) )
        zlog_err ("routing_socket: Can't lower privileges");
      zlog_warn ("Can't init kernel routing socket");
      return;
    }

  /* XXX: Socket should be NONBLOCK, however as we currently 
   * discard failed writes, this will lead to inconsistencies.
   * For now, socket must be blocking.
   */
  /*if (fcntl (routing_sock, F_SETFL, O_NONBLOCK) < 0) 
    zlog_warn ("Can't set O_NONBLOCK to routing socket");*/
    
  if ( zserv_privs.change (ZPRIVS_LOWER) )
    zlog_err ("routing_socket: Can't lower privileges");

  /* kernel_read needs rewrite. */
  thread_add_read (zebrad.master, kernel_read, NULL, routing_sock);
}
```

需要补充的一点是，在Linux下，AF_ROUTE实际使用的就是AF_NETLINK。但是你并不能使用这种套接字兼容UNIX下的route消息格式rt_msghdr。

```c
#define AF_NETLINK	16
#define AF_ROUTE	AF_NETLINK /* Alias to emulate 4.4BSD */
```