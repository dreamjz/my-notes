---
index: 'Index'
date: '2022-04-02'
publish: false
---

## 1. 事务的四个特性

ACID (Atomicity, Correspondence, Isolation, Durability)。

1. **原子性**（Atomicity)：事务中的操作，要么**全部成功**，要么**全部失败**。事务在执行过程中发生错误，就会回滚（Rollback）到事务开始前的状态，就像事务从未发生过一样。
2. **一致性** (Correspondence)：事务开始前和结束后，必须使数据库从一个一致性状态变换到另一个一致性状态，数据库的完整性约束没有被破坏。
   假设用户A和用户B两者的钱加起来一共是5000，那么不管A和B之间如何转账，转几次账，事务结束后两个用户的钱相加起来应该还得是5000，这就是事务的一致性。
3. **隔离性** (Isolation)：多个用户并发访问数据库时，每个事务之间不能相互干扰，由此引申出**隔离级别**。
4. **持久性** (Durability):  事务一旦被提交，对数据库的改动是永久性的。

## 2. 视图

视图是**虚拟**表，只包含查询的数据。

使用视图可以简化复杂的 SQL 操作。

视图不能被索引，也不能有关联的触发器或默认值，若视图内有 `order by` ，在使用视图时将会覆盖。

创建视图：

```sql
CREATE VIEW view_name AS sql_statement;
```

## 3. drop, delete, truncate

1. drop 直接删除表；
2. truncate 删除表中所有数据，相关的触发器和索引等会被删除，重新添加数据则 ID 从新开始；
3. delete 删除数据，但依赖的 约束，触发器和索引等不会被删除；
4. delete 属于 DML，可以回滚；drop 和 truncate 属于 DDL 不能回滚。
5. 执行效率：drop > truncate > delete。

## 4. JOIN



## 5. MySQL 索引

MySQL 索引使用 B+ 树，查询效率高（时间复杂度 *O(logN)* ），适配磁盘的页，尽量减少磁盘 IO 操作，适合**读多写少**的场景。

## 6. MySQL 存储引擎

| Name   | Decription |
| ------ | ---------- |
| InnoDB | 默认引擎   |

## 7. MySQL 事务隔离级别

### 7.1 事务并发的问题

1. **脏读 (Dirty Read)**
   一个事务读取了另一个事务未提交的数据。
   
2. **不可重复读 (Non-repeatable Read)**

   一个事务多次读取一个数据。

   



