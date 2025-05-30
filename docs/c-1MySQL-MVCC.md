------

# MVCC

> 作者：hou
------
# 1.1 什么是MVCC

MVCC（Multi-Version Concurrency Control）是一种并发控制机制，用于在数据库管理系统中管理多个事务的并发访问。它通过为每个事务分配一个唯一的版本号（Snapshot），来实现对数据的并发读取和写入，从而避免了锁的开销。

# 1.2 相关概念

## 1.2.1 事务版本号

事务每次开启时，都会从数据库获得一个自增长的事务ID，可以从事务ID判断事务的执行先后顺序。这就是事务版本号。

## 1.2.2 隐藏列

在数据库表中，有两个隐藏列：

- trx_id：表示当前事务的ID。
- roll_pointer：指向当前事务的回滚记录。
- row_id：表示当前行的ID。(非必须)

## 1.2.3 undo log

undo log 是一种用于事务回滚的日志，它记录了事务对数据库的修改操作。当事务执行失败时，undo log 可以用来撤销已经执行的操作，从而保证数据库的一致性。

## 1.2.4 版本链

多个事务并行操作某一行数据时，不同事务对该行数据的修改会产生多个版本，然后通过回滚指针（roll_pointer），连成一个链表，这个链表就称为版本链

## 1.2.5 快照读和当前读

快照读：读取的是事务开始前的数据库状态，即事务开始时的快照。

当前读：读取的是事务开始后的数据库状态，即事务执行过程中对数据库的修改。

## 1.2.6 ReadView

ReadView 是 MVCC 的核心，它用于管理事务的可见性。

### 核心组件
1. **trx_ids（活跃事务列表）**
   - 定义：当前事务开始时，系统中未提交的事务ID列表
   - 作用：记录活跃事务ID，创建者在此列表中则不可见

2. **low_limit_id（下限ID）**
   - 定义：系统下一个将分配的新事务ID(next_trx_id)
   - 作用：此ID之后的事务都不可见

3. **up_limit_id（上限ID）**
   - 定义：trx_ids中的最小事务ID
   - 作用：此ID之前提交的事务都可见

4. **creator_trx_id（创建者事务ID）**
   - 定义：记录被哪个事务创建的事务ID
   - 作用：判断记录对当前事务是否可见的关键依据

### 可见性判断规则
- **自己修改的可见**  
  `trx_id == creator_trx_id` → 可见

- **已提交的老事务可见**  
  `trx_id < up_limit_id` → 可见

- **新事务不可见**  
  `trx_id >= low_limit_id` → 不可见

- **中间状态判断**  
  `up_limit_id <= trx_id < low_limit_id`时：
  - 在活跃列表`trx_ids`中 → 说明未提交，不可见
  - 不在活跃列表中 → 说明已提交，可见