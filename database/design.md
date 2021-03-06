# 数据仓库设计 
### [go back](/x2q/database/database)      
### [go home](/x2q)       
## 前言 
数据仓库一般针对 olap(在线分析处理)的业务,olap和oltp参考https://my.oschina.net/u/2969788/blog/2875200,重点用于处理大规模数据集的分析工作,通常的操作只有添加和查询,不会涉及到严格事务要求和实时的并发操作.
数据库的设计一般是针对 oltp业务,即在线事务处理,要求事务的一致性和实时性以及并发访问

## 数据仓库的设计的目的,为什么要分层和划分作用域
主要目的是为了对数据有一个更加清晰的掌控
+ 明确分层,主题的作用域,这样如果出了问题方便定位影响范围,
+ 解耦底层逻辑与应用层逻辑,可以简单 完整的提供业务数据给应用层使用
+ 减少开发工作量,避免重复工作,和同一指标多个口径
+ 清楚的管理数据血缘关系,发生问题方便追查

## 数据仓库分层
数据仓库有很多分层的方式,但是都大同小异, 分层的目的是为了让数据仓库的结构更清晰,也是通过分层来实现我们数据方库设计的目的,我们希望数据能够有层次,有规律的合理的分布在我们的磁盘上,并且能够优雅的应对各种业务场景的需求.

### 两种常见的层次划分
#### 第一种,数据明细和数据汇总层为上下游关系
![数仓分层1](/static/img/171700_3nUA_2969788.png)
#### 第二种,数据明细层和数据汇总层平行
![数仓分层2](/static/img/172123_F3sO_2969788.png)
具体选择,需要根据每个数据仓库的业务场景进行抉择,或者来说 没有严格意义上的分层,只是为了让逻辑跟清晰,如果dm 数据集市里面需要,数据明细层表中的数据,那么则将数据明细表开放给dm也无妨
### 各数据层简介
![图片](/static/img/174111_a3eL_2969788.png)

## 命名规则和存储格式
+ 命名规则建议统一使用,  层级_主题__表内容_分区规则  的命名方式
+ stg/ods/dwd/dws 存储格式 parquet,有利于MapReduce计算



## 相关概念
## 范式建模概述
+ 同一个数据只存一份,没有冗余,保证了数据一致性
+ 解耦系统级别和业务级别,方便维护
+ 开放成本高
## 范式建模步骤  自下而上(数据源->数仓->数据集市/dm)
+ 顶层抽象:一个高度的抽象模型,描述主要的**主题**和主题间的关系,用于描述企业的业务总体概况
+ 中层模型:在顶层抽象的基础上,细化**主题**的数据项
+ 物理模型:或者叫底层模型,针对中层明细,考虑物理存储,主要考虑基于的平台特点(是hdfs还是myql/oracle),存储性能和计算性能,可以做表的合并,分区,存储周期,存储格式,压缩格式等

### 数据库设计三范式 3nf 直白的说明
+ 每个属性值唯一，不具有多义性;**(列原子性,列为最小属性不可拆分)**
+ 每个非主属性必须完全依赖于整个主键，而非主键的一部分;**(行唯一性 一个表里面的每一行都是唯一不能重复)**
+ 每个非主属性不能依赖于其他关系中的属性，因为这样的话，这种属性应该归到其他关系中;**(数据库内不允许冗余字段(列),出来外键)**
范式建模是通过实体关系来描述业务
### 3范式组成基础
+ 实体:拥有相同属性和特征的逻辑的抽象
+ 属性:实体的某种特质
+ 关系:实体间的关系

## 维度-事实建模  自上而下 (数据集市->数仓->数据源)
+ 1.模型结构简单，星型模型为主；
+ 2.开发周期短，能够快速迭代；
+ 3.维护成本较高；
----------------------------------------------------------------------
+ 维度建模是另一种思想,主要是以分析决策为需求,触发构建模型,它不注重修改和删除,侧重点在于快速查找汇总,方便用户理解业务(直接吧使用方-可能是分析师,需要的数据都塞到一个表里面)
+ 每张维度表对应一个现实世界一个逻辑概念,如 客户,产品,地区编码,用户标签,就是码表


### 事实表和维度表
**事实表**是记录事实记录的表(md -_-! 写的有些绕口),通常包含很多行,每一列大多为客观数值或者维度表的外键,比如说 淘宝交易记录表,里面包含了 交易的时间 金额 数量 这些都是客观事实,
还有包括 用户账号 商品id 支付手段id 等这些都是维度表的外键  
  
**维度表** 反映的事物的维度,通常不会包含很多列,只会记录维度信息, 通常以dim前缀命名,维度表饱含了事实表中属性的详细信息,比如商品属性,客户的属性等等,
  
**星型模型和雪花模型** 通常在设计dm数据集市层的时候要考虑这两种模型    
**星型模型**是一种不完全符合3nf的模型,通常为一个事实表关联多个维度表,维度表不在关联其他维度表,好处是查询可以减少join,坏处是冗余了一部分数据,类似于下图
![图片](/static/img/7ae5b3a0720050177436aa229c57c1e424f.jpg)  
**雪花模型**通常是一张事实表会关联多张维度表但是维度表还可能关联其他维度表,这样好处是减少了数据冗余,坏处是查询的时候可能join表太多,雪花模型更符合3nf一些,但是二者的选择要结合具体业务来看,雪花模型如下图
![图片](/static/img/56ab2166d34435bad6fb7b4337d469f3c9c.jpg)  
**星系模型**:和星型模型很像,但是就是多个事实表可以共用一个维度表,减少维度表的个数,但是如果修改该维度表会带来较大的维护成本
### 宽表和窄表
**宽表**顾名思义就是一张非常宽的表,通常包含多个维度字段和很多事实数据,宽表通常是在数据汇总层dws,宽表设计要考虑到尽可能的覆盖业务范围,通常一张表要包含全部业务需要的汇总字段,宽表是对事实表的丰富,如果事实表包含汇总信息的话就叫做汇总表  
  
**宽表**不符合3nf,宽表的设计是为了服务于olap业务,尽量一张表满足全部查询需求,宽表一般只包含插入和查询操作,窄表不是为了oltp业务服务,要去符合3nf,通过表之间的关联反映业务逻辑  
### 维度建模流程
维度建模就是生成事实表和维度表的过程:
+ 选择业务:分析现实中的业务,抽象出事实表,
+ 确定粒度/维度表:确认改事实表的粒度,如比如消费账单,是一天一条还是一次一条,和维度,比如这个消费账单里面是否有用户id主键,如果有那么肯定有对应的用户维度表,如果有日期字段可能有日期的维度
如果有商品类型的id,就有商品类型的维度
















#### 联系邮箱 xxx_xxx@aliyun.com

