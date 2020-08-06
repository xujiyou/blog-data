---
title: Linux查看Java线程状态
date: 2020-07-15 20:50:23
tags:
---

1，使用jps查找出java进程的pid，如7777
 或 ps -ef | grep java

2，使用top -p 7777观察进程情况，然后Shift+h,显示该进程的所有线程。

3，找出CPU消耗较多的线程id，如7788，将7788转换为16进制0x1e6c，注意是小写。

4，使用jstack 7777 | grep -A 10 0x1e6c 来查询出具体的线程状态。
 -A 10表示查找到所在行的后10行