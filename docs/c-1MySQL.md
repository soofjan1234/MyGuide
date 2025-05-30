------

# MySQL

> 作者：hou
------
## 1.1 MySQL基础

### 1.1.1  MySQL基础架构

1. **连接器**  
- 身份认证和权限相关（登录 MySQL 的时候）  

2. **查询缓存**  
- 执行查询语句的时候，会先查询缓存（MySQL 8.0 版本后移除，因为这个功能不太实用）  

3. **分析器**  
- 没有命中缓存的话，SQL 语句就会经过分析器，分析器说白了就是要先看你的 SQL 语句要干嘛，再检查你的 SQL 语句语法是否正确  

4. **优化器**  
- 按照 MySQL 认为最优的方案去执行  

5. **执行器**  
- 执行语句，然后从存储引擎返回数据。执行语句之前会先判断是否有权限，如果没有权限的话，就会报错  

6. **插件式存储引擎**  
- 主要负责数据的存储和读取，采用的是插件式架构，支持 InnoDB、MyISAM、Memory 等多种存储引擎。InnoDB 是 MySQL 的默认存储引擎，绝大部分场景使用 InnoDB 就是最好的选择  

### 1.1.2 MyISAM 和 InnoDB 的区别

1. **事务支持**  
2. **锁机制**  
3. **外键支持**  
4. **崩溃恢复**  
- MyISAM 不支持崩溃后恢复
- InnDB 的关键机制
  - **Write-Ahead Logging (WAL) 预写日志**：在数据写入磁盘之前，先将变更记录写入日志文件（Redo Log）。这样即使发生崩溃，也可以通过重放日志来恢复未完成的事务
  - **redo log**：重做日志，用于保证事务的持久性，防止数据丢失
  - **undo log**：回滚日志，用于保证事务的原子性和一致性  

5. **MVCC**   
6. **索引结构**  
- InnoDB 引擎中，其数据文件本身就是索引文件。相比 MyISAM，索引文件和数据文件是分离的，其表数据文件本身就是按 B+Tree 组织的一个索引结构，树的叶节点 data 域保存了完整的数据记录  

## 1.2 MySQL事务

### 1.2.1 并发事务带来的问题

1. **脏读**：第二个事务读到脏数据  

2. **丢失修改**：两个事务同时修改同一数据  

3. **不可重复读**：两次读取同一数据结果不一致  

4. **幻读**：不可重复读是修改，幻读是新增  

### 1.2.2 并发事务的控制方式

1. **锁机制**  
- 通过锁机制控制并发事务  

2. **乐观控制**  
- 使用版本号、时间戳等乐观控制机制  

3. **MVCC**  
- 多版本并发控制方法  
  - **多版本**：为每个数据项维护多个版本，允许事务在不同的极短点查看数据的不同状态  
  - **读一致性**：读取操作不会阻塞写入操作，反之亦然。每个事务可以看到数据在开始时的状态，而不是当前正在进行的修改  

### 1.2.3 事务隔离级别

1. **READ-UNCOMMITTED（读取未提交）**  
- 最低的隔离级别，允许读取尚未提交的数据变更，都可能导致  

2. **READ-COMMITTED（读取已提交）**  
- 允许读取并发事务已经提交的数据，可以阻止脏读，但是幻读或不可重复读仍有可能发生  

3. **REPEATABLE-READ（可重复读）**  
- 对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，可以阻止脏读和不可重复读，但幻读仍有可能发生
- MVCC（多版本并发控制）主要用于解决 不可重复读 和 部分幻读 问题，但它并不能完全解决所有幻读问题

4. **SERIALIZABLE（可串行化）**  
- 最高的隔离级别，完全服从 ACID 的隔离级别。所有的事务依次逐个执行，这样事务极短之间就完全不可能产生干扰，也就是说，该级别可以防止丢失修改、脏读、不可重复读以及幻读  

### 1.2.4 ACID特性

1. **Atomicity（原子性）**：事务要么成功提交，要么不提交  

2. **Consistency（一致性）**：执行事务前后，数据保持一致  

3. **Isolation（隔离性）**：事务不受其他事务干扰  

4. **Durability（持久性）**：事务提交后，改变是持久的  

## 1.3 MySQL索引

### 1.3.1 MySQL索引为什么不是二叉树？为什么不是平衡二叉树？ 

1. 二叉树特殊化为一个链表，相当于全表扫描，效率低
2. 平衡二叉树的高度和数据量有关，数据量越大，高度越大，查询效率越低
3. B+树减少磁盘IO

### 1.3.2 聚集索引和非聚集索引

1. **非聚集索引**  
- data 存放的数据记录的地址，然后根据地址读取相应的记录  

2. **聚集索引**  
- 数据文件就是索引文件，data 存放的 key 是数据库主键的地址  

3. **辅助索引极短**  
- data 存放的是主键的值，再走一遍主索引  

### 1.3.3 索引类型

1. **主键索引**  
- 唯一，不能为 null  

2. **唯一索引**  
- 大多是为了保证该列的数据唯一  

3. **覆盖索引**  
- 无需回表  

4. **联合索引**  
- 多个列组合的索引  

### 1.3.4 索引失效

1. **对索引列计算**   
2. **隐式转化**  
3. **like “%xx”**  
4. **or**    
5. **is null/is not null**   
6. **联合索引违反最左前缀**   
7. **不用索引更好**    

## 1.4 MySQL日志

### 1.4.1 MySQL的binlog

记录所有修改数据的操作  

1. **主从复制** 
2. **数据恢复**  
3. **审计** 

## 1.5 MySQL优化

### 1.5.1 如何知道要用索引

0. **基本步骤**
- 0.先运行看看是否真的很慢，注意设置SQL_NO_CACHE
- 1.where条件单表查，锁定最小返回记录表。这句话的意思是把查询语句的where都应用到表中返回的记录数最小的表开始查起，单表每个字段分别查询，看哪个字段的区分度最高
- 2.explain查看执行计划，是否与1预期一致（从锁定记录较少的表开始查询）
- 3.order by limit 形式的sql语句让排序的表优先查
- 4.了解业务方使用场景
- 5.加索引时参照建索引的几大原则
- 6.观察结果，不符合预期继续从0分析

1. **慢查询分析**  
- 有个函数出现了慢查询，然后就去 dao 层看，发现有三次查数据库的情况，我将他改为查一次数据库，时间减少很多。
  - 网络开销 ：每次查询都需要建立和关闭数据库连接，增加了网络开销。
  - 数据库负载 ：多次查询会增加数据库的负载，特别是在高并发场景下。  

2. **EXPLAIN 语句**  
- 使用 `EXPLAIN` 语句查看查询的执行计划，如果类型是 `ALL`，说明进行了全表扫描，通常表示需要索引。理想情况下，应该看到 `const`、`eq_ref` 或 `ref`  

3. **优化 SQL 查询**  
- 优化 SQL 查询，从 O(n) 降低到 O(log n)