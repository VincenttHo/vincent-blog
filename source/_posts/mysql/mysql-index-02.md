---
title: mysql索引原理02--存储引擎索引的实现
date: 2019-09-25 22:21:38
tags: [mysql,索引]
categories: 技术
---
# 前情回顾
&emsp;&emsp;上一集讲到各种数据结构，以及解析了mysql为什么选用b+树作为索引的数据结构。这一集来讲一下mysql中索引在存储引擎中是如何实现的。这一篇过去以后，就能对整个索引有一个深刻的理解。
&emsp;&emsp;mysql有两个存储引擎：InnoDB和MyISAM。下面就来看一下这两个存储引擎里，索引都是如何实现的。

# MyISAM
&emsp;&emsp;MyISAM存储引擎是一个不支持事务的表级锁的存储引擎。它存储结构的特性是，索引以及数据是分开的。下面讲一个例子来说明一下：
&emsp;&emsp;我们在mysql中建立一个库demo_database下建立一个表t_test_myisam，将存储引擎设置成MyISAM
{% asset_img myisam1.jpg myisam示例1 %}
&emsp;&emsp;然后在mysql目录下的/data/demo_database可以看到mysql建立了两个文件。分别是t_test_myisam.MYD和t_test_myisam.MYI
{% asset_img myisam2.jpg myisam示例2 %}
&emsp;&emsp;上面我们说了，MyISAM存储引擎存储的特性就是将索引和数据分开。没错，这两个文件，，一个是保存索引的，一个是保存数据的。
&emsp;&emsp;.MYI文件是索引文件，上一篇文章中介绍的索引的树状结构就是保存在这一个文件里面。
&emsp;&emsp;.MYD文件是数据文件，住要保存的是数据表里的数据。
&emsp;&emsp;如下图的结构，树状结构部分是MYI文件保存的索引部分，右边的表格数据内容则是MYD文件保存的部分，索引的叶子节点带有一个指针，指向MYD文件中所保存的表数据。
{% asset_img myisam3.png myisam示例3 %}

# InnoDB
&emsp;&emsp;InnoDB是一个支持事务而且支持行级锁的存储引擎。它的存储特性是将索引和数据储存在一起的。
&emsp;&emsp;我们继续在mysql中建立一个库demo_database下建立一个新的表，命名为t_test_innodb，将存储引擎设置成InnoDB
{% asset_img innodb1.jpg InnoDB示例1 %}
&emsp;&emsp;然后在mysql目录下的/data/demo_database可以看到mysql建立了1个文件：t_test_innodb.idb
{% asset_img innodb2.jpg InnoDB示例2 %}
&emsp;&emsp;很明显的，这个文件就是InnoDB的存储文件，这个文件既保存了索引也保存了表数据。它的结构可以看下图：
{% asset_img innodb3.jpg InnoDB示例3 %}
&emsp;&emsp;大家可以发现，本来在MyISAM中叶子节点保存指针的地方变成了保存数据信息。和就是InnoDB在储存时的特性。而这个索引保存方式，也被称为聚集索引。
## InnoDB引发思考
&emsp;&emsp;InnoDB是鼓励数据表是一定要建立一个主键的，而且最好是整型的自增主键。为什么要这样限制呢？我们可以从InnoDB的存储结构思考。
1.一定要建立主键。
根据上面的了解，我们现在知道InnoDB将索引和数据存储在一起的。那么可以想一下，假如没有主键，那么是不是数据表就没有了主键索引，那么我们数据会保存在哪里？这样一想是不是就能知道一定要建立主键的原因了。
2.主键要用整型自增主键
这个其实是和查询有关系的，做个比喻，有好一些业务系统中，主键都是用uuid的，这种做法的坏处是，假如我们不管是查询，还是插入修改等操作，在根据主键搜索的时候，在各个索引值比对的时候会存在一个问题，就是字符串比对得转换成ASCII码进行比对，这样效率很明显是降低的。这也就是我们提倡使用整型自增主键的原因，不管是插入，还是查询，效率都会比较高。