#dbx 开发文档
```
    应cctv云媒资项目升级过程的数据转移需要，规划构建由老数据转移为新系统结构定义下的规范数据。由于之前的数据迁移工具dataant存在
 转移方向单一、数据处理简单、类型匹配不合理、数据merge无法扩展等诸多局限；决心开发一款相对而言，移植处理逻辑更加规范、适用性更加
 广泛、配置扩展更加方便的统一的普适性数据移植软件。

```

## 一、设计思路

```$xslt
    构建之初考虑，数据移植在应用迭代过程中的位置特殊，就大型软件而言，尤其是在面向的客户群体多元化的项目中，数据仓库的选型，产品
 升级带来的数据方案的更新是无法避免的。无论客户是喜欢MS-SqlServer、Oracle，还是更具潜力的MySQL等开源数据库，对客户而言的都是可以
 理解的，对软件提供商而言，应该综合到所有软件客户的需求，尽最大努力完成不同方案的融合并及时给出合理的产品方案。
    dbx软件在设计上考虑了MS-SqlServer、Oracle、MySQL 多种数据方案，以两两可互逆的体系规划软件的多向通道；这些比之之前的一些数据
 迁移工具来言算是较为明显的优化与融合。
    另外，dbx的数据移植逻辑是建立在科学性的方法逻辑和数据规则上的，尤其是在数据类型校验、转化方面，参考了各DB提供商的数据类型定
 义，力求在软件基础上更具科学性和依据性，以期保证释出产品的质量。
    
```
```$xslt
   因为数据移植不可避免的存在跨库、跨数据平台的问题，在移植逻辑中必然要建立数据校验、数据适配、数据映射、数据修复的诸多核心模块，
 在下文中就这些主要概=概念和相关重要逻辑给出尽可能详细的描述。
```
### 设计要点
+ 跨库移植
+ 数据移植原则
+ 数据Merge
+ 类型转换与DDL生成

#### 1、跨库移植
```$xslt
    产品迭代在一定阶段，不仅需要会在产品设计上形成巨大改变；更直接的反映在数据上。在迭代过程中，数据方案的整体更改，在我们的应用
 中，经常性的表现出来；为了适用新的产品，建立新的数据结构是常见的方案。然而对于绝大多数用户而言，数据在产品及结构迭代过程中是不允
 许丢失的；所以数据移植的常规表现是将老数据从之前的库中依据一定的规则植入新产品的库中。
    数据移植是一个跨库移植的思路，其性质体现为两点。
    一是，数据从原数据库移植到目标数据库，数据流已不在单独的数据源内部进行，应用内部需要维护两个或多个数据连接。
    二是，应用面向的数据库产品是多元的，保证应用的可用性上支持跨平台跨数据库产品。比如，目前dbx应用内部建立了完善的数据库类型转
 换机制，支持MS-SqlServer、Oracle、MySQL各数据库产品之间互相进行数据的移植。
    
```

#### 2、数据移植原则
```$xslt
    两个原则。数据移植过程需要遵守的两个基本规则是“勿要污染，勿要损失”。
    勿要污染。污染是指，由于新数据的进入，导致原本良好的环境被破坏。数据移植处理首要的规则就是严格禁止导致这现象产生的操作存在。
 比如，在数据移植处理的逻辑中，要避免程序自我的修改新的数据结构。
    勿要损失。在可工作的现有数据之上植入新数据，需保证新数据入库过程不会损失现有数据，类似幂等规则，现有环境不因新数据的进入发生
 异常。其次，目标数据的移植过程中呈现事务特性，正确完整的一次数据移植，不会存在数据的丢失。
```
#### 3、数据Merge
```$xstl
   数据Merge是面向数据结构设计上存在巨大差异而言的。
   比如，在老的媒资系统向CRE系统升迁，很早之前的CRE产品向目前CRE产品同步，以及其他企业的融合媒体产品向CRE同步时；为保证客户已有
 数据不丢失，并确保CRE产品环境正常运行，通常需要做整个数据产品库的适配和移植。目前dbx是支持这样的场景的，即，基于简单的配置，即
 可完成整个产品迭代过程的数据移植。
    那么数据的Merge数据体现在？
    一是，对数据表存在差异时的数据映射，dbx支持原数据表和目标数据表存在巨大差异的情况，可以通过相关配置为不同column做适配映射，
 以及植入新的列。
    二是，数据的转义，该场景考虑是不同产品中数据含义在定义时存在差异的情况。
    比如，源数据库表的某个TYPE值的含义在目标产品数据库中发生了变化。为保证产品系统的可用性，这种情况就需要考虑进去。然而，这部分
 在最初的设计中，就考虑不在应用内部做；因为设计之初，就打算将dbx作为一款使用性较高的通用数据移植软件，所以将这种定制化较高的转义
 逻辑考虑在应用外部。
    具体实践上，可以考虑以脚本修复数据的方式来完成数据转义过程。（比如，当前的CCTV移植后的数据转义就是通过发布脚本的方式；相关
    转义逻辑核对以及脚本都有具体的文件备份，可参考documents目录中的相关文件）。
```

#### 4、类型转换与DDL生成
```$xstl
    前面提到，dbx是一款跨平台跨数据库产品的数据移植应用。
    熟悉MS-SqlServer、Oracle、MySQL，以及有过这些数据库产品开发经验的同学都该知道，各大数据库产品的实现上是存在巨大差异的，比如
 事务管理、数据库锁、索引实现、分布式协调等等，每一款优秀的数据库产品背后都有着可以挖掘和值得推敲的原理和机制。
    当然，面向数据移植过程的应用开发，我们不需要一一挖掘这些优秀的设计。但是最基本的数据类型的定义是一定要搞清楚的。比如MySQL、
 Oracle他们的每一种数据类型定义的具体含义是需要推敲的。
    个人认为，dbx开发中一个值得尊重的地方是，对各大数据库产品的数据类型定义的转换。
    该部分，自己是仔细阅览了个数据库提供商的官方文档推导出的可用的数据类型的对应关系，有心者可以再行推敲。
    
    DDL-generator是dbx自建的DDL语法生成模块，构建与类型转换之上。
    该模块可以依据已有的源数据库的表结构定义，生成目标数据库的DDL语法（跨数据库产品）。
```

## 二、开发方案
```$xstl
    dbx设计思路，基本如上。
    具体实现其实算是较简单和清晰。
    1、使用SpringBoot代替传统Spring配置管理，用极少的配置完成对整个web应用结构的搭建。
    2、使用原始的JDBC作为持久层操纵工具，jdbc整体应用也是比较简单，基本做java开发的都是比较熟悉的。
    3、封装了完整的运行时环境，包括MS-SqlServer、Oracle、MySQL的JDBC驱动程序等。
    4、采用JSON、YAML、Properties等文本格式承载应用的配置，具体语法也比较实用，都有官方文档可参考具体语法。
    5、遵循简易部署的风格，应用内嵌Jetty web容器，构建成一个可执行jar的方式，并提供win-batch，linux-shell等多种启动脚本，极简
        化了实施运维人员的工作。
```

## 三、运行时环境
```$xslt
    该工具的释出文件以可执行jar文件发布（内嵌Jetty Web容器），只需要依赖JRE即可执行（修正版本后，可在Java7及以上版本的java环境执行）；
    考虑软件的轻巧且面向工程师用户，并未将JRE 运行时环境一起封装。
```