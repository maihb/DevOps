创建 1 个自增表

```sql
-- 创建一个示例表
CREATE TABLE lt (
    id INT AUTO_INCREMENT PRIMARY KEY ,
    value INT
);

-- 插入一些示例数据
INSERT INTO lt (id, value) VALUES (3, 333);
INSERT INTO lt (value) VALUES (100), (200);

-- 开始事务 或者直接 begin;
START TRANSACTION; 


-- 在事务中查询数据
SELECT * FROM lt;

-- 在另一个会话中更新数据
-- 在另一个客户端中执行以下命令：
-- UPDATE lt SET value = 300 WHERE id = 2;

UPDATE lt SET value = 300 WHERE value = 3234;

-- 再次在本事务中查询数据
SELECT * FROM lt;

-- 提交事务
COMMIT;

```


一个死锁案例

```sql
CREATE TABLE `t_order` (
  `id` int NOT NULL AUTO_INCREMENT PRIMARY KEY,
  `order_no` int DEFAULT NULL,
  `create_date` datetime DEFAULT NULL,
  KEY `index_order` (`order_no`) USING BTREE
) ENGINE=InnoDB ;

--清空数据，方便从头来
TRUNCATE TABLE t_order;
--插入几条数据
INSERT INTO t_order (order_no, create_date) VALUES (1001,NOW()) ;
INSERT INTO t_order (order_no, create_date) VALUES 
(1002,NOW()), 
(1003,NOW()), 
(1004,NOW()), 
(1005,NOW()), 
(1006,NOW())
;
select * from t_order;
```

|                            事务 A                            |                            事务 B                            |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
|                            begin;                            |                            begin;                            |
|    select id from t_order where order_no=1007 for update;    |                                                              |
|                                                              |    select id from t_order where order_no=1008 for update;    |
| INSERT INTO t_order (order_no, create_date) VALUES (1007,NOW()) ; |                                                              |
|                                                              | INSERT INTO t_order (order_no, create_date) VALUES (1008,NOW()) ; |
|                        insert 成功。                         | 先报错：ERROR 1213 (40001): Deadlock found when trying to get lock; try restarting transaction |

