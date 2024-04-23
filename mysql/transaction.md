# 事务相关测试 sql

查看当前事务隔离级别：
SELECT @@global.tx_isolation;

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
SET autocommit=0;
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

-- 调用存储过程插入数据
CALL insertTestData(1, 1000000);
-- 删掉存储过程
DROP PROCEDURE insertTestData;

-- 清空表数据
TRUNCATE TABLE wode;
OPTIMIZE TABLE wode;


```