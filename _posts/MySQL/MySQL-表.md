---
title: MySQL-表
date: 2018-04-10 14:09:52
tags: DB
---
以InnoDB存储引擎表为例。

#### 索引组织表
**InnoDB存储引擎中，表都是根据主键顺序组织存放的，称为索引组织表（index organized table）。**
每张表都有个主键（Primary Key），如果建表时没有显式定义，则按如下顺序选择或创建：
* 是否有非空的唯一索引（Unique NOT NULL），如有，则选为主键。如果有多个，选择建表时第一个定义的非空唯一索引列；
* 自动创建一个6字节大小的指针。

#### 逻辑存储结构
所有数据逻辑地存放在一个名为表空间（tablespace）的地方。表空间由段（segment）、区（extent）、页（page）组成（页在一些文档中也成为块（block））。结构大致如下：
{% asset_img 表空间.PNG InnoDB逻辑存储结构 %}

##### 表空间
默认情况下所有数据存放在同一个共享表空间（名为 ibdata1）内。如果启用 innodb_file_per_table 参数，则每张表内的数据单独存放到一个表空间内。但即使启用该参数，每张表的表空间也只是用来存放数据、索引和入缓冲Bitmap页，其它类数据如回滚（undo）信息，插入缓冲索引页、系统事务信息、二次写缓冲（Double Write Buffer）等还是存放在原来的共享表空间内。所以共享表空间还是会不断增大。

##### 段（segment）
常见的段有数据段、索引段、回滚段等。数据段就是B+树的叶子节点（上图的 Leaf node segment），索引段就是B+树的非索引节点（上图的 Non-leaf node segment），回滚段见事务部分。对段的管理都由InnoDB存储引擎自身完成。

##### 区（extent）
由连续页组成的空间，在任何情况下每个区的大小都是1MB。为保证区中页的连续性，InnoDB一次从磁盘申请4~5个区。默认情况下，InnoDB页大小为16KB，即一个区中共有64个连续的页。InnoDB 1.2.x引入 innodb_page_size 参数（一次性设置，不可再次修改），可将默认页大小设置为4K、8K等。

##### 页（page） / 块（block）
InnoDB磁盘管理的最小单位。默认页大小为16KB。常见页类型：
* 数据页（B-tree Node）
* undo页（undo Log Page）
* 系统页（System Page）
* 事务数据页（Transaction system Page）
* 插入缓冲位图页（Insert Buffer Bitmap）
* 插入缓冲空闲列表页（Insert Buffer Free List）
* 未压缩的二进制大对象页（Uncompressed BLOB Page）
* 压缩的二进制大对象页（compressed BLOB Page）

##### 行
InnoDB存储引擎是面向行的（row-oriented），即数据是按行进行存放的。每个页存放的行记录最多为 16KB / 2 - 200，即7992条（？）。

#### 行记录格式
TBA

#### 数据页（B-tree Node）
TBA

#### 约束
* InnoDB提供以下几种约束：
* Primary Key
* Unique Key
* Foreign Key
* Default
* NOT NULL

两种创建方式：
* 表建立时就进行约束定义；
* 利用 ALTER TABLE 命令创建。

_**约束 vs 索引**_
Primary Key、Unique Key 也是创建索引的方法。当创建了一个唯一索引，也会同时自动创建一个唯一的约束。不同在于，约束更是一个逻辑的概念，用来保证数据的完整性；索引是一个数据结构，既有逻辑上的概念，在数据库中还代表着物理存储的方式。

**ENUM 和 SET 约束**
MySQL不支持传统的CHECK约束，但可以通过ENUM和ET类型解决这种约束需求。例如性别：
{% codeblock ENUM lang:sql  %}
mysql > CREATE TABLE a (
          -> id INT,
          -> sex ENUM('male', 'female'));
{% endcodeblock %}

**外键约束**
外键用来保证参照完整性。InnoDB完整支持外键约束（MyISAM不支持）。
一般称被引用的表为父表，引用表为子表。外键定义是的 ON DELETE 和 ON UPDATE 表示在对父表进行 DELETE 和 UPDATE 操作时，对子表所做的操作，有：
* CASCADE：对子表中的数据也进行相应操作（DELETE | UPDATE）；
* SET NULL：将子表中的数据更新为 NULL（对应列必须允许为 NULL）；
* NO ACTION：抛出错误，不允许这类操作发生；
* RESTRICT：同上，抛出错误，不允许这类操作发生（默认设置）。
