---
layout: post
title:  "Linux select"
date:   2026-08-09
category: networking
tags:   linux select
---

## select使用记录

### man select 

https://www.man7.org/linux/man-pages/man2/select.2.html

```
       WARNING: select() can monitor only file descriptors numbers that
       are less than FD_SETSIZE (1024)—an unreasonably low limit for many
       modern applications—and this limitation will not change.  All
       modern applications should instead use poll(2) or epoll(7), which
       do not suffer this limitation.
```

select只能监测FD_SETSIZE(一般是1024)个文件描述符，如果实际应用超过这个，则推荐使用poll或者epoll。

```
       On Linux, select() also modifies timeout if the call is
       interrupted by a signal handler (i.e., the EINTR error return).
       This is not permitted by POSIX.1.  The Linux pselect() system call
       has the same behavior, but the glibc wrapper hides this behavior
       by internally copying the timeout to a local variable and passing
       that variable to the system call.

       On Linux, select() modifies timeout to reflect the amount of time
       not slept; most other implementations do not do this.  (POSIX.1
       permits either behavior.)  This causes problems both when Linux
       code which reads timeout is ported to other operating systems, and
       when code is ported to Linux that reuses a struct timeval for
       multiple select()s in a loop without reinitializing it.  Consider
       timeout to be undefined after select() returns.
```

select会修改timeout参数，比如被信号中断，或者在超时时间内返回，或者超时时间到了会返回，总之不要信任select返回后timeout的值，必须在每次select之前重新初始化timeout。

除了上述特点之外，select每次都会讲所有的readfd, writefd, exceptfd拷贝到内核中，当fd很多或者select频繁收发数据的时候，这个开销可能会造成的一定影响。

下面我们看看ping, traceroute，ppp,frrouting是如何使用select/poll进行多路复用的。

### ubuntu ping源码地址

https://git.launchpad.net/ubuntu/+source/iputils/tree/ping?h=ubuntu/plucky

其中ping_common.c文件中main_loop函数中使用poll轮询fd数据

```c
			if (!polling &&
			    (rts->opt_adaptive || rts->opt_flood_poll || rts->interval)) {
				struct pollfd pset;
				pset.fd = sock->fd;
				pset.events = POLLIN;
				pset.revents = 0;
				if (poll(&pset, 1, next) < 1 ||
				    !(pset.revents & (POLLIN | POLLERR)))
					continue;
				polling = MSG_DONTWAIT;
			
```

### ubuntu traceroute源码地址

https://git.launchpad.net/ubuntu/+source/traceroute/tree/traceroute

其中poll.c文件中do_poll函数使用poll轮询fd数据

```c
void do_poll (double timeout, void (*callback) (int fd, int revents)) {
	int nfds, n, i;

	nfds = cleanup_polls ();

	if (!nfds)  return;

	n = poll (pfd, nfds, ceil(timeout * 1000));
	if (n < 0) {
	    if (errno == EINTR)  return;
	    error ("poll");
	}

	for (i = 0; n && i < num_polls; i++) {
	    if (pfd[i].revents) {
		callback (pfd[i].fd, pfd[i].revents);
		n--;
	    }
	}

	return;
}
```

当然，也有一些历史悠久的代码，还使用select实现多路复用，比如ppp

### ppp源码地址

https://github.com/ppp-project/ppp/blob/master/pppd/event-handler.c#L56

```c
void wait_input(struct timeval *timo)
{
    fd_set ready, exc;
    int n;
    struct event_handler* h = handlers, *nh;

    called_remove = 0;
    ready = in_fds;
    exc = in_fds;
    n = select(max_in_fd + 1, &ready, NULL, &exc, timo);
    if (n < 0 && errno != EINTR)
	fatal("select: %m");

    while (h) {
	nh = h->next;
	if (FD_ISSET(h->fd, &ready)) {
	    FD_CLR(h->fd, &ready); /* clear so that if we need to re-iterate we won't call again */
	    h->cb(h->fd, h->ctx);

	    if (called_remove) {
		nh = handlers;
		called_remove = 0;
	    }
	}
	h = nh;
    }
}
```

### FRRouting 源码地址

https://github.com/FRRouting/frr/blob/master/lib/event.c#L1080

当前的Frrouting(20260809)中event.c文件中的do_poll函数实现io多路复用，根据编译选项支持poll/ppoll/epoll几种方式。

```c
#if defined(USE_EPOLL) && defined(HAVE_EPOLL_PWAIT)
	num = epoll_pwait(m->handler.epoll_fd, m->handler.revents, m->handler.eventsize,
			  timeout, &origsigs);
	pthread_sigmask(SIG_SETMASK, &origsigs, NULL);
#elif defined(HAVE_PPOLL)
	num = ppoll(m->handler.copy, count + 1, tsp, &origsigs);
	pthread_sigmask(SIG_SETMASK, &origsigs, NULL);
#else
	/* Not ideal - there is a race after we restore the signal mask */
	pthread_sigmask(SIG_SETMASK, &origsigs, NULL);
	num = poll(m->handler.copy, count + 1, timeout);
#endif
```

其实在Frrouing 3.0版本以及之前的Quagga/Zebra的版本上，是支持select的。

https://github.com/FRRouting/frr/blob/stable/3.0/lib/thread.c#L606

```c
#if defined(HAVE_POLL_CALL)
	/* recalc timeout for poll. Attention NULL pointer is no timeout with
	select, where with poll no timeount is -1 */
	int timeout = -1;
	if (timer_wait != NULL)
		timeout = (timer_wait->tv_sec * 1000)
			  + (timer_wait->tv_usec / 1000);

	num = poll(m->handler.pfds,
		   m->handler.pfdcount + m->handler.pfdcountsnmp, timeout);
#else
	num = select(size, read, write, except, timer_wait);
#endif
```

但是从4.0开始就去掉对select的支持了。

https://github.com/FRRouting/frr/blob/stable/4.0/lib/thread.c#L663

具体的是这个commit: https://github.com/FRRouting/frr/commit/75bcb3558d25b8ca7d3383f5c2c648d0aceae103

```
lib: remove select()
poll() is present on every supported platform and does not have an upper
limit on file descriptors.

Signed-off-by: Quentin Young <qlyoung@cumulusnetworks.com>
[DL: split off from AWAKEN() change]
```

在这个commit(2020年10月29日)增加了对ppoll的支持: https://github.com/FRRouting/frr/commit/d81ca9a3faabe54f57b11acf87585e48d3a44480 

在这个commit中(2023年3月24日)把thread.c/h修改为event.c/h: https://github.com/FRRouting/frr/commit/cb37cb336a2cca77bfbaf6b0cfab12e847e45623

大概是在2025年的12月这个commit引入了epoll的支持：https://github.com/FRRouting/frr/commit/ff05cdcc153fcda62413646bef063f668edbaa21。但是根据blame的显示，应该是这个commit: https://github.com/FRRouting/frr/commit/747af764419291c802480665d7a7e918eae8c112,不过这个日期是2121年，可能是提交者本地的时间设置错了？具体时间应该是在2025年的11月9日到2025年的12月5日之间。