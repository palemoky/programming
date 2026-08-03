# MySQL

## 索引

- 聚簇索引：索引与数据放在一起，找到索引就拿到了数据
- 非聚簇索引：索引与数据分别存放，找到索引后还需要根据主键回表查询
- 覆盖索引：索引中包含查询所需字段，查询时无需回表

最左匹配失效的情况：
- 使用 `!=`, `or`等
- `like '%...`
- 字符串不加引号
- join 字段类型不同，造成隐式转换

MySQL InnoDB采用B+树作为索引，是因为：

1. **磁盘 IO 友好**：B+树的非叶子节点只存储索引和指针（叶子节点存放数据），单个节点能容纳更多索引项，树的高度更低（通常 3~4 层即可覆盖千万级数据），每次查询所需的磁盘 IO 次数更少
2. **天然支持范围查询**：B+树的叶子节点通过双向链表连接，范围查询（`BETWEEN`、`>`、`ORDER BY` 等）只需定位到起始叶子节点后顺序遍历，效率远高于 B 树需要中序遍历整棵树
3. **查询性能稳定**：所有数据都存储在叶子节点，任何查询都要走从根到叶子的完整路径，时间复杂度稳定为 O(log n)；而 B 树的数据分布在所有节点，不同记录的查询深度不一致
4. **相比哈希索引**：哈希索引虽然等值查询 O(1)，但不支持范围查询和排序，也无法利用最左前缀匹配
5. **相比跳表**：跳表在内存中表现优异（如 Redis），但其指针结构不利于磁盘顺序读取，无法充分利用操作系统的预读机制

!!! note "页分裂（Page Split）"

    InnoDB 以**页（Page，默认 16KB）**为最小 IO 单位，每个 B+ 树叶子节点对应一个数据页，页内记录按主键顺序排列。

    当向一个已满的数据页中插入新记录时，InnoDB 会将该页的一部分数据搬移到新分配的页中，并调整父节点指针，这个过程就是**页分裂**。页分裂会导致：

    - 额外的磁盘 IO（分配新页、写入、更新父节点）
    - 页空间利用率下降（分裂后每个页约半满）
    - 大量分裂时产生碎片，影响范围查询的顺序读性能

    **如何减少页分裂**：使用自增主键（`AUTO_INCREMENT`）作为聚簇索引，保证数据总是追加到最后一个数据页，避免中间插入导致的分裂。这也是不推荐使用 UUID 作为主键的主要原因——UUID 的随机性会导致频繁的随机插入和页分裂。

## 锁

- 记录锁：锁定指定的数据
- 间隙锁：锁定指定数据的范围区间
- 临键锁（默认）：记录锁+间隙锁

MySQL默认的隔离级别是RR（可重复读），通过临键锁解决了RR下的幻读问题。

## 日志

- undo log：记录事务的反向SQL，支持事务回滚、MVCC版本链，事务的原子性、一致性
- redo log（WAL）：避免崩溃导致数据丢失，事务的持久性
- bin log：主从同步，有statement、mixed、row三种模式

> MVCC 支持了事务的隔离性

## 并发操作金额

在并发场景下对同一账户进行余额扣减、充值等操作时，核心问题是**更新覆盖**（Lost Update）：线程 A 和 B 分别读取到相同余额后各自计算并写回，导致其中一次扣款被另一次覆盖。以下介绍三种常见的解决方案。

### 1. 悲观锁：SELECT ... FOR UPDATE

通过 `FOR UPDATE` 对目标行加排他锁（X 锁），在事务提交前阻止其他事务对该行的并发修改。

```sql
-- 1. 显式开启事务
BEGIN;

-- 2. 查询并加排他锁（锁会持有至 COMMIT / ROLLBACK）
SELECT balance FROM wallet WHERE user_id = 1 FOR UPDATE;

-- 3. 在应用层进行业务校验：
-- if (balance >= deductAmount) { ... } else { ROLLBACK; return; }

-- 4. 执行扣款
UPDATE wallet SET balance = balance - 100 WHERE user_id = 1;

-- 5. 插入交易流水
INSERT INTO wallet_log(user_id, amount, type) VALUES (1, -100, 'DEDUCT');

-- 6. 提交事务（释放排他锁）
COMMIT;
```

**锁的兼容性**：`FOR UPDATE` 不会阻塞普通的 `SELECT`（快照读），但会阻塞以下操作：

- `SELECT ... FOR UPDATE` — 尝试加 X 锁
- `SELECT ... LOCK IN SHARE MODE` — 尝试加 S 锁
- `UPDATE` / `DELETE` — 隐式加 X 锁

**注意事项**：悲观锁的并发性能较差，且容易引发死锁（例如两个账户互相转账时加锁顺序不一致）。对于简单扣款场景，推荐优先使用 WHERE 条件原子更新或 CAS 乐观锁。

!!! note "SKIP LOCKED"

    `FOR UPDATE SKIP LOCKED` 会自动跳过已被其他事务锁定的行，非常适合任务队列等"抢占式消费"场景：

    ```sql
    SELECT * FROM task_queue
    WHERE status = 'PENDING'
    LIMIT 1
    FOR UPDATE SKIP LOCKED;
    ```

### 2. WHERE 条件原子更新（推荐）

将余额校验下沉到 `WHERE` 条件中，利用 MySQL 在执行 `UPDATE` 时自动加行级排他锁的特性，保证原子性。

```sql
BEGIN;

-- 仅当余额充足时才执行扣减，MySQL 自动加行锁保证原子性
UPDATE wallet
SET balance = balance - 100
WHERE user_id = 1 AND balance >= 100;

-- 通过 affected_rows 判断结果：
--   affected_rows == 1 → 扣款成功，继续插入日志并 COMMIT
--   affected_rows == 0 → 余额不足，ROLLBACK 并返回错误

INSERT INTO wallet_log(user_id, amount, type) VALUES (1, -100, 'DEDUCT');

COMMIT;
```

**优势**：减少了一次 SELECT 的网络往返（RTT），锁仅在 UPDATE 执行瞬间持有，死锁风险极低。

**局限**：仅适用于逻辑简单的单表扣减。如果业务涉及跨表查询（风控、优惠券、会员折扣等复杂校验），仍需使用 `FOR UPDATE` 先读后写。

### 3. 乐观锁：CAS 版本号

不加锁，而是在更新时校验版本号（Compare-And-Swap），若版本不匹配则说明数据已被其他事务修改，需要重试。

```sql
-- 1. 普通 SELECT，不加锁
SELECT balance, version FROM wallet WHERE user_id = 1;

-- 2. 更新时校验 version，防止覆盖
UPDATE wallet
SET balance = balance - 100, version = version + 1
WHERE user_id = 1 AND version = <old_version> AND balance >= 100;

-- affected_rows == 0 时需在应用层重试
```

### 方案对比

| 方案 | 优点 | 缺点 | 适用场景 |
|---|---|---|---|
| `SELECT ... FOR UPDATE` | 强一致性；支持复杂业务逻辑（先读后写） | 并发性能差；持锁时间长；易死锁 | 跨表校验、复杂业务流程 |
| WHERE 条件原子更新 | 实现简单；锁持有时间极短；死锁风险低 | 无法在扣减前执行复杂校验逻辑 | 单表余额扣减等简单场景（推荐） |
| CAS 版本号 | 读操作不加锁；并发性能好 | 高竞争时重试频繁；需应用层实现重试机制 | 读多写少、冲突概率低的场景 |