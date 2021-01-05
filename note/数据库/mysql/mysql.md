
### 语句
----------------
####  查询表
```sql
select table_name,TABLE_COMMENT from information_schema.tables where table_schema = 'project'
```
#### 查询表字段
```sql
SELECT
    TABLE_NAME,
    COLUMN_NAME,
    ORDINAL_POSITION,
    COLUMN_DEFAULT,
    DATA_TYPE AS 'data_type',
    COLUMN_KEY 'KEY',
    COLUMN_COMMENT AS 'des'
FROM
    information_schema.`COLUMNS`
WHERE
    TABLE_SCHEMA = 'project'
ORDER BY
    TABLE_NAME,
    ORDINAL_POSITION;
```

--------------



### 异常
----------
#### navicat连接mysql8.0提示Authentication plugin 'caching_sha2_password' cannot be loaded
```
查看默认密码的加密方式
show variables like 'default_authentication_plugin';

查看登录用户的加密方式
select host,user,plugin from mysql.user;

1 连接mysql 
mysql -uroot -p

2 修改加密规则：
ALTER USER 'root'@'localhost' IDENTIFIED BY 'password' PASSWORD EXPIRE NEVER;

3 更新用户密码：
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';

4 刷新权限： 
FLUSH PRIVILEGES;

5 重置密码：
ALTER USER 'root'@'localhost' IDENTIFIED BY '新密码';

执行完后查看下登录用户的加密方式,root账号可能需要将'root@%'重复执行2-5步骤
``` 


----------