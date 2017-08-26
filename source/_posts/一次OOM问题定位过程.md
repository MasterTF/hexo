---
title: 一次OOM问题定位过程
date: 2017-08-26 20:10:44
tags:
    - JVM
    - JAVA
categories:
    - 技术
    - JAVA
---

## 背景

8.21日bdop dev环境机器突然发生oom服务不可用，重启服务后也很快发生OOM。

之前一直运行没问题，OOM是突然发生的。

<!-- more -->

## 排查过程

### 获取dump文件

之前机器上使用**jps -v **查看过启动参数**：(下面的参数不是当时的，只是日期有差别,删除了公司目录相关参数）**

> 4597 Bootstrap -Dfile.encoding=UTF-8 -Dsun.jnu.encoding=UTF-8 -Djava.net.preferIPv6Addresses=false -Djava.io.tmpdir=/tmp -Djetty.defaultsDescriptor=WEB-INF/web.xml -Duser.timezone=GMT+08 -Xloggc:/XX/gc.log.201708251824 -XX:ErrorFile=/vmerr.log.201708251824 -XX:HeapDumpPath=/XX/bdop.heaperr.log.201708251824 -XX:+HeapDumpOnOutOfMemoryError -XX:+DisableExplicitGC -XX:+PrintGCDetails -XX:+PrintGCDateStamps -Denvironment=test -Dmedis_environment=test -Dcore.zookeeper=sgconfig-zk.sankuai.com:9331 

注意 **XX:HeapDumpPath=/var/sankuai/logs/com.sankuai.sjst.m.bdop.heaperr.log.201708251824 -XX:+HeapDumpOnOutOfMemoryError **参数，这个是配置发生内存溢出时dump出文件到该路径下 。

到路径下获取dump文件。

ps：从服务器上取文件可用 python -m SimpleHTTPServer开一个http服务获取。

由于dump文件很大，下载前先压缩会小很多。

### MAT工具排查

将下载的dump文件后缀名改为".hprof"，然后使用MAT打开，MAT介绍：

[http://www.jianshu.com/p/d8e247b1e7b2](http://www.jianshu.com/p/d8e247b1e7b2)

[http://wiki.eclipse.org/MemoryAnalyzer](http://wiki.eclipse.org/MemoryAnalyzer)

[https://www.yourkit.com/docs/java/help/sizes.jsp](https://www.yourkit.com/docs/java/help/sizes.jsp)

1. 在界面选择 "Dominator Tree":
   {% asset_img dominator_tree.png dominator tree %}
2. 按对象大小排序
   {% asset_img object_size_sorted.png object size sorted %}
   发现databus线程对象有900M大小。databus在服务中是用来同步表数据到ES中的，继续向下看对象里包含了什么
3. 线程对象持有的对象
   {% asset_img reference_objects.png reference objects %}
   发现持有了大量的hashmap和hashset对象。hashset是使用hashmap实现，所以直接看hashset。
   mat显示hashset中一共有450000+个数据，内存溢出很可能就是这里导致的。
   数据内容都还是一样的：
   {% asset_img same_data.png same data %}
4. 由于是databus线程时同步表数据，于是到表中查找包含这些拼音的数据，但并没找到。
5. 继续看线程对象持有的对象，看看是否有其它信息有用信息
   {% asset_img poi_id.png poi_id %}
   对象持有的string对象直接给出了poi_id，所以到表里直接查该数据，发现该poi的poi_name字段为“需要超长超长超长超长超长超长超长超长....”
6. 表中数据找到了但为什么会导致内存溢出呢？当时使用了一个比较笨的方法，直接在项目中搜 "HashSeet"，因为hashset还是比较少用的
7. 发现在databus更新ES时有用到 将门店名称拼音存到索引中。直接将该门店名称放到单测里跑，发现果然很长时间不能结束。
8. 一步步打断点定位到出问题的方法：
   {% asset_img descart_method.png descart method %}
9. 该方法会计算多音字的所有组合。目的是为了支持多音字搜索。debug时发现pinyinSets大小有49，汉字“超长超长超长...”有多个“长”，因此会有多个2的n次方中组合。
   **导致内存中有大量hashset，最终导致内存溢出**

## 解决

找到相关负责人修改，限制笛卡尔积最大长度为32，超过这个大小的多音字不再处理。

## 后续

该问题发生在线下，当天就改完了但没上线。第二天线上也很巧的也发生了一样故障导致OOM。还好前一天已经定位出了问题，直接发布上线解决。

不然线上服务会不可用比较长时间。

## 感想

1.线下很奇怪的问题也需要重视，这次如果没有及时定位解决，到了第二天线上出问题就会引起比较严重的事故。会直接导致门店列表页无法展示，几乎影响M端所有系统。

2.难得的一次OOM问题排查实战，还是挺有收获的，后续需继续研究jvm及mat工具使用。