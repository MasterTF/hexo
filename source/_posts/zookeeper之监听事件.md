---
title: ZooKeeper watch机制
date: 2017-04-24 22:49:14
tags:
    - zookeeper
categories:
    - 技术
    - ZooKeeper
---
## zookeeper watch机制
一个zk的节点可以被监控，包括这个目录中存储的数据的修改，子节点目录的变化，一旦变化可以通知设置监控的客户端，这个功能是zookeeper对于应用最重要的特性，通过这个特性可以实现的功能包括配置的集中管理，集群管理，分布式锁等等。
<!-- more -->

**getData(), getChildren(), and exists()**可以设置对某个节点进行监听。
** New ZooKeeper时注册的watcher叫default watcher，它不是一次性的，只对client的连接状态变化作出反应。**

## zookeeper watch机制特点
### One-time trigger
当数据改变的时候，那么一个Watch事件会产生并且被发送到客户端中。但是客户端只会收到一次这样的通知，如果以后这个数据再次发生改变的时候，之前设置Watch的客户端将不会再次收到改变的通知，因为Watch机制规定了它是一个一次性的触发器。           

当设置监视的数据发生改变时，该监视事件会被发送到客户端，例如，如果客户端调用了 getData("/znode1", true) 并且稍后 /znode1 节点上的数据发生了改变或者被删除了，客户端将会获取到 /znode1 发生变化的监视事件，而如果 /znode1 再一次发生了变化，除非客户端再次对 /znode1 设置监视，否则客户端不会收到事件通知。

```java
package com.ytf.zk.nameservice;

import com.ytf.zk.constant.ClientBase;
import org.apache.zookeeper.CreateMode;
import org.apache.zookeeper.KeeperException;
import org.apache.zookeeper.ZooDefs;
import org.apache.zookeeper.ZooKeeper;

import java.io.IOException;

/**
 * Created by
 * DATE: 17/4/23星期日.
 */
public class Naming {
    private static final int CLIENT_PORT = 2181;
    private static final String ROOT_PATH = "/root";
    private static final String CHILD_PATH = "/root/childPath";
    private static final String CHILD_PATH_2 = "/root/childPath2";
    static ZooKeeper zk = null;

    public static void main(String[] args) throws Exception {
        try {
            zk = new ZooKeeper("localhost:" + CLIENT_PORT, ClientBase.CONNECTION_TIMEOUT, (watchedEvent) -> {
                System.out.println(watchedEvent.getPath() + "触发了" + watchedEvent.getType() + "事件!" + "data:" + Naming.getData(watchedEvent.getPath()));
            }
            );

            // 创建根目录
            zk.create(ROOT_PATH, ROOT_PATH.getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
            System.out.println(zk.getChildren(ROOT_PATH, true));


            // 创建子目录
            zk.create(CHILD_PATH, "childPath".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
            // 取出子目录数据
            System.out.println(CHILD_PATH + "数据: " + new String(zk.getData(CHILD_PATH, true, null)));

            // 修改子目录节点数据
            zk.setData(CHILD_PATH, "modification".getBytes(), -1);
            System.out.println(new String(zk.getData(CHILD_PATH, true, null)));

            zk.setData(CHILD_PATH, "modification2".getBytes(), -1);

            zk.delete(CHILD_PATH, -1);
            // 删除父目录节点
            zk.delete(ROOT_PATH, -1);
            // 关闭连接
            zk.close();
        } catch (KeeperException e) {
            e.printStackTrace();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public static String getData(String path) {
        if (path == null) {
            return null;
        }
        try {
            return new String(zk.getData(path, false, null));
        } catch (KeeperException e) {
            e.printStackTrace();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return null;
    }
}
```
输出:
>null触发了None事件!data:null
>[]
>/root/childPath数据: childPath
>/root触发了NodeChildrenChanged事件!data:/root
>/root/childPath触发了NodeDataChanged事件!data:modification
>modification
>/root/childPath触发了NodeDataChanged事件!data:modification2

注意将第41行的参数true改为false后的执行结果：
```java
package com.ytf.zk.nameservice;

import com.ytf.zk.constant.ClientBase;
import org.apache.zookeeper.CreateMode;
import org.apache.zookeeper.KeeperException;
import org.apache.zookeeper.ZooDefs;
import org.apache.zookeeper.ZooKeeper;

import java.io.IOException;

/**
 * Created by
 * DATE: 17/4/23星期日.
 */
public class Naming {
    private static final int CLIENT_PORT = 2181;
    private static final String ROOT_PATH = "/root";
    private static final String CHILD_PATH = "/root/childPath";
    private static final String CHILD_PATH_2 = "/root/childPath2";
    static ZooKeeper zk = null;

    public static void main(String[] args) throws Exception {
        try {
            zk = new ZooKeeper("localhost:" + CLIENT_PORT, ClientBase.CONNECTION_TIMEOUT, (watchedEvent) -> {
                System.out.println(watchedEvent.getPath() + "触发了" + watchedEvent.getType() + "事件!" + "data:" + Naming.getData(watchedEvent.getPath()));
            }
            );

            // 创建根目录
            zk.create(ROOT_PATH, ROOT_PATH.getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
            System.out.println(zk.getChildren(ROOT_PATH, true));


            // 创建子目录
            zk.create(CHILD_PATH, "childPath".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
            // 取出子目录数据
            System.out.println(CHILD_PATH + "数据: " + new String(zk.getData(CHILD_PATH, true, null)));

            // 修改子目录节点数据
            zk.setData(CHILD_PATH, "modification".getBytes(), -1);
            System.out.println(new String(zk.getData(CHILD_PATH, false, null)));

            zk.setData(CHILD_PATH, "modification2".getBytes(), -1);

            zk.delete(CHILD_PATH, -1);
            // 删除父目录节点
            zk.delete(ROOT_PATH, -1);
            // 关闭连接
            zk.close();
        } catch (KeeperException e) {
            e.printStackTrace();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public static String getData(String path) {
        if (path == null) {
            return null;
        }
        try {
            return new String(zk.getData(path, false, null));
        } catch (KeeperException e) {
            e.printStackTrace();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return null;
    }
}
```
输出:
>null触发了None事件!data:null
>[]
>/root触发了NodeChildrenChanged事件!data:/root
>/root/childPath数据: childPath
>/root/childPath触发了NodeDataChanged事件!data:modification
>modification

看到对于路径/root/childPath的数据修改并没有监听到

### Sent to the client
Watch的通知事件是从服务器异步发送给客户端，不同的客户端收到的Watch的时间可能不同。但是ZooKeeper有保证：当一个客户端在收到Watch事件之前是不会看到结点数据的变化的。例如：A=3，此时在上面设置了一次Watch，如果A突然变成4了，那么客户端会先收到Watch事件的通知，然后才会看到A=4。

Zookeeper 客户端和服务端是通过 Socket 进行通信的，由于网络存在故障，所以监听事件很有可能不会成功地到达客户端，监听事件是异步发送至监视者的，Zookeeper 可以保障顺序性(ordering guarantee)：即客户端只有首先收到监听事件后，才会感知到它所监听的 znode 发生了变化.
>a client will never see a change for which it has set a watch until it first sees the watch event). 

网络延迟或者其他因素可能导致不同的客户端在不同的时刻感知某一监视事件，但是不同的客户端所看到都是一致的顺序。
>The key point is that everything seen by the different clients will have a consistent order.

### The data for which the watch was set
**这部分是重点**
>This refers to the different ways a node can change. It helps to think of ZooKeeper as maintaining two lists of watches: data watches and child watches. getData() and exists() set data watches. getChildren() sets child watches. Alternatively, it may help to think of watches being set according to the kind of data returned. getData() and exists() return information about the data of the node, whereas getChildren() returns a list of children. Thus, setData() will trigger data watches for the znode being set (assuming the set is successful). A successful create() will trigger a data watch for the znode being created and a child watch for the parent znode. A successful delete() will trigger both a data watch and a child watch (since there can be no more children) for a znode being deleted as well as a child watch for the parent znode.

这意味着 znode 节点本身具有不同的改变方式。你也可以想象 Zookeeper 维护了两条监听链表：

数据监听和子节点监听(data watches and child watches) 

**getData() and exists() 设置数据监听,getChildren() 设置子节点监听.**也可以理解成设置为哪种监听是由返回的数据类型决定的。getData() 和 exists() 返回节点数据相关信息，getChildren()返回一个子节点列表。因此，setData()会触发数据监听，create()会触发数据监听及父节点的child watch; delete() 操作将会触发当前节点的数据监视和子节点监视事件，同时也会触发该节点父节点的child watch。

实验一下：
```java
package com.ytf.zk.nameservice;

import com.ytf.zk.constant.ClientBase;
import org.apache.zookeeper.CreateMode;
import org.apache.zookeeper.KeeperException;
import org.apache.zookeeper.ZooDefs;
import org.apache.zookeeper.ZooKeeper;

import java.io.IOException;

/**
 * Created by
 * DATE: 17/4/23星期日.
 */
public class Naming {
    private static final int CLIENT_PORT = 2181;
    private static final String ROOT_PATH = "/root";
    private static final String CHILD_PATH = "/root/childPath";
    private static final String CHILD_PATH_2 = "/root/childPath2";
    static ZooKeeper zk = null;

    public static void main(String[] args) throws Exception {
        try {
            zk = new ZooKeeper("localhost:" + CLIENT_PORT, ClientBase.CONNECTION_TIMEOUT, (watchedEvent) -> {
                System.out.println(watchedEvent.getPath() + "触发了" + watchedEvent.getType() + "事件!" + "data:" + Naming.getData(watchedEvent.getPath()));
            }
            );

            // 创建根目录
            zk.create(ROOT_PATH, ROOT_PATH.getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
            // 创建子目录
            zk.create(CHILD_PATH, "childPath".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);

            // 创建对子目录的监听
            System.out.println(zk.getChildren(CHILD_PATH, true));

            // 修改子目录节点数据,观察根节点是否监听到
            zk.setData(CHILD_PATH, "modification".getBytes(), -1);

            zk.delete(CHILD_PATH, -1);

            // 删除父目录节点
            zk.delete(ROOT_PATH, -1);
            // 关闭连接
            zk.close();
        } catch (KeeperException e) {
            e.printStackTrace();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public static String getData(String path) {
        if (path == null) {
            return null;
        }
        try {
            return new String(zk.getData(path, false, null));
        } catch (KeeperException e) {
            e.printStackTrace();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return null;
    }
}
```
输出：
>null触发了None事件!data:null
>[]

修改代码：
```java
package com.ytf.zk.nameservice;

import com.ytf.zk.constant.ClientBase;
import org.apache.zookeeper.CreateMode;
import org.apache.zookeeper.KeeperException;
import org.apache.zookeeper.ZooDefs;
import org.apache.zookeeper.ZooKeeper;

import java.io.IOException;

/**
 * Created by
 * DATE: 17/4/23星期日.
 */
public class Naming {
    private static final int CLIENT_PORT = 2181;
    private static final String ROOT_PATH = "/root";
    private static final String CHILD_PATH = "/root/childPath";
    private static final String CHILD_PATH_2 = "/root/childPath2";
    static ZooKeeper zk = null;

    public static void main(String[] args) throws Exception {
        try {
            zk = new ZooKeeper("localhost:" + CLIENT_PORT, ClientBase.CONNECTION_TIMEOUT, (watchedEvent) -> {
                System.out.println(watchedEvent.getPath() + "触发了" + watchedEvent.getType() + "事件!" + "data:" + Naming.getData(watchedEvent.getPath()));
            }
            );

            // 创建根目录
            zk.create(ROOT_PATH, ROOT_PATH.getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
            // 创建子目录
            zk.create(CHILD_PATH, "childPath".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);

            // 创建对子目录的数据监听
            System.out.println(new String(zk.getData(CHILD_PATH, true, null)));

            // 修改子目录节点数据,观察根节点是否监听到
            zk.setData(CHILD_PATH, "modification".getBytes(), -1);

            zk.delete(CHILD_PATH, -1);

            // 删除父目录节点
            zk.delete(ROOT_PATH, -1);
            // 关闭连接
            zk.close();
        } catch (KeeperException e) {
            e.printStackTrace();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public static String getData(String path) {
        if (path == null) {
            return null;
        }
        try {
            return new String(zk.getData(path, false, null));
        } catch (KeeperException e) {
            e.printStackTrace();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return null;
    }
}
```
输出:
>null触发了None事件!data:null
>childPath
>**/root/childPath触发了NodeDataChanged事件!data:modification**

对比一下可以看出不同的操作触发不同的监听事件。

## 各种watch触发的情况总结
可以注册watcher的方法：getData、exists、getChildren。      可以触发watcher的方法：create、delete、setData。连接断开的情况下触发的watcher会丢失。
一个Watcher实例是一个回调函数，被回调一次后就被移除了。如果还需要关注数据的变化，需要再次注册watcher。
New ZooKeeper时注册的watcher叫default watcher，它不是一次性的，只对client的连接状态变化作出反应。

### 写操作与ZK内部产生的事件的对应关系
|                        | event For "/path"             | event For "/path/child"   |
| ---------------------- | ----------------------------- | ------------------------- |
| create("/path")        | EventType.NodeCreated         | 无                         |
| delete("/path")        | EventType.NodeDeleted         | 无                         |
| setData("/path")       | EventType.NodeDataChanged     | 无                         |
| create("/path/child")  | EventType.NodeChildrenChanged | EventType.NodeCreated     |
| delete("/path/child")  | EventType.NodeChildrenChanged | EventType.NodeDeleted     |
| setData("/path/child") | 无                             | EventType.NodeDataChanged |
### 事件类型与watcher的对应关系
| event For "/path"             | defaultWatcher | exists("/path") | getData("/path") | getChildren("/path") |
| ----------------------------- | -------------- | --------------- | ---------------- | -------------------- |
| EventType.None                | √              | √               | √                | √                    |
| EventType.NodeCreated         |                | √               | √                |                      |
| EventType.NodeDeleted         |                | √               | √                |                      |
| EventType.NodeDataChanged     |                | √               | √                |                      |
| EventType.NodeChildrenChanged |                |                 |                  | √                    |

### 写操作与watcher的对应关系


|                        | "/path" |         |             |        | "/path/child" |             |
| ---------------------- | ------- | ------- | ----------- | ------ | ------------- | ----------- |
|                        | exists  | getData | getChildren | exists | getData       | getChildren |
| create("/path")        | √       | √       |             |        |               |             |
| delete("/path")        | √       | √       | √           |        |               |             |
| setData("/path")       | √       | √       |             |        |               |             |
| create("/path/child")  |         |         | √           | √      | √             |             |
| delete("/path/child")  |         |         | √           | √      | √             | √           |
| setData("/path/child") |         |         |             | √      | √             |             |

**值得注意的是：getChildren("/path")监视/path的子节点，如果（/path）自己删了，也会触发NodeDeleted事件。**

### 永久监听



## 参考文档
https://zookeeper.apache.org/doc/trunk/zookeeperProgrammers.html
http://lixuguang.iteye.com/blog/2342721
https://www.ibm.com/developerworks/cn/opensource/os-cn-zookeeper/


