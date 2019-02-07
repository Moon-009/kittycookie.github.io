---
title: ubuntu 挂载虚拟机镜像
date: 2012-06-23 14:54:00
layout: post
---

1.losetup /dev/loop10 /.....img （通过losetup -f查看目录名，替代loop10） 
2.kpartx -a /dev/loop10 
2.mount /dev/mapper/loop10p* /mnt/（p*为mapper目录下的文件名，替换为具体的文件名） 
