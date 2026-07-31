# MySQL

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