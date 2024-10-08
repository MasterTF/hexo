---
title: UML和模式应用
date: 2021-03-16 17:32:43
tags: 
    - 面向对象
categories:
    - 技术
    - 系统设计
typora-root-url: ../../source
typora-copy-images-to: ../../source/img
---



> 什么是分析和设计
>分析和设计可以概括为：做正确的事(分析)和正确的做事(设计)。

![uml](/img/uml.png)

<!-- more -->

书的主要内容：

- UML和对象思想
- OOD的原则和模式：应该如何为对象分配职责？对象之间如何协作？针对这些问题，经过反复验证的解决方案已经被表示成为些最佳实践的原则、启示或模式，即问题-解决方案公示，这些公示是系统化的、典范的设计原则。
- 案例研究
- 用例：OOD与需求分析(requirements analysis)具有紧密联系，而在需求分析中通常包含用例的编写。
- 迭代开发、敏捷建模和敏捷UP



## 面向对象分析和设计
### 学习目标
OO开发中，至关重要的能力是为软件对象分配职责。掌握对象设计和职责分配的9项基本原则：GRASP

### 什么是分析和设计
分析和设计可以概括为：
**做正确的事(分析)和正确的做事(设计)**

- 分析：强调的是对问题和需求的调查研究，而不是解决方案。例如，如果需要一个在线交易系统，那么应该如何使用它？它应该具有哪些功能？
**“分析”一词含义广泛，最好加以限制，如需求分析(对需求的调查研究)或面向对象分析(对领域对象的调查研究)。**
- 设计：强调的是满足需求的概念上的解决方案(软件方面和硬件方面)，而不是其实现。例如，对数据库方案和软件对象的描述。设计思想通常排斥底层或“显而易见”的细节。最终，设计可以实现，而实现(如代码)则表达了真实和完整的设计。
与“分析”相同，“设计”一词也最好加以限制，如面向对象涉及或数据库设计。

### 什么是面向对象分析和设计
- 面向对象分析(OOA):强调的是在问题领域内发现和描述对象(概念)。例如，在航班信息系统里包含飞机(Plane)、航班(Flight)和飞行员(Pilot)等概念。
- 面向对象设计(OOD):强调的是定义软件对象以及他们如何协作以实现需求。例如软件对象Plane可以有tailNumber属性和getFlightHistory方法。
最后，会实现出来设计的对象，如Java中的Plane类。

### 领域模型
OOA关注从对象的角度创建领域描述。面向对象分析需要鉴别重要的概念、属性和关联。
面向对象分析的结果可以表示为领域模型(domain model)，在领域模型中展示重要的概念或对象。

需要注意：
**领域模型并不是对软件对象的描述，它使真实世界汇总的概念和想象可视化。因此也被称为概念对象模型(Conceptual Object Model).**

### 分配对象职责
OOD关注软件对象的定义——对象的协作和职责。序列图(sequence diagram)是描述协作的常见表示法。它展示出软件对象之间的消息流和有消息引起的方法调用。
序列图十分重要，消息的指向就是在进行职责分配，通过序列图可以很方便看出对象的职责。

序列图以动态的方式显示对象协作，还可以用 *设计类图*（design class diagram）来表示类定义的静态视图。这样可以描述类的属性和方法。

**领域模型表示的是真实世界的类，设计类图表示的是软件类。**
尽管设计类图不同于领域模型，但是其中的某些类名和内容还是相似的。从这一方面讲，OO设计和语言能够缩小软件构建和我们所设想的领域模型之间的差距，即实现 *低表示差距*(lower representiational gap) 

## 迭代、进化和敏捷
### 敏捷建模

> 建模的目的主要是为理解，而非文档。

建模能够对理解问题或解决方案提供更好的方式。实行OOA/D的目的并不是让设计者创建大量详细的UML图交给编程者,而是为良好的OOD快速探索可选的方案和途径。
### 什么是UP阶段

> up倡导的核心思想：短时间定量迭代、进化和可适应性开发。

1. 初始(Inception)：大体上的构想、业务案例、范围和模糊评估
2. 细化(Elaboration)：已精化的构想、和新架构的迭代实现、高风险的解决、确定大多数需求和范围以及进行更为实际的评估
3. 构造(Construction)：对遗留下来的风险较低和比较简单的元素进行迭代实现，准备部署。
4. 移交(Transition)：进行beta测试和部署
### UP科目
- 业务建模：领域模型制品，使应用领域中的重要概念的可视化。
- 需求：用以补货功能需求和非功能需求的用例模型机器补充性的规格说明制品。
- 设计：设计模型制品，用于对软件对象进行设计。
![](/img/16213446274566.jpg)


![](/img/16213446351071.jpg)



## OOA:领域模型
领域模型是OOA最重要的模型，阐述领域中的重要概念。
领域模型可以作为设计某些软件对象的灵感来源，也作为其他产出物的输入：
![](/img/16213446468416.jpg)

> 领域模型是对领域内的概念类或现实世界中对象的可视化表示[MO95,Fowler96].领域模型也成为概念模型、领域对象模型和分析对象模型。

在UP中，领域模型指的是对现实世界概念类的表示，而非软件对象的表示。该术语并不是指用来描述软件类、软件架构领域层或职责软件对象的一组图。

### 领域模型和数据模型是一回事吗？
领域模型不是数据模型。
在领域模型里，即使需求没要求记录相关信息的类 也可能出现在领域模型中。

### 为什么要创建领域模型
1. 面向对象的关键思想：领域层软件类的名称要源于领域模型中的名称，使对象具有源于领域的信息和职责。有助于减小人的思维与软件模型之间的差异。
![](/img/16213446617898.jpg)


2. 方便变更与扩展
3. 管理和隐藏复杂性

### 准则
#### 准则：如何创建领域模型
1. 寻找概念类
2. 将其绘制为UML类图中的类
3. 添加关联和属性

#### 准则：如何寻找概念类
1. 复用和修改已有的领域模型(最好的做法)
2. 使用分类列表：先分类，然后往对应的分类中添加名词。
3. 确定名词短语：从对领域的文本性描述(如用例、专业书籍、专家的想法等)中识别名次和名词短语，将其作为候选的概念类或属性。缺点在于自然语言不精确性，不同名词短语可能表示统一概念类或属性，还可能有歧义。建议与分类列表一起使用

#### 准则：使用领域术语
1. 以地图绘制者的思维创建领域模型，使用地域中现有名称。如图书馆模型，将顾客命名为“借书者”。
2. 排出无关或超出范围的特性。当前迭代没有的特性可以不在模型中表示。
3. 不要凭空增加事物

#### 准则：属性与类的常见错误
创建领域模型时最常见的错误是把应该是概念类的事物表示为属性。
避免这类错误的方法：
**如果我们认为某概念类X不是现实世界中的数字或文本,那么X可能是概念类而不是属性。**

#### 准则：何时使用“描述”类建模
描述类(description class) 包含描述其他事物的信息，如ProductDescription记录item的价格、图片和文字描述。
1. 需要有关商品或服务的描述，对于任何商品或服务的现有实例。
2. 删除其所有事物的实例后，导致信息丢失，而这些信息是需要维护的。
3. 减少冗余或重复信息

#### 准则：有下列情况时创建子类；
1. 子类具有我们感兴趣的额外属性
2. 子类具有我们感兴趣的额外关联
3. 对子类的影响、处理、反应和操作与超类或其他子类有显著差异。
4. 子类概念表示了一个活动体(如动物、机器人等),其行为与超类或者其他子类不同，而这些行为是我们所关注的。

#### 准则：何时定义超类
1. 潜在的概念子类表示的是相似概念的不同变体。
2. 子类满足100%和Is-a规则。
3. 所有子类都具有相同的属性，可以放在超类中表达。
4. 所有子类都具有相同的关联，可以剑气解析出来与超类关联。

#### 准则：不要讲概念X的状态建模为X的子类
有两种可选方案：
1. 新建 状态类，并与X关联
2. 领域模型中忽略状态的概念，在状态图中显示。
![](/img/16213447211081.jpg)

#### 准则：在领域模型中增加关联类的可能线索
1. 有某个属性与关联相关
2. 关联类的实例具有依赖于关联的生命周期
3. 两个概念之间有多对多关联，并且存在与关联自身相关的信息。

#### 准则：下列情形可以使用组合关系
1. 部分的生命期在组合的生命期界限之内，部分的创建和删除依赖于整理。
2. 在屋里或者逻辑组装上，整体-部分关系和明确。
3. 组合的某些属性(如位置)会传递给部分。
4. 对组合的操作(如销毁，移动和记录等)可能传递给部分。

组合关系意味着：
- 在某一时刻，部分的实例只属于某一个组合实例
- 部分必须总是属于组合。
- 组合要负责创建和删除其部分。

### 关联
关联(association)是类之间的关系，表示有意义和指的关注的连接。

#### 准则：何时表示关联
关联表示了需要持续一旦时间的关系，根据语境，可能是毫秒或数年。需要记录哪些对象之间的关系？
领域模型是从概念角度出发的，所以是否需要记录关联，要基于现实世界的需要而不是软件的需要。

#### 准则：为什么应该避免加入大量关联
关联太多会产生“视觉干扰”，使图变得混乱。重点关注“需要记住”的关联

#### 观点：关联是否会在软件中实现
关联不是关于数据流、数据库外键联系、实例变量或软件方案中的对象连接的语句；关联声明的是针对现实，领域从纯概念角度看有意义的关系。
这些关系大部分将作为设计模型和数据模型中的导航和可见性路径在软件中实现。但是领域模型不是数据模型，添加关联是为了突出我们对重要关系的大致理解，而非记录对象或数据的结构。

### 多重性
多重性(multiplicity)定义了类A有多少个实例可以和类B的一个实例关联。
两个类可能有多重关联，应当分别表示。如航班和机场的关系，一个机场有多个航班，但可以继续区分为飞来的航班，和离开的航班
![](/img/16213447369902.jpg)

### 属性
属性(attribute)是对象的逻辑数据值

#### 准则：何时展示属性
当需求建议或暗示需要记住信息时，引入属性。

#### 准则：什么样的属性是适当的
1. 关注领域模型中的数据类型属性：大部分应该是“简单”数据类型，如数字、布尔等。通常，属性的类型不应该是复杂的领域概念。
2. 通过关联而不是属性表达概念之间的关系
3. 领域模型中建议属性主要是基本数据类型，并不意味着java对象中的属性也是基本数据类型。因为领域模型是概念透视图，不是软件透视图。在设计模型中，属性可以是任何类型。

#### 准则：何时定义新的数据类型
需要把最初认为是数字或字符串的数据类型表示为新的数据类型类：
- 有不同的小节组成，如电话号码、人名
- 具有与之相关的操作，例如解析或校验。如社会安全号
- 具有其他属性。如促销价格有开始日期和结束日期
- 单位的数量。如支付金额有货币单位
- 具有以上性质的一个或多个类型的抽象。

#### 准则：任何属性都不表示外键
- 领域模型中的属性不应该用于表示概念类的关系。
- 使用关联而不是属性将类型关联起来。

#### 准则：对数量和单位建模
大部分用数字表示的数量不应该表示为纯数字。比如价格是13，重量是37，并不能说明是元还是千克。
应当给这些数量加上单位，并且还需要知道单位之间的转换关系。

### 结论：领域模型是否正确
没有所谓唯一正确的领域模型。所有模型都是对试图要理解的领域的近似。领域模型主要是在特定群体中用于理解和沟通的工具。
**有效的领域模型捕获了当前需求语境下的本质抽象和理解领域所需要的信息，并且可以帮助人们理解领域的概念、术语和关系。**

## OOD

### 架构
- 逻辑架构(logical architecture):软件类的宏观组织结构，将软件类组织为包(或命名为空间)、子系统和层等。之所以称其为逻辑架构，是因为并未决定如何在不同的操作系统进程或网络中物理的计算机上对这些元素进行部署。
- 软件架构：架构是一组重要决策，其中设计软件系统的组织，对结构元素及其组成系统所籍接口的选择，这些元素特定于其相互协作的行为，这些结构和行为元素到规模更大的子系统的组成，以及指导该组织结构(这些元素及其接口、协作和组织)的架构风格。

不管是哪种定义，所有软件架构定义的共同主题是，必须与宏观事物有关——动机、约束、组织、模式、职责和系统指连接(或系统之系统)的重要思想。

- 层(layer)：对类、包或子系统的粗粒度分组，具有对系统主要方面加以内聚的职责。"较高"层可以调用"较低"层，反之不能。OO系统中常见的分层：
	- 用户界面
	- 应用逻辑和软件对象
	- 技术服务：提供支持性技术服务的常用对象和子系统，例如蜀葵恶寇或错误日志。这些服务通常是独立于应用的，也可在多个系统复用。
	![](/img/16213447510519.jpg)


- 分层架构解决的问题：
	- 大部分系统是高度耦合的，代码变更会波及整个系统
	- 应用逻辑与用户界面耦合，应用逻辑部分无法复用
	- 一般性技术服务与应用逻辑耦合，不能复用
	- 不同关注领域之间的耦合，难以为不同开发人员清晰地界定和分配任务

### 设计对象
对象模型有两种类型：动态和静态。
- 动态建模有助于设计逻辑、代码行为或方法体，使用顺序图。
- 静态建模有助于设计包、类名、属性和方法特征标记的定义，使用类图。
- 两者的关系：并行进行，花费较短的时间创建顺序图，然后转到对应的类图，交替进行。
- 应当把时间花在序列图上，而不仅是类图。

### 序列图
不展开讲

### 类图

**依赖**
UML中的普通依赖表示：客户(client)元素（任何种类，包括类、包、用例等）了解其他的提供者元素（supplier）,并且表示党提供者有所改变时会对客户产生影响。
依赖关系比较常见的类型：
1. 拥有提供者类型的属性
2. 向提供者发送消息
3. 接收提供者类型的参数
4. 提供者是超类或接口
所有这些类型在UML中都可以用依赖线表示。但其中有些类型已经有了暗示依赖的特殊线条表示法。如表示超类的特殊UML线、表示接口实现的线、表示属性的线。所以这些情形不需要使用依赖线。

何时表示依赖？
**在类图中，使用依赖线描述对象之间的全局变量、参数变量、局部变量和静态方法(对其他类的静态方法加以调用)的依赖**

**组合优于聚合**
组合关系有几层含义：
1. 在某一时刻，部分的实例只属于一个组成实例
2. 部分必须总是属于组成
3. 组成要负责创建和删除其部分，既可以自己来创建/删除部分，也可以与其他对象协作来创建/删除部分。与该约束相关的是，如果组成被销毁，部分也必须被销毁，或者依附于其他组成。


## OOA与OOD的产出物
**OOA产出物**
 - 领域模型
 - 系统顺序图SSD(感觉是业务序列图)
 - 操作契约(用例) P133
 - 状态模型 P133

**OOD产出物**
- 逻辑架构（包图）
- 对象序列图
- 类图
- UI的草图和原型
- 数据库模型
- 报表的草图和原型
![](/img/16213448515627.jpg)


## GRASP:基于职责设计对象
OOD之前的输入：
![](/img/16213447921708.jpg)

- 用例
- 序列图
- 领域模型


RDD(职责驱动设计):把软件对象想象成具有某种职责的人，他要与其他人协作以完成工作。RDD使我们把OO设计看作是有职责对象进行协作的共同体。

GRASP对一些基本的职责分配原则进行了命名和描述，因此掌握这些原则有助于支持RDD。

如何给对象分配职责？绘制序列图就是在分配职责。GRASP指导分配职责时可做的选择。

职责分为两种：行为和认知

### 行为职责
1. 自身执行一些行为，如创建对象或计算
2. 初始化其他对象中的动作
3. 控制和协调其他对象中的活动

### 认知职责
1. 对私有封装数据的认知
2. 对相关对象的认知
3. 对其能够导出或计算的事物的认知

领域模型描述领域对象的属性和关联，通常产生与“认知”相关的职责。

### 创建者
问题：谁创建对象A
方案：
1. B“包含”或组成聚集了A
2. B记录A
3. B紧密地使用A
4. B具有A的初始化数据
以上条件之一为真(越多越好),将创建职责分配给B。

### 信息专家
问题：给对象分配职责的基本原则是什么
方案：把职责分配给具有完成该职责所需信息的那个类。

### 控制器
问题：首先接收UI层消息、协调系统操作的对象是什么？
方案：把职责分配给能代表下列选择之一的对象：
1. 代表全部“系统”、“根对象”、运行软件的设备或主要的子系统（外观控制器(facade controller)的变体）
2. 代表发生系统操作的用例场景
正常情况下，控制器应当把需要完成的工作委派给其他对象。控制器知识协调或控制这些活动，本身并不完成大量工作。

### 低耦合
问题：怎样降低依赖性，减少变化带来的影响
方案：分配职责，是耦合性尽可能低。利用这一原则来评估可选方案。

### 高内聚
问题：怎样保持对象是有重点的、可理解的、可管理的，并且能够支持低耦合？
内聚是对元素职责的相关性和集中度的度量。如果元素具有高度相关的职责，而且没有过多工作，那么该元素具有高内聚性。
方案：分配职责可保持较高的内聚性，利用这点来评估候选方案。
内聚性低会导致的问题：
1. 难以理解
2. 难以服用
3. 难以维护
4. 错误，经常会受到拜年话影响。
内聚性低的类通常表示大粒度的抽象，或承担了本应委托给其他对象的职责。

### 多态
问题：如何处理基于类型的选择？如何创建可插拔的软件构件？
方案：使用多态操作为变化的行为类型分配职责。

### 纯虚构
问题：不想违背高内聚和低耦合或其他目标，但是基于专家模式所提供的方案又不合适时，哪些对象应该承担这一职责？
方案：对人为制造的类分配一组高内聚的职责，该类并不代表问题领域的概念——虚构的事务，泳衣支持高内聚、低耦合。
例子：order对象存到db，按照专家模式，order对象应该负责与db的连接。但不满足高内聚、低耦合，引入虚构的dao对象负责存储。

### 间接性
问题：为了避免两个或多个事物之间直接耦合，应该如何分配职责？如何使对象解耦合，以支持低耦合并提高复用性？
方案：将职责分配给中介对象，使其作为其他构件或服务之间的媒介，以避免他们直接耦合。

### 防止异变
问题：如何设计对象、子系统和系统，使其内部的变化或不稳定性不会对其他元素产生不良影响？
方案：识别预计变化或不稳定之处，分配职责用以在这些变化之外创建稳定接口。“接口”指的是广泛意义上的访问视图，而不仅仅是诸如Java接口等字面含义。

