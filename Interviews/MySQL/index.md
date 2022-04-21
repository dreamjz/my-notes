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

   在一个事务中两次查询的数据不一致。

3. **幻读 (Phantom Read)**
   当用户读取某一范围的数据行时，另一事物又在该范围插入新行，再次读取时会发现出现“幻影”行。

### 7.2 MySQL 事务隔离级别

| 事务隔离级别     | 脏读 | 不可重复读 | 幻读 |
| ---------------- | ---- | ---------- | ---- |
| 读未提交         | Y    | Y          | Y    |
| 读已提交         | N    | Y          | Y    |
| 可重复读         | N    | N          | Y    |
| 串行化（序列化） | N    | N          | N    |

- **读未提交 (Read Uncommitted)**
  一个事务可以读取另一个未提交事务的数据。
  - 所有事务都可以看到其他事务的执行结果；
  - 此隔离级别使用较少；
  - 引发的问题为 **脏读 (Dirty Read)**：读取未提交的数据；
- **读已提交 (Read Committed)**
  一个事务等另一事物提交后才能提交数据，可以避免脏读。
  - 出现的问题是 **不可重复读 (Non-Repeatable Read)**；
- **可重复读 (Repeatable Read)**
  开始事务时，已读取的数据无法进行修改操作，可避免脏读和不可重复读。
  - MySQL 默认隔离级别；
  - 确保同一事务的多个实例在并发读取数据时，会看到同样的数据行；
  - 出现的问题是 **幻读 (Phantom Read)**：当事务读取某一范围的数据行之后，另一事物在范围内插入了新行，再次读取时出现了“幻影”行；
  - InnoDB 和 Falcon 通过 MVCC (Multiversion Concurrency Control, 多版本并发控制)机制解决该问题；
  - InnoDB 通过 MVCC 支持高并发，默认为可重复度，通过间隙锁 (Netx-key Locks) 策略防止幻读出现；
- 可串行化 (Serializable)
  事务串行化顺序执行，可避免脏读、不可重复度和幻读。
  - 强制事务排序，不可能冲突，在每个读数据上添加共享锁，解决幻读问题；
  - 会导致大量的超时现象和锁竞争；

## 8. utf8 和 utf8mb4

utf8mb4 中的 mb4 为 *most bytes 4*，专门用于兼容四字节的 unicode；

MySQL 中的 utf8 最多支持长度为 3 字节；

## 9. MySQL 乐观锁和悲观锁

- **悲观锁 (Pessimistic Lock)**
  每次获取数据时都会认为会被别人修改，所以在每次获取数据时都会上锁。
  适用于**写多读少**的场景；

- **乐观锁 (Optimistic Lock)**
  每次获取数据时，认为别人不会修改，所以不会上锁，但是在更新时会判断期间数据是否被更新，一般有两种方式：

  1. 使用数据版本 (Version) 记录机制。为数据增加一个版本标识，每次更新数据时则将 version 加一，当提交时，将第一次读取的 ver 和数据库当前的 ver 进行比较，若相同则继续更新；
  2. 增加控制字段，时间戳，在更新时比较时间戳；

   乐观锁适用于**读多写少**的场景。

  乐观锁一般流程如下：

  ```
  1. SELECT data AS old_data, version AS old_version FROM …;
  2. 根据获取的数据进行业务操作，得到new_data和new_version
  3. UPDATE SET data = new_data, version = new_version WHERE version = old_version
  if (updated row > 0) {
      // 乐观锁获取成功，操作完成
  } else {
      // 乐观锁获取失败，回滚并重试
  }
  ```

## 10. MVCC



## 11. 

