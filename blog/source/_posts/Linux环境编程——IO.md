---
title: Linux环境编程——I/O
date: 2020-08-09 20:06:24
tags:
categories:
- Linux
- Linux环境编程
---
**一切皆文件**是Unix/Linux的基本哲学之一。普通文件、目录、字符设备、块设备和网络设备在Linux中均被当作文件来看待，虽然类型不同，但是Linux系统为它们提供了统一的操作接口。

基于POSIX标准，Linux提供了文件I/O，文件I/O称之为不带缓存的I/O（接口包括open()、close()、read()、write()、ioctl()等），不带缓存意味者每次读（read）写（write）都调用内核中的一个系统调用（sys_read()和sys_write()）。这也就是一般所说的低级I/O——操作系统提供的基本I/O服务，与系统绑定，特定于Unix/Linux平台。

标准I/O被称为高级磁盘I/O，是ANSI C建立的标准I/O模型，具备一定的可移植性。标准I/O提供三种类型的缓存，分别是全缓存、行缓存和不带缓存。
<!-- more -->
未完待续……