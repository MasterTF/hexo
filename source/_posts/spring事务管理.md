---
title: spring事务管理
date: 2016-11-23 22:35:35
tags:
    - Spring
    - 数据库
    - 事务
categories:
    - 技术
---
## 事务是什么
弄清spring事务管理机制，首先要弄清什么是事务。
人们创建了一个术语来表示事务：ACID。ACID代表四个特性，相信大家都很熟悉，但我也要贴出来。

> 原子性：一个事务（transaction）中的所有操作，要么全部完成，要么全部不完成，不会结束在中间某个环节。事务在执行过程中发生错误，会被回滚（Rollback）到事务开始前的状态，就像这个事务从来没有执行过一样。

> 一致性：在事务开始之前和事务结束以后，数据库的完整性没有被破坏。这表示写入的资料必须完全符合所有的预设规则，这包含资料的精确度、串联性以及后续数据库可以自发性地完成预定的工作。

> 隔离性：数据库允许多个并发事务同时对齐数据进行读写和修改的能力，隔离性可以防止多个事务并发执行时由于交叉执行而导致数据的不一致。事务隔离分为不同级别，包括读未提交（Read uncommitted）、读提交（read committed）、可重复读（repeatable read）和串行化（Serializable）。

> 持久性：事务处理结束后，对数据的修改就是永久的，即便系统故障也不会丢失。

（以上内容来自维基百科）

但ACID还是太过抽象，事务到底是怎么创建出来的？下面通过具体的代码来描述下。

<!-- more -->

## 如何开启事务
mysql中默认是自动提交事务的，可以通过

```sql
set autocommit = 0;
```
关闭自动提交。

或者通过

```sql
begin;
```
显示指定开启一个事务。

***这是什么意思呢？*** 
不特殊指定，**每条DML语句都是个事务**；若指定了begin，则遇到commit或ddl语句事务结束。
现在就很清楚了，原来事务是这样产生的，原来这就是一个事务。
那多个事务如何协同工作？有的事务是查找，有的是删除，根据acid特性，事务间是有隔离性的，那到底要隔离到什么程度呢？并发的事务如何相互影响？
这就牵扯出了另外一个经典话题——
## 事务隔离级别
> 可序列化(Serializable)
最高的隔离级别。在基于锁机制并发控制的DBMS实现可序列化，要求在选定对象上的读锁和写锁保持直到事务结束后才能释放。在SELECT 的查询中使用一个“WHERE”子句来描述一个范围时应该获得一个“范围锁(range-locks)”。这种机制可以避免“幻影读(phantom reads)”现象（详见下文）。当采用不基于锁的并发控制时不用获取锁。但当系统探测到几个并发事务有“写冲突”的时候，只有其中一个是允许提交的。这种机制的详细描述见“快照隔离”

> 可重复读(Repeatable reads)
在可重复读(REPEATABLE READS)隔离级别中，基于锁机制并发控制的DBMS需要对选定对象的读锁(read locks)和写锁(write locks)一直保持到事务结束，但不要求“范围锁(range-locks)”，因此可能会发生“幻读(phantom reads)”。

> 提交读(Read committed)
在提交读(READ COMMITTED)级别中，基于锁机制并发控制的DBMS需要对选定对象的写锁(write locks)一直保持到事务结束，但是读锁(read locks)在SELECT操作完成后马上释放（因此“不可重复读”现象可能会发生，见下面描述）。和前一种隔离级别一样，也不要求“范围锁(range-locks)”。

> 未提交读(Read uncommitted)
未提交读(READ UNCOMMITTED)是最低的隔离级别。允许脏读(dirty reads)，事务可以看到其他事务“尚未提交”的修改。


> 通过比低一级的隔离级别要求更多的限制，高一级的级别提供更强的隔离性。标准允许事务运行在更强的事务隔离级别上。(如在可重复读(REPEATABLE READS)隔离级别上执行提交读(READ COMMITTED)的事务是没有问题的)

（以上内容来自维基百科.)

不同的隔离级别并发时产生的结果也不一样。mysql中默认的隔离级别是 可重复读，在这个隔离级别下，同一个事务内多次读得到的结果是一样的，即使有其他事务删除了数据，仍不影响读事务的结果。

下面我们就以可重复读为例，看看事务是否是可重复读的。

### 验证可重复读
#### 在表中预置两条数据

| id | tenant_id | modify_time |
| --- | --- | --- |
| 4 | 2 | 2016-11-16 17:06:25 |
| 8 | 1 | 2016-11-16 17:06:25 |

#### 开启查询事务(事务A)

```sql
begin;
select * from tenant;
```
| id | tenant_id | modify_time |
| --- | --- | --- |
| 4 | 2 | 2016-11-16 17:06:25 |
| 8 | 1 | 2016-11-16 17:06:25 |
可以看到此时表中仍有两条数据。

#### 另启一个删除事务（事务B）
由于mysql每个dml语句都是自动提交的，这里我们直接写delete语句即可：

```sql
delete from tenant where tenant_id = 1;
select * from tenant;
```
| id | tenant_id | modify_time |
| --- | --- | --- |
| 4 | 2 | 2016-11-16 17:06:25 |
可以看到tenant_id为1的记录已经被删除，表中只剩下一条记录。

#### 回到事务A查看表中记录

还记得事务A第一次查询时有两条记录么？现在表中数据已经被删除了一条，按照“可重复读”的隔离级别，在事务A中再次执行select操作，看到的应该还是两条记录，执行下看看

```sql
select * from tenant;
```
| id | tenant_id | modify_time |
| --- | --- | --- |
| 4 | 2 | 2016-11-16 17:06:25 |
| 8 | 1 | 2016-11-16 17:06:25 |

果然，和第一次的select结果相同。

看到没，就是这么神奇。事务B虽然已经把tenant_id为1的记录删除了，但事务A仍然可以看到那条记录，就好像事务A完全不知道有事务B，丝毫没受事务B的影响。

这是“可重复读”隔离级别下的结果，大家可以去验证下“读以提交”的结果。

#### 提交事务A再查

```sql
commit;
select * from tenant_wechat_auth;
```

| id | tenant_id | modify_time |
| --- | --- | --- |
| 4 | 2 | 2016-11-16 17:06:25 |

提交之后事务结束，然后再次查询时就看不到已经被删除的记录了。
 
如此下来，不禁要问，mysql是如何做到的？
这里就引出了另外一个经典方案——

### MVCC
mysql给每个表隐含加了两列：创建版本号，结束版本号。
mysql每当新开启一个事务时，事务版本号会自动递增。事务开始时刻的系统版本号会作为事务的版本号，用来喝查询到的每行记录的版本号进行比较。
 
在“可重复读”隔离级别下的mcvv的具体操作：
> SELECT
InnoDB会根据以下两个条件检查每条记录：
a.InnoDB只查找版本遭遇当前事务版本的数据行（也就是，行的系统版本号小于或等于事务的系统版本号），这样可以确保事务读取的行，要么是在事务开始前已经存在的，要么是事务自身插入或者修改过的。
b.行的删除版本要么未定义，要么大于当前事务版本号。这样可以确保事务读取到的行，在事务开始之前未被删除。
INSERT
InnoDB为新插入的每一行保存当前系统版本号座位行版本号。
DELETE
InnoDB为删除的每一行保存当前系统版本号座位删除标识。
UPDATE
InnoDB为插入一行新纪录，保存当前系统版本号作为行版本号，同时保存当前系统版本号到原来的行作为删除标识。

（以上内容来自《高性能MYSQL》）

所以这就很好解释了我们的结果（结果自己分析下吧，看我写的分析过程的时间自己完全可以分析出来）。
## spring事务管理机制
前面弄清了什么是事务，下面就可以谈一下spring是如何管理事务了。

### spring中事务属性

```java
public interface TransactionDefinition{
    int getIsolationLevel();
    int getPropagationBehavior();
    int getTimeout();
    boolean isReadOnly();
}
```

也许你会奇怪，为什么接口只提供了获取属性的方法，而没有提供相关设置属性的方法。其实道理很简单，事务属性的设置完全是程序员控制的，因此程序员可以自定义任何设置属性的方法，而且保存属性的字段也没有任何要求。唯一的要求的是，Spring 进行事务操作的时候，通过调用以上接口提供的方法必须能够返回事务相关的属性取值。

### 事务隔离级别
隔离级别是指若干个并发的事务之间的隔离程度。TransactionDefinition 接口中定义了五个表示隔离级别的常量：

* TransactionDefinition.ISOLATION_DEFAULT：这是默认值，表示使用底层数据库的默认隔离级别。对大部分数据库而言，通常这值就是TransactionDefinition.ISOLATION_READ_COMMITTED。
* TransactionDefinition.ISOLATION_READ_UNCOMMITTED：该隔离级别表示一个事务可以读取另一个事务修改但还没有提交的数据。该级别不能防止脏读和不可重复读，因此很少使用该隔离级别。
* TransactionDefinition.ISOLATION_READ_COMMITTED：该隔离级别表示一个事务只能读取另一个事务已经提交的数据。该级别可以防止脏读，这也是大多数情况下的推荐值。
* TransactionDefinition.ISOLATION_REPEATABLE_READ：该隔离级别表示一个事务在整个过程中可以多次重复执行某个查询，并且每次返回的记录都相同。即使在多次查询之间有新增的数据满足该查询，这些新增的记录也会被忽略。该级别可以防止脏读和不可重复读。
* TransactionDefinition.ISOLATION_SERIALIZABLE：所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，该级别可以防止脏读、不可重复读以及幻读。但是这将严重影响程序的性能。通常情况下也不会用到该级别。

### 事务传播行为
所谓事务的传播行为是指，如果在开始当前事务之前，一个事务上下文已经存在，此时有若干选项可以指定一个事务性方法的执行行为。在TransactionDefinition定义中包括了如下几个表示传播行为的常量：

* TransactionDefinition.PROPAGATION_REQUIRED：如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。
* TransactionDefinition.PROPAGATION_REQUIRES_NEW：创建一个新的事务，如果当前存在事务，则把当前事务挂起。
* TransactionDefinition.PROPAGATION_SUPPORTS：如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。
* TransactionDefinition.PROPAGATION_NOT_SUPPORTED：以非事务方式运行，如果当前存在事务，则把当前事务挂起。
* TransactionDefinition.PROPAGATION_NEVER：以非事务方式运行，如果当前存在事务，则抛出异常。
* TransactionDefinition.PROPAGATION_MANDATORY：如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。
* TransactionDefinition.PROPAGATION_NESTED：如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务，则该取值等价于TransactionDefinition.PROPAGATION_REQUIRED。


这里需要指出的是，前面的六种事务传播行为是 Spring 从 EJB 中引入的，他们共享相同的概念。而 PROPAGATION_NESTED是 Spring 所特有的。以 PROPAGATION_NESTED 启动的事务内嵌于外部事务中（如果存在外部事务的话），此时，内嵌事务并不是一个独立的事务，它依赖于外部事务的存在，只有通过外部的事务提交，才能引起内部事务的提交，嵌套的子事务不能单独提交。如果熟悉 JDBC 中的保存点（SavePoint）的概念，那嵌套事务就很容易理解了，其实嵌套的子事务就是保存点的一个应用，一个事务中可以包括多个保存点，每一个嵌套子事务。另外，外部事务的回滚也会导致嵌套子事务的回滚。

### 事务超时
所谓事务超时，就是指一个事务所允许执行的最长时间，如果超过该时间限制但事务还没有完成，则自动回滚事务。在 TransactionDefinition 中以 int 的值来表示超时时间，其单位是秒。

### 事务的只读属性
事务的只读属性是指，对事务性资源进行只读操作或者是读写操作。所谓事务性资源就是指那些被事务管理的资源，比如数据源、 JMS 资源，以及自定义的事务性资源等等。如果确定只对事务性资源进行只读操作，那么我们可以将事务标志为只读的，以提高事务处理的性能。在 TransactionDefinition 中以 boolean 类型来表示该事务是否只读。
### 事务的回滚规则
通常情况下，如果在事务中抛出了未检查异常（继承自 RuntimeException 的异常），则默认将回滚事务。如果没有抛出任何异常，或者抛出了已检查异常，则仍然提交事务。这通常也是大多数开发者希望的处理方式，也是 EJB 中的默认处理方式。但是，我们可以根据需要人为控制事务在抛出某些未检查异常时任然提交事务，或者在抛出某些已检查异常时回滚事务。
## Spring 事务管理 API 分析
spring并不直接管理事务，而是提供了多种事务管理器，它们将事务管理职责委托给JTA或者其他其他持久化机制所提供的平台相关的事务实现。

Spring 框架中，涉及到事务管理的 API 大约有100个左右，其中最重要的有三个：TransactionDefinition、PlatformTransactionManager、TransactionStatus。所谓事务管理，其实就是“按照给定的事务规则来执行提交或者回滚操作”。“给定的事务规则”就是用 TransactionDefinition 表示的，“按照……来执行提交或者回滚操作”便是用 PlatformTransactionManager 来表示，而 TransactionStatus 用于表示一个运行着的事务的状态。
### TransactionDefinition
该接口在前面已经介绍过，它用于定义一个事务。它包含了事务的静态属性，比如：事务传播行为、超时时间等等。Spring 为我们提供了一个默认的实现类：DefaultTransactionDefinition，该类适用于大多数情况。如果该类不能满足需求，可以通过实现 TransactionDefinition 接口来实现自己的事务定义。
### PlatformTransactionManager
PlatformTransactionManager 用于执行具体的事务操作。接口定义如下：

```java
Public interface PlatformTransactionManager{
  TransactionStatus getTransaction(TransactionDefinition definition)
   throws TransactionException;
   void commit(TransactionStatus status)throws TransactionException;
   void rollback(TransactionStatus status)throws TransactionException;
}
```

根据底层所使用的不同的持久化 API 或框架，PlatformTransactionManager 的主要实现类大致如下：

* DataSourceTransactionManager：适用于使用JDBC和iBatis进行数据持久化操作的情况。
* HibernateTransactionManager：适用于使用Hibernate进行数据持久化操作的情况。
* JpaTransactionManager：适用于使用JPA进行数据持久化操作的情况。
* 另外还有JtaTransactionManager 、JdoTransactionManager、JmsTransactionManager等等。

如果我们使用JTA进行事务管理，我们可以通过 JNDI 和 Spring 的 JtaTransactionManager 来获取一个容器管理的 DataSource。JtaTransactionManager 不需要知道 DataSource 和其他特定的资源，因为它将使用容器提供的全局事务管理。而对于其他事务管理器，比如DataSourceTransactionManager，在定义时需要提供底层的数据源作为其属性，也就是 DataSource。与 HibernateTransactionManager 对应的是 SessionFactory，与 JpaTransactionManager 对应的是 EntityManagerFactory 等等。

各部分关系如下图：
![Spring事务配置](/img/Spring%E4%BA%8B%E5%8A%A1%E9%85%8D%E7%BD%AE.png)
### TransactionStatus
PlatformTransactionManager.getTransaction(…) 方法返回一个 TransactionStatus 对象。返回的TransactionStatus 对象可能代表一个新的或已经存在的事务（如果在当前调用堆栈有一个符合条件的事务）。TransactionStatus 接口提供了一个简单的控制事务执行和查询事务状态的方法。该接口定义如清单3所示：

```java
public  interface TransactionStatus{
   boolean isNewTransaction();
   void setRollbackOnly();
   boolean isRollbackOnly();
}
```
### 基于注解的声明式事务实现
目前这种方式是最流行的，上古的编程式事务并没有接触过，感兴趣可以上网搜下。

@Transactional 可以作用于接口、接口方法、类以及类方法上。当作用于类上时，该类的所有 public 方法将都具有该类型的事务属性，同时，我们也可以在方法级别使用该标注来覆盖类级别的定义。

```java
@Transactional(propagation = Propagation.REQUIRED)
    public boolean transfer(Long fromId， Long toId， double amount) {
        return bankDao.transfer(fromId， toId， amount);
}
```
Spring 使用 BeanPostProcessor 来处理 Bean 中的标注，因此我们需要在配置文件中作如下声明来激活该后处理 Bean:

```xml
<tx:annotation-driven transaction-manager="transactionManager"/>
```

与前面相似，transaction-manager 属性的默认值是 transactionManager，如果事务管理器 Bean 的名字即为该值，则可以省略该属性。

虽然 @Transactional 注解可以作用于接口、接口方法、类以及类方法上，但是 Spring 小组建议不要在接口或者接口方法上使用该注解，因为这只有在使用基于接口的代理时它才会生效。另外， @Transactional 注解应该只被应用到 public 方法上，这是由 Spring AOP 的本质决定的。如果你在 protected、private 或者默认可见性的方法上使用 @Transactional 注解，这将被忽略，也不会抛出任何异常。

