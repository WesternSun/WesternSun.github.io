---
title: runtime 中的环境变量
date: 2019-02-27 23:47:09
categories:
- Golang
tags:
- Golang runtime
---

# runtime 中的环境变量

The following environment variables ($name or %name%, depending on the host operating system) control the run-time behavior of Go programs. The meanings and use may change from release to release.

## GOGC
The GOGC variable sets the initial garbage collection target percentage. A collection is triggered when the ratio of freshly allocated data to live data remaining after the previous collection reaches this percentage. The default is GOGC=100. Setting GOGC=off disables the garbage collector entirely. The runtime/debug package's SetGCPercent function allows changing this percentage at run time. 

## GODEBUG

只要在程序执行之前加上环境变量GODEBUG gctrace =1 ，如：
GODEBUG=gctrace=1 ./binary or GODEBUG=gctrace=1 go run main.go or GODEBUG gctrace =1 ./binary > trace.log

程序将会显示gc信息，如下
```text
gc 6 @4.741s 0%: 0.008+35+0.17 ms clock, 0.035+0.19/35/100+0.70 ms cpu, 76->76->75 MB, 78 MB goal, 4 P
gc 7 @6.688s 0%: 0.020+117+0.34 ms clock, 0.082+11/117/330+1.3 ms cpu, 147->148->146 MB, 151 MB goal, 4 P
gc 8 @68.645s 0%: 0.019+146+0.30 ms clock, 0.078+0.006/146/407+1.2 ms cpu, 285->285->248 MB, 292 MB goal, 4 P
scvg0: inuse: 426, idle: 0, sys: 427, released: 0, consumed: 427 (MB)
```

```text
gctrace: setting gctrace=1 causes the garbage collector to emit a single line to standard
error at each collection, summarizing the amount of memory collected and the
length of the pause. The format of this line is subject to change.
Currently, it is:
	gc # @#s #%: #+#+# ms clock, #+#/#/#+# ms cpu, #->#-># MB, # MB goal, # P
where the fields are as follows:
	gc #        the GC number, incremented at each GC
	@#s         time in seconds since program start
	#%          percentage of time spent in GC since program start
	#+...+#     wall-clock/CPU times for the phases of the GC
	#->#-># MB  heap size at GC start, at GC end, and live heap
	# MB goal   goal heap size
	# P         number of processors used
The phases are stop-the-world (STW) sweep termination, concurrent
mark and scan, and STW mark termination. The CPU times
for mark/scan are broken down in to assist time (GC performed in
line with allocation), background GC time, and idle GC time.
If the line ends with "(forced)", this GC was forced by a
runtime.GC() call.

Setting gctrace to any value > 0 also causes the garbage collector
to emit a summary when memory is released back to the system.
This process of returning memory to the system is called scavenging.
The format of this summary is subject to change.
Currently it is:
	scvg#: # MB released  printed only if non-zero
	scvg#: inuse: # idle: # sys: # released: # consumed: # (MB)
where the fields are as follows:
	scvg#        the scavenge cycle number, incremented at each scavenge
	inuse: #     MB used or partially used spans
	idle: #      MB spans pending scavenging
	sys: #       MB mapped from the system
	released: #  MB released to the system
	consumed: #  MB allocated from the system
```text

**举例说明：**
```text
垃圾回收信息
gc 1 @2.104s 0%: 0.018+1.3+0.076 ms clock, 0.054+0.35/1.0/3.0+0.23 ms cpu, 4->4->3 MB, 5 MB goal, 4 P。

1 表示第一次执行
@2.104s 表示程序执行的总时间
0% 垃圾回收时间占用的百分比，（不知道和谁比？难道是和上面的程序执行总时间，这样比较感觉没意义）
0.018+1.3+0.076 ms clock 垃圾回收的时间，分别为STW（stop-the-world）清扫的时间, 并发标记和扫描的时间，STW标记的时间
0.054+0.35/1.0/3.0+0.23 ms cpu 垃圾回收占用cpu时间
4->4->3 MB 堆的大小，gc后堆的大小，存活堆的大小
5 MB goal 整体堆的大小
4 P 使用的处理器数量

系统内存回收信息
scvg0: inuse: 426, idle: 0, sys: 427, released: 0, consumed: 427 (MB)

426 使用多少M内存
0 剩下要清除的内存
427 系统映射的内存
0 释放的系统内存
427 申请的系统内存
```