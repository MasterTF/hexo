---
title: ZSH 安装
date: 2017-03-29 10:41:44
tags:
    - ZSH
    - 工具
categories:
    - 工具
---

介绍下比较好用的zsh安装以及相关的配置

<!-- more -->

## 安装iTerm2
iTerm2官方下载地址 http://www.iterm2.com/downloads.html

## 安装Oh My Bash
1. 通过cat /etc/shells命令可以查看当前系统可以使用哪些shell；
    - /bin/bash
    - /bin/csh
    - /bin/ksh
    - /bin/sh
    - /bin/tcsh
    - /bin/zsh
2. 通过echo $SHELL命令可以查看我们当前正在使用的shell；

    - Mac系统中默认的shell为bash shell
    - /bin/bash

3. 如果当前的shell不是zsh，我们可以通过chsh -s /bin/zsh命令可以将shell切换为shell之zsh，终端重启之后即可生效。

4. 将shell切换为zsh之后，我们就可以安装Oh My ZSH了
官方推荐的安装方法为：

sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"

记住，爱翻墙的童鞋记得先关闭代理哦，不然没法下载成功的
出线以下提示，便是安装成功了^_^
![](/img/15301540285993.jpg)


## 配置agnoster主题
Oh My Zsh提供的所有主题在线预览：
https://github.com/robbyrussell/oh-my-zsh/wiki/Themes

安装成功之后，我们可以通过vi ~/.zshrc，设置ZSH_THEME="agnoster"对主题进行修改。

注意，agnoster主题能否设置成功，还依赖于以下东西：

1. Solarized Dark配色方案
下载完成之后解压，在iTerm2的Preferences——Profiles——colors——Load Presets——Solarized Dark即可设置终端配色

2. 特殊字体安装
下载完成之后解压，执行其中的install.sh文件，在iTerm2的Preferences——Profiles——Text中同时将Regular Font和Non—ASCII Font设置为Meslo LG M DZ Regular for Powerline即可

## 主题列表
https://github.com/robbyrussell/oh-my-zsh/wiki/themes

