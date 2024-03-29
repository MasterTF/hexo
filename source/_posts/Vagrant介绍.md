---
title: Vagrant介绍
date: 2017-06-26 10:56:09
tags:
    - Vagrant
    - 工具
categories:
    - 工具
typora-root-url: ../../source
typora-copy-images-to: ../../source/img
---

## 虚拟开发环境
平常我们经常会遇到这样的问题：在开发机上面开发完毕程序，放到正式环境之后会出现各种奇怪的问题：描述符少了、nginx配置不正确、MySQL编码不对、php缺少模块、glibc版本太低等。

所以我们就需要虚拟开发环境，我们虚拟和正式环境一样的虚拟开发环境，而随着个人开发机硬件的升级，我们可以很容易的在本机跑虚拟机，例如VMware、VirtualBox等。因此使用虚拟化开发环境，在本机可以运行自己喜欢的OS（Windows、Ubuntu、Mac等），开发的程序运行在虚拟机中，这样迁移到生产环境可以避免环境不一致导致的莫名错误。

<!-- more -->

虚拟开发环境特别适合团队中开发环境、测试环境、正式环境不同的场合，这样就可以使得整个团队保持一致的环境，我写这一章的初衷就是为了让大家和我的开发环境保持一致，让读者和我们整个大团队保持一致的开发环境。

## Vagrant
>Vagrant是一款用于构建及配置虚拟开发环境的软件，基于Ruby,主要以命令行的方式运行。

>主要使用Oracle的开源VirtualBox虚拟化系统，与Chef，Salt，Puppet等环境配置管理软件搭配使用， 可以实行快速虚拟开发环境的构建。

>早期以VirtualBox为对象，1.1以后的版本中开始对应VMware等虚拟化软件，包括Amazon EC2之类服务器环境的对应。

>https://zh.wikipedia.org/wiki/Vagrant

官方文档：https://www.vagrantup.com/intro/getting-started/index.html

Vagrant就是为了方便的实现虚拟化环境而设计的，使用Ruby开发，基于VirtualBox等虚拟机管理软件的接口，提供了一个可配置、轻量级的便携式虚拟开发环境。使用Vagrant可以很方便的就建立起来一个虚拟环境，而且可以模拟多台虚拟机，这样我们平时还可以在开发机模拟分布式系统。

Vagrant还会创建一些共享文件夹，用来给你在主机和虚拟机之间共享代码用。这样就使得我们可以在主机上写程序，然后在虚拟机中运行。如此一来团队之间就可以共享相同的开发环境，就不会再出现类似“只有你的环境才会出现的bug”这样的事情。

团队新员工加入，常常会遇到花一天甚至更多时间来从头搭建完整的开发环境，而有了Vagrant，只需要直接将已经打包好的package（里面包括开发工具，代码库，配置好的服务器等）拿过来就可以工作了，这对于提升工作效率非常有帮助。

Vagrant不仅可以用来作为个人的虚拟开发环境工具，而且特别适合团队使用，它使得我们虚拟化环境变得如此的简单，只要一个简单的命令就可以开启虚拟之路。

