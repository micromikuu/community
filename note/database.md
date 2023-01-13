# 安装mysql
1. 先`mysqld --initialize`，再`mysqld -install`
2. Failed to find valid data directory
- mysql的错误提示是真的不靠谱，目录不建好是*Can't create test file*、*Failed to set datadir*。启动参数打错了都是一样的*Failed to find valid data directory*错误，包括`mysqld --initialize-insecure`和`mysqld -install`

# 配置mysql
1. 字符集配置
- utf8是utf8mb3的别名，是不完整的utf-8实现，字符最大长度为3字节，默认推荐的配置为utf8mb4
- 客户端字符集配置：`default-character-set=utf8mb4`
- 服务端字符集配置：`character-set-server=utf8mb4`

2. 安全软件问题
- 安装 MySQL 服务器后，建议在用于存储 MySQL 表数据的主目录 （datadir） 上禁用病毒扫描。 此外，默认情况下，MySQL在标准Windows临时目录中创建临时文件。要防止同时扫描临时文件，请为 MySQL 临时文件配置单独的临时目录，并将此目录添加到病毒扫描排除列表中。为此，请将 tmpdir 参数的配置选项添加到配置文件中。

# 查看参数
1. 查看数据库
- `show databases;`
2. 查看当前数据库
- `select database();`
3. 查看表
- `show tables;`
4. 查看参数
- `show variables like '%char%';`
5. 查看当前连接
- 需要用root用户才能看到，所有用户的连接，否则只能看到登录用户自己使用的连接:`show processlist;`

# 用户配置
1. 修改密码
- mysql8.0:`alter user 'root'@'localhost' identified by '464646';`
- mysql5.7:`update user set authentication_string=password('464646') where user='root;`，如果报错需要执行`flush privileges;`
- localhost可以为%，表示用户可以来自任意的访问域
- 由于mysql8.0使用了新的加密方式，连接mysql8时使用旧的加密方式就可能出现密码错误的情况，所以需要把加密方式还原成旧的:`alter user 'root'@'localhost' identified with mysql_native_password by '464646';`
2. 创建用户
- mysql8.0/5.7:`create user 'username'@'localhost' identified by 'password';`
3. 查看用户
- ```
  use mysql;
  select user,host,authentication_string from user;
  ```
4. 删除用户
- `drop user 'username'@'localhost';`
# 用户权限
1. 授权用户
- `grant priv(all privileges:select,delete....)  on database.table to 'username'@'localhost';`
- 刷新权限:`flush privileges;`
- database、table可以是通配符*
2. 撤销权限
- `revoke priv(all privileges:select,delete....) on database.table from 'username'@'localhost';`
3. 显示用户权限
- `show grants for 'username';`
4. 创建角色
- `create role 'rolename';`
5. 授权角色
- `grant priv(all privileges:select,delete....)  on database.table to 'rolename';`
6. 授权用户角色
- `grant 'rolename' to 'username';`


# 表操作
1. 创建表
- ```
    create table tablename (
    'id' int not null auto_increment primary key,
    'name' varchar(20) not null default '',
    'age' int nut null default 0,
    'create_time' datetime not null default current_timestamp,
    ...
    )
    ```
- 字段名全小写
- 字段名后必须是数据类型
2. 插入数据
- `insert into tablename (field1,field2,...) values(value1,value2,...),(value1,value2,...);`
3. 删除数据
- `delete from tablename where condition;`
4. 更新数据
- `update tablename set field1=value1,field2=value2 where condition;`

# 数据类型
1. datetime和timestamp
- TIMESTAMP 在存储时，会将时间戳从当前时区(time_zone参数值)转换成 UTC 进行存储，在读取时，会将时间戳从 UTC 时区转换为当前时区
  但是 DATE 和 DATETIME 在存取时并不会做上面的转换，而是将字面值直接存取
2. 报错`Row size too large`
- 有一个字段设置为`varchar(16111)`,我踏马字符集是`utf8mb4`，不是utf8，这个字段的字节是16111*4=64444，真的服了。


# 数据导入
1. 文件导入需要有`FILE`和`INSERT`权限
2. 基本语句格式:
  ```
  load data infile 'D:\\d\\file' into tablename
  fields terminated by ','
  enclosed terminated by '"'
  lines terminated by '\r\n'
  ignore 1 rows;
  ```
- Windows系统的文件路径记得加转义字符
- 文件字段由`fields terminated by`
3. 报错`The MySQL server is running with the --secure-file-priv option so it cannot execute this statement.`
- 因为在安装MySQL的时候限制了导入与导出的目录权限。只允许在规定的目录下才能导入。可以通过`show variables like 'secure_file_path'`查看`secure-file-priv`当前的值是什么。
- (1) NULL表示禁止 (2) 如果有目录，则只允许导入该目录的文件（不包括子目录） (3) 空表示不限制目录
- 可以修改配置文件，增加键项为`secure_file_priv=`，再重启mysql
2. 报错`No such file or directory`
- 导入目录必须为英文
3. 报错`Row 1 doesn't contain data for all colums`
- 添加行分割符`lines terminated by '\r\n'`，但是无效
- 关掉strict模式：`set sql_mode=''`，调整sql_mode, 把STRICT_TRANS_TABLES,NO_ENGINE_SUBSTITUTION