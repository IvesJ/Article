> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/_XxTYIoZh5iKqvlwlixUPw)

  

![](https://mmbiz.qpic.cn/mmbiz_png/icfqxwApZy9vtWPFVfCcvmHeuicG1glRCjicMicIpDN178onU16Atvr4K3JlwjA7ntHdFODjFA9mqMqBSPlO5nEY7w/640?&wx_fmt=png)

关于 STR

在车载电子系统中，广泛应用 STR(Suspend to RAM) 功能。当 IVI 进入 STR 模式后，整个系统只有 DDR 保持供电，其他的外设都掉电，从而满足低功耗的要求。通过 STR 功能，IVI 可以从低功耗状态下快速恢复工作。在车载项目中，STR 问题是比较繁琐的问题。接下来，将和大家分享一下 STR 问题的一些调试技巧

![](https://mmbiz.qpic.cn/mmbiz_png/3ictUVfS1oSC4Qs000kLGdLuMTEsLQaqQOtuHGknColL235fibibPhAH6WibruQ5icbLcnOKtGnsicpSp5wkeowhUP6A/640?&wx_fmt=png)

抓取更多 suspend 过程中的内核日志

more kernel log

![](https://mmbiz.qpic.cn/sz_mmbiz_png/42ur9DmlIY3pkV0VjgtOlC9Ke5tKgwHquIXbXstA9w3iauGFpOq6IicQykiaIibDtKKq47NmnxNiclgicVtXS73Hj25w/640?&wx_fmt=png)

cmdline 中添加如下内容

```
loglevel=8 no_console_suspend

```

loglevel=8 调整 log 等级为 8

no_console_suspend 禁止 console 的挂起

或者在 console 里执行如下命令

```
echo N > /sys/module/printk/parameters/console_suspend
echo 8 > /proc/sys/kernel/printk

```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/BiccUIcPqLiaaR9eLPDfIwEqZsE67R2NOKibqiajd3t4pk8kbho8V3qicJ8bOpQD0tiarZWduyyuKGtPLempEj1iaoNvw/640?&wx_fmt=png)

打开内核电源管理和休眠调试宏

pm debug

![](https://mmbiz.qpic.cn/mmbiz_png/8FYZvTd2qWC9abJVapqOiaBYdrmpyK9siaJsYgpWn05JF5YEUKyQub9p1HyoDDNLAA23qMWP7KaqPibLZT1twXwQw/640?&wx_fmt=png)

**1. 在内核配置中使能如下宏**

```
CONFIG_PM_DEBUG=y
CONFIG_PM_SLEEP_DEBUG=y

```

打开这两个宏可以在休眠唤醒时打印更多的 log

cmdline 中添加如下内容默认使能调试功能

**2.cmdline 中添加如下内容默认使能调试功能**

```
pm_debug_messages=1

```

或者在 console 中执行如下命令

```
echo 1 > /sys/power/pm_debug_messages

```

![](https://mmbiz.qpic.cn/mmbiz_png/3ictUVfS1oSC4Qs000kLGdLuMTEsLQaqQOtuHGknColL235fibibPhAH6WibruQ5icbLcnOKtGnsicpSp5wkeowhUP6A/640?&wx_fmt=png)

打开 susped/resume 耗时调试开关

pm print times

![](https://mmbiz.qpic.cn/sz_mmbiz_png/42ur9DmlIY3pkV0VjgtOlC9Ke5tKgwHquIXbXstA9w3iauGFpOq6IicQykiaIibDtKKq47NmnxNiclgicVtXS73Hj25w/640?&wx_fmt=png)

在 console 执行如下命令

```
echo 1 > /sys/power/pm_print_times

```

使能后可以从 kernel log 中看到休眠唤醒流程中各个驱动模块的休眠唤醒耗时，以便排查耗时长的模块

![](https://mmbiz.qpic.cn/mmbiz_png/3ictUVfS1oSC4Qs000kLGdLuMTEsLQaqQOtuHGknColL235fibibPhAH6WibruQ5icbLcnOKtGnsicpSp5wkeowhUP6A/640?&wx_fmt=png)

打开内核调试和驱动调试宏

debug driver

![](https://mmbiz.qpic.cn/sz_mmbiz_png/42ur9DmlIY3pkV0VjgtOlC9Ke5tKgwHquIXbXstA9w3iauGFpOq6IicQykiaIibDtKKq47NmnxNiclgicVtXS73Hj25w/640?&wx_fmt=png)

在内核配置中使能如下宏

```
CONFIG_DEBUG_KERNEL=y
CONFIG_DEBUG_DRIVER=y

```

CONFIG_DEBUG_DRIVER 可以用来调试驱动程序的初始化和退出流程

![](https://mmbiz.qpic.cn/mmbiz_png/3ictUVfS1oSC4Qs000kLGdLuMTEsLQaqQOtuHGknColL235fibibPhAH6WibruQ5icbLcnOKtGnsicpSp5wkeowhUP6A/640?&wx_fmt=png)

打印进程冻结失败时的进程信息

debug freezing issue

![](https://mmbiz.qpic.cn/sz_mmbiz_png/42ur9DmlIY3pkV0VjgtOlC9Ke5tKgwHquIXbXstA9w3iauGFpOq6IicQykiaIibDtKKq47NmnxNiclgicVtXS73Hj25w/640?&wx_fmt=png)

在内核代码中打上如下 patch

```
--- a/kernel/sched/core.c
+++ b/kernel/sched/core.c
@@ -6687,9 +6687,14 @@ void sched_show_task(struct task_struct *p)
if (pid_alive(p))
	ppid = task_pid_nr(rcu_dereference(p->real_parent));
	rcu_read_unlock();
	- pr_cont(" stack:%5lu pid:%5d ppid:%6d flags:0x%08lx\n",
	- free, task_pid_nr(p), ppid,
	- (unsigned long)task_thread_info(p)->flags);
	+ if (p->real_parent && p->real_parent->comm)
		+ pr_cont(" stack:%5lu pid:%d pid_comm:%s ppid:%d ppid_comm:%s flags:0x%08lx\n",
			+ free, task_pid_nr(p), p->comm, ppid, p->real_parent->comm,
			+ (unsigned long)task_thread_info(p)->flags);
	+ else
		+ pr_cont(" stack:%5lu pid:%5d ppid:%6d flags:0x%08lx\n",
			+ free, task_pid_nr(p), ppid,
			+ (unsigned long)task_thread_info(p)->flags);
	print_worker_info(KERN_INFO, p);
	trace_android_vh_sched_show_task(p);

```

![](https://mmbiz.qpic.cn/mmbiz_png/ia6mcaXnsliaOzpTLYVw3QCO7ZkN5wpCJCOugzksaFSqILMMV9FxYKDh4xm69OtLwK4G0vhzmm6znTuL1HPPZZ0g/640?wx_fmt=png&from=appmsg&watermark=1)