---
title: MySQL常见面试题
createTime: 2025/08/21 10:31:57
permalink: /java-guide/396o7sc3/
---
## 介绍数据库中的悲观锁和乐观锁
### 悲观锁
> 悲观锁正如其名称，比较悲观。总会认为：每当修改数据时，会有其他线程也会同时修改该数据。所以针对这种情况悲观锁的做法是：读取数据之后就加锁(eg: select...for update)，这样别的线程读取该数据的时候就需要等待当前线程释放锁，获得到锁的线程才能获得该数据的读写权限。从而保证了并发修改数据错误的问题。但是由于阻塞原因，所以导致吞吐量不高。悲观锁更适用于多写少读的情况。

**场景:**  同学A和同学B都要给你转500块钱(开心坏了吧，这样最终你能得到1000块钱)。

**使用悲观锁的流程:**
1. 同学A获取到你的账户余额`balance = 0`并对该条记录加锁。
2. 同学B获取你的账户余额。由于同学A已经对这条记录加锁了，所以同学B需要等同学A转帐完成(释放锁)才能获得余额。
3. 同学A转账完成并释放锁，此时你的账户余额`balance=balance + 500 = 500`
4. 同学B获取到你的账户余额`balance = 500`，并对该条记录加锁(如果你人缘好，此时同学C给你转账也是需要等待同学B转账完成才可以转账哦)
5. 同学B转账完成并释放锁(如果有同学C想给你转账，此时同学C就可以获得锁并转账了)。此时你的账户余额为`balance = balance + 500 = 1000`
6. 最终你开开心心的得到了1000块钱。

使用悲观锁查询的SQL示例如下，同学A获取到账号余额之后，会对该行记录进行加锁，期间其他用户阻塞等待访问该记录。所以只有等同学A的操作事务完成之后，同学B才能继续操作。
```SQL
BEGIN;
SELECT * FROM `account` WHERE `account_id` = #{你的账户Id} FOR UPDATE;
UPDATE `account` set `balance` = `balance` + 500 WHERE `account_id` = #{你的账户Id};
COMMIT;
```

**假设转账过程没有锁，我们看看会发生什么:**

1. 同学A获取到你的账户余额balance_a = 0(没有加锁，此时同学B也可以获取到账户余额)
2. 同学B获取到你的账户余额balance_b = 0
3. 同学A转账完成，此时你的账户余额为balance = balance_a + 500 = 500
4. 同学B转账完成，此时你的账户余额为balance = balance_b + 500 = 500
5. 最终同学A和同学B都转了500，但是你最终只获得了500。这一定是不能接受的吧。

丢失的500块去哪里了呢？从第2步可以看到同学B获取到的账户余额是0，而不是同学A转帐之后的余额500。所以问题出在这里，这是高并发场景的常见问题。所以加锁是非常必须的。但是加了悲观锁，同学都要排队给我转账，对于没有耐心的同学就直接不转帐了，我岂不是错失了发财的好机会。那有什么好办法呢？答案就是下面的乐观锁

### 乐观锁
> 乐观锁顾名思义比较乐观，他只有在更新数据的时候才会检查这条数据是否被其他线程更新了(这点与悲观锁一样，悲观锁是在读取数据的时候就加锁了)。如果更新数据时，发现这条数据被其他线程更新了，则此次更新失败。如果数据未被其他线程更新，则更新成功。由于乐观锁没有了锁等待，提高了吞吐量，所以乐观锁适合多读少写的场景。
> 常见的乐观锁实现方式是：版本号version和CAS(compare and swap)。此处只介绍版本号方式。

要采用版本号，首先需要在数据库表中新增一个字段`version`，表示此条记录的更新版本，记录每变动一次，版本号加1。依旧使用上面转账的例子说明：
1. 同学A获取到你的账户余额balance = 0和版本号version_a = 0
2. 同学B获取到你的账户余额balance = 0和版本号version_b = 0
3. 同学A转账完成`UPDATE account SET balance = #{balance}, version = version + 1 WHERE account_id=#{你的账户Id} AND version = 0`。(此时版本号为0，所以更新成功)
4. 同学B转账完成`UPDATE account SET balance = #{balance}, version = version + 1 WHERE account_id=#{你的账户Id} AND version = 0`。(此时版本号为1，所以更新失败，更新失败之后同学B再转一次即可)
5. 同学B获取到你的账户余额balance = 500和版本号version_b = 1
6. 同学B重新转帐之后，你还是美滋滋的获得了1000。

使用乐观锁查询的SQL示例如下：
```SQL
-- 同学A获取你的账户信息，假设此时version=0
SELECT `version`, `balance` FROM `account` WHERE `account_id` = #{你的账户Id}

-- 假设同学A的转账还未完成，同学B就发起了请求。因为此时同学A的更新未完成，所以version依然是0
SELECT `version`, `balance` FROM `account` WHERE `account_id` = #{你的账户Id}

-- 同学A转账更新操作
UPDATE `account ` SET `balance` = `balance` + 500, `version` = `version` + 1 WHERE `account_id`=#{你的账户Id}  AND `version` = 0

-- 同学B转账更新操作，此时由于同学A转账完成，DB的version变成1，所以此次更新失败
UPDATE `account ` SET `balance` = `balance` + 500, `version` = `version` + 1 WHERE `account_id`=#{你的账户Id}  AND `version` = 0

-- 同学B第二次获取你的账户信息，假设此时version=1
SELECT `version`, `balance` FROM `account` WHERE `account_id` = #{你的账户Id}

-- 同学B转账更新操作，更新成功
UPDATE `account ` SET `balance` = `balance` + 500, `version` = `version` + 1 WHERE `account_id`=#{你的账户Id}  AND `version` = 1

```

### 总结
1. 悲观锁：读取时加锁，更新完释放锁，再此过程中会造成其他线程阻塞，导致吞吐量低，适用于多写场景。
2. 乐观锁：不加锁，只有在更新时验证数据是否被其他线程更新，吞吐量较高，适用于多读场景

## 数据库事务和隔离级别

### 数据库事务四大特性(ACID)
#### 原子性(Atomicity)
原子性是指事务中的操作要么全部成功，要么失败回滚。

#### 一致性(Consistency)
一致性是指事务必须使数据库从一个一致性状态变换到另一个一致性状态，也就是说一个事务执行之前和执行之后都必须处于一致性状态。拿转账来说，假设用户A和用户B两者的钱加起来一共是5000，那么不管A和B之间如何转账，转几次账，事务结束后两个用户的钱相加起来应该还得是5000，这就是事务的一致性。

#### 隔离性(Isolation)
隔离性是指两个事务之间的操作互不影响。关于事务的隔离级别后面将会介绍。

#### 持久性(Durability)
持久性是指一个事务一旦被提交了，那么对数据库中的数据的改变就是永久性的，即便是在数据库系统遇到故障的情况下也不会丢失提交事务的操作。

### 隔离性会出现的问题
如果不采取数据库隔离会出现以下问题：

#### 脏读
事务A将data的值由4修改成5，但是并未提交；此时事务B读取data的值是5.但是事务A并没有提交进行了回滚，此时事务B读取的就是一个脏数据(因为此时数据库中data的数据是4不是5)。

#### 不可重复读
不可重复读是指在对于数据库中的某个数据，一个事务范围内多次查询却返回了不同的数据值，这是由于在查询间隔，被另一个事务修改并提交了。例如事务T1在读取某一数据，而事务T2立马修改了这个数据并且提交事务给数据库，事务T1再次读取该数据就得到了不同的结果，发生了不可重复读。不可重复读和脏读的区别是，脏读是某一事务读取了另一个事务未提交的脏数据，而不可重复读则是读取了前一事务提交的数据。在某些情况下，不可重复读并不是问题，比如我们多次查询某个数据当然以最后查询得到的结果为主。但在另一些情况下就有可能发生问题，例如对于同一个数据A和B依次查询就可能不同，A和B就可能打起来了。
**注：不可重复读强调的是多次查询，同样的select语句，但是2次读取的结果不一样。**

#### 幻读
幻读是事务非独立执行时发生的一种现象。例如事务T1对一个表中所有的行的某个数据项做了从“1”修改为“2”的操作，这时事务T2又对这个表中插入了一行数据项，而这个数据项的数值还是为“1”并且提交给数据库。而操作事务T1的用户如果再查看刚刚修改的数据，会发现还有一行没有修改，其实这行是从事务T2中添加的，就好像产生幻觉一样，这就是发生了幻读。幻读和不可重复读都是读取了另一条已经提交的事务（这点就脏读不同），所不同的是不可重复读查询的都是同一个数据项，而幻读针对的是一批数据整体（比如数据的个数）。
**注：幻读的重点在于新增或删除：同样条件的select，读取的记录不一样。**

### 数据库隔离级别
> ✅表示在该隔离级别下可能会存在对应的问题
> 
> ❌表示在该隔离级别下不会存在对应的问题

|  | 脏读 | 不可重复读 | 幻读 |
| :---: | :---: | :---: | :---: |
| 读未提交(read-uncommitted) | ✅ | ✅ | ✅ |
| 读提交(read-committed) | ❌ | ✅ | ✅ |
| 可重复度(repeatable-read) | ❌ | ❌ | ✅ |
| 串行化(Serializable) | ❌ | ❌ | ❌ |

- 串行化(Serializable)保证了完整的隔离性，是一致性最好，但是并发性最差的；
- 读未提交(read-uncommitted)一致性最差。
- 串行化、读未提交这两种在实际开发中基本不会用到。
- 可重复读(Repeatable-read)：是MySQL默认的隔离级别。
- 通过“select @@tx_isolation”查看采用的隔离级别，通过"set tx_isolation=${isolation_level}"设置相应的隔离级别。

## binlog、redo Log、undo log作用分别是什么？结构有什么区别？

## 分库分表的组件，如何生成分布式ID？

## 什么是MVCC， 可以解决什么问题

## MySQL索引失效的场景
> 平时是不是遇到过明明加了索引，查询还是慢得像蜗牛？很可能是你的索引‘失灵’了！今天我们就来盘点那些导致索引失效的经典场景，附上实战示例，让你彻底搞懂

在深入细节之前，我们可以通过一张图来建立一个整体的认知。下图概括了最常见的索引失效场景及其核心原因：
![](/images/sql_index_invalid.png)

接下来，我们来逐一剖析这些场景的具体表现。

**1. 违背最左前缀原则 (The Leftmost Prefix Rule)**

这是联合索引最常见的坑。
- **示例：** 我们有一个联合索引 `INDEX idx_name_age (name, age)`。
    ```sql
    -- 【有效✅】使用了索引的最左列`name`
    EXPLAIN SELECT * FROM users WHERE name = '小毛';

    -- 【有效✅】使用了索引的最左列`name`，并且包含`age`
    EXPLAIN SELECT * FROM users WHERE name = '小毛' AND age = 25;

    -- 【失效❌】没有从最左列`name`开始，跳过了它
    EXPLAIN SELECT * FROM users WHERE age = 25;

    -- 【部分有效】使用了最左列`name`，但因为是范围查询，`age`列无法用索引进一步筛选
    EXPLAIN SELECT * FROM users WHERE name = '小毛' AND age > 20;
    ```

**2. 在索引列上使用函数或计算**

一旦对索引列进行操作，MySQL就无法直接使用该索引的排序结构了。

- **示例：** 在`create_time`、 `age` 列上有索引。
    ```sql
    -- 【失效❌】对索引列使用了函数
    EXPLAIN SELECT * FROM users WHERE YEAR(create_time) = 2023;

    -- 【有效✅】改为范围查询，避免对列操作
    EXPLAIN SELECT * FROM users WHERE create_time BETWEEN '2023-01-01' AND '2023-12-31';

    -- 【失效❌】对索引列进行了计算
    EXPLAIN SELECT * FROM users WHERE age + 1 = 30;

    -- 【有效✅】将计算移到等号另一边
    EXPLAIN SELECT * FROM users WHERE age = 29;
    ```

**3. 隐式类型转换 (Implicit Type Conversion)**

如果字符串类型的索引列，你传入了一个数字，MySQL会进行隐式转换，相当于在列上使用了函数。

- **示例：** `user_id` 是 `VARCHAR` 类型，并且有索引。
    ```sql
    -- 【失效❌】数据库需要将每个user_id转换成数字，才能与123比较，相当于使用了CAST函数
    EXPLAIN SELECT * FROM users WHERE user_id = 123;

    -- 【有效✅】传入字符串，类型匹配
    EXPLAIN SELECT * FROM users WHERE user_id = '123';
    ```

**4. 使用不等于 (!= 或 <>)**

不等于条件无法直接利用索引的有序性来快速定位数据，通常会导致全表扫描。

- **示例：** 在 `age` 列上有索引。
    ```sql
    -- 【失效❌】不等于查询，需要检查所有行
    EXPLAIN SELECT * FROM users WHERE age != 25;
    EXPLAIN SELECT * FROM users WHERE age <> 25;
    ```

**5. 使用 `LIKE` 以通配符开头**

索引是按照字段值的前缀排序的。以 `%` 开头，意味着没有确定的前缀，索引无法定位。

- **示例：** 在 `name` 列上有索引。
    ```sql
    -- 【失效❌】以通配符开头，索引不知道从哪找起
    EXPLAIN SELECT * FROM users WHERE name LIKE '%毛';

    -- 【有效✅】确定的前缀，索引可以快速定位
    EXPLAIN SELECT * FROM users WHERE name LIKE '小%';
    ```

**6. 使用 `OR` 连接非索引列**

如果 `OR` 前后的条件并非都基于索引，MySQL通常会选择全表扫描。

- **示例：** 只在 `name` 列上有索引，`age` 列没有。
    ```sql
    -- 【失效❌】`age`列无索引，导致整个查询无法有效使用索引
    EXPLAIN SELECT * FROM users WHERE name = '小毛' OR age = 25;

    -- 【解决方案】可以考虑将`age`也加上索引，或使用UNION改写。
    ```

**7. 数据库评估使用全表扫描更快时**

如果表非常小，或者查询需要返回表中很大比例的数据（例如超过30%），MySQL优化器可能认为直接读取整个表（全表扫描）比“索引查找+回表”的代价更小，从而放弃使用索引。

- **示例：** 在 `status` 列（只有0，1两种状态）上有索引。
    ```sql
    -- 【可能失效】当status=1的数据占全表40%时，优化器可能选择全表扫描
    EXPLAIN SELECT * FROM orders WHERE status = 1;
    ```