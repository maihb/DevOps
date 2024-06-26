# 事务相关测试 sql

查看当前事务隔离级别：
SELECT @@global.tx_isolation;
查看当前用的是哪个库
SELECT DATABASE();
查看日志等级
show variables like 'log_error_verbosity'; --默认 3

开启 swap

```sh
dd if=/dev/zero of=/mnt/swap bs=1M count=4096
chmod 0600 /mnt/swap 
mkswap /mnt/swap 
swapon /mnt/swap
echo '/mnt/swap swap swap defaults 0 0' >>/etc/fstab
```

```sql
-- 创建表
CREATE TABLE `wode` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `username` varchar(50) COLLATE utf8mb4_bin NOT NULL COMMENT '用户名',
  `password` varchar(255) CHARACTER SET utf8mb4 DEFAULT NULL COMMENT '密码',
  `phone` varchar(20) CHARACTER SET utf8mb4 DEFAULT NULL COMMENT '手机号',
  `email` varchar(255) CHARACTER SET utf8mb4 DEFAULT NULL COMMENT '邮箱',
  `create_time` datetime DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  PRIMARY KEY (`id`),
  unique index pid (phone)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin;

-- 随便添加几个数据
INSERT INTO wode (username, password, phone, email, create_time) VALUES ('abc','sdf','12317231234' ,'mhb@qq.com',NOW()) ;
INSERT INTO wode (username, password, phone, email, create_time) VALUES ("abc","sdf","12317231233" ,"mhb@qq.com",NOW()) ;
select * from wode order by id DESC limit 3;
-- 关闭自动提交事务
SET autocommit=0; SHOW VARIABLES LIKE 'autocommit';
-- 插入数据
DELIMITER $$
CREATE PROCEDURE insertTestData(IN start INT, IN total INT)
BEGIN
    DECLARE i INT DEFAULT 0;
    DECLARE j INT DEFAULT 0;
    SET i = start;
    WHILE i < start + total DO
        INSERT INTO wode (username, password, phone, email, create_time) 
        VALUES 
        (CONCAT('user', i), 
         MD5(CONCAT('password', i)), 
         CONCAT('1234567890', i), 
         CONCAT('user', i, '@example.com'), 
         NOW());
        SET i = i + 1;
        SET j = j + 1;
        IF j = 1000 THEN COMMIT; SET j = 0; END IF;
    END WHILE;
    COMMIT;
END$$
DELIMITER ;

-- 调用存储过程插入数据 insertTestData(开始位置，条数) 100w
CALL insertTestData(1, 1000000);
-- +500w 数据， 共 600w
CALL insertTestData(1000001, 5000000);
-- +400w，共 1kw   ， 没开 swap 情况下，卡死了
CALL insertTestData(6000001, 4000000);       

-- 查看数据空间大小
select
TABLE_NAME,
concat(truncate(data_length/1024/1024,2),' MB') as data_size,
concat(truncate(index_length/1024/1024,2),' MB') as index_size
from information_schema.tables
where TABLE_SCHEMA = 'test' and table_name = 'wode'
order by data_length desc;

-- 删掉存储过程
DROP PROCEDURE insertTestData;

-- 清空表数据
TRUNCATE TABLE wode;
OPTIMIZE TABLE wode;

```