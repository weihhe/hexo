title: 数据库基础(MySQL为例)
author: weihehe
tags:
  - MySQL
  - ''
  - SQL
categories:
  - 数据库
date: 2024-08-25 15:49:00
---
有关概念，基础配置和登录
<!--more-->
## 概念
1. **MySQL (mysql)**:
  
  - `mysql` 是 MySQL 的**命令行客户端工具**。它允许用户通过命令行接口与 MySQL 数据库进行交互。你可以使用 `mysql` 命令来连接到 MySQL 服务器，执行 SQL 语句，管理数据库和用户，查询数据，等等。
  -  它主要用于直接向 MySQL 数据库发送查询或执行命令，并返回结果。例如，用户可以用 `mysql` 登录到 MySQL 数据库，然后运行 SQL 查询，创建或删除数据库，管理用户权限等。
  
2. **MySQL Server (mysqld)**:
  
  - `mysqld` 是**MySQL 数据库服务器。**`带d——daemon表明其是一个守护进程`**。它是 MySQL 数据库系统的核心服务，负责处理客户端发送的 SQL 请求并返回相应的结果。`mysqld` 会一直在后台运行，监听网络端口，等待客户端连接请求。
  - 这个进程管理着数据库实例，处理数据的存储、检索、更新等操作。它还负责执行 MySQL 的各种后台任务，如数据备份、恢复、索引管理和事务处理。`mysqld` 进程启动后，MySQL 服务器就会开始运行，准备好接受来自客户端（例如 `mysql` 客户端工具）的连接。

### mysql本质：基于C(mysql)S(mysqld)模式的一种网络服务。

### 存储引擎

- 存储引擎是：数据库完成管理系统存储数据、建立索引和更新、查询数据等功能的实现方法。
	- MySQL支持多种存储引擎。（`show engines`）
    

### 数据库

- 一般是指一个有组织的数据集合，通常用于存储、管理和检索大量的结构化数据。
	- 产生原因：
		- 文件的安全性问题。

		- 文件不利于数据查询和管理。

		- 文件不利于存储海量数据。

		- 文件在程序中控制不方便。
 
 ### 服务器，数据库，表关系

- 数据库服务器，只是在机器上安装了一个数据库管理系统程序。
  
- 这个管理程序可以管理多个数据库，一般开发人员会针对每一个应用创建一个数据库。
  
- 为保存应用中实体的数据，一般会在数据库中创建多个表，以保存程序中实体的数据。
![关系](/images/数据库-关系.png)

## 运行使用

1. [root@ubuntu]# vim /etc/my.cnf # 打开mysql配置⽂件

2. 在[mysqld]最后⼀栏配置加⼊: skip-grant-tables 选项并保存退出。
	- `systemctl restart mysqld`,跳过权限验证。这样就可以进行无密码登录`root`,并进行创建新用户等设置。
    
3. 登录 MySQL
	- 首先，以 `root` 或其他具有足够权限的用户登录 MySQL：
```bash
mysql -u root
```
4. 创建新用户
在 MySQL 命令行中执行以下命令创建一个新用户：
	- `username`: 新用户的用户名。
	- `hostname`: 允许该用户从哪个主机连接。如果想允许从任何主机连接，可以使用 `'%'`。
	- `password`: 新用户的密码。
    ```sql
CREATE USER 'username'@'hostname' IDENTIFIED BY 'password';
```

5. 为新用户分配权限

	- 为新用户分配权限，使用 `GRANT` 命令。以下命令为用户分配所有数据库的所有权限：
    ```sql
GRANT ALL PRIVILEGES ON *.* TO 'newuser'@'%';
```

6. 刷新权限

	- 为了使更改生效，执行以下命令刷新权限：`	- FLUSH PRIVILEGES;`

7. 继续配置
- 例如：
```sql
[mysqld]
 port=6666
 character-set-server=utf8
 default-storage-engine=innodb
```

## 数据库语言分类：

1. **DDL（Data Definition Language） - 数据定义语言**：
   - 用于定义和管理数据库中的数据结构，如表、视图、索引等。DDL 语句会自动提交，也就是说，一旦执行，无法回滚。
   - **代表指令**：
     - `CREATE`：用于创建数据库对象，如表、视图、索引等。例如，`CREATE TABLE` 用于创建新表。
     - `DROP`：用于删除数据库对象，如表、视图、索引等。例如，`DROP TABLE` 用于删除表。
     - `ALTER`：用于修改现有数据库对象的结构。例如，`ALTER TABLE` 用于修改表的结构。

2. **DML（Data Manipulation Language） - 数据操纵语言**：
   - 用于操作和处理数据库中的数据，包括插入、删除、更新数据等。DML 语句通常可以在事务中执行，并且可以回滚。
   - **代表指令**：
     - `INSERT`：用于向表中插入新数据。
     - `DELETE`：用于删除表中的数据。删除数据后可以通过事务回滚来恢复。
     - `UPDATE`：用于更新表中的现有数据。

3. **DQL（Data Query Language） - 数据查询语言**：
   - **功能**：DQL 实际上是 DML 的一个子集，专门用于查询数据。它允许用户从数据库中检索数据。
   - **代表指令**：
     - `SELECT`：用于查询数据，指定查询条件并从一个或多个表中检索数据。

4. **DCL（Data Control Language） - 数据控制语言**：
   - 用于控制访问权限和事务管理，确保数据库的安全性和完整性。DCL 语句通常在事务中执行，并且可以提交或回滚。
   - **代表指令**：
     - `GRANT`：用于授予用户特定的数据库权限，例如 `GRANT SELECT ON table_name TO user_name;`。
     - `REVOKE`：用于撤销授予用户的权限，例如 `REVOKE SELECT ON table_name FROM user_name;`。
     - `COMMIT`：用于提交当前事务，将所有更改永久保存到数据库中。
     - `ROLLBACK`：用于回滚当前事务，撤销所有未提交的更改。
     
### 常见操作

| 操作类型     | SQL 语句                                          | 功能描述                                  |
|--------------|---------------------------------------------------|-------------------------------------------|
| **创建数据库** | `CREATE DATABASE dbname;`                        | 创建一个新的数据库                        |
| **选择数据库** | `USE dbname;`                                     | 选择一个数据库以供后续操作                |
| **删除数据库** | `DROP DATABASE dbname;`                          | 删除一个数据库                            |
| **创建表**   | `CREATE TABLE tablename (columns);`                | 在数据库中创建一个新表                    |
| **删除表**   | `DROP TABLE tablename;`                            | 删除一个表                                |
| **插入数据** | `INSERT INTO tablename (columns) VALUES (values);` | 向表中插入一行或多行数据                  |
| **查询数据** | `SELECT columns FROM tablename WHERE condition;`   | 从表中查询数据                            |
| **更新数据** | `UPDATE tablename SET column=value WHERE condition;` | 更新表中符合条件的数据                    |
| **删除数据** | `DELETE FROM tablename WHERE condition;`           | 删除表中符合条件的数据                    |
| **创建用户** | `CREATE USER 'username'@'host' IDENTIFIED BY 'password';` | 创建一个新用户并指定密码                  |
| **授权权限** | `GRANT privileges ON dbname.* TO 'username'@'host';` | 授予用户对数据库的权限                    |
| **撤销权限** | `REVOKE privileges ON dbname.* FROM 'username'@'host';` | 撤销用户对数据库的权限                    |
| **开始事务** | `START TRANSACTION;`                               | 开始一个事务                              |
| **提交事务** | `COMMIT;`                                          | 提交当前事务中的所有操作                  |
| **回滚事务** | `ROLLBACK;`                                        | 回滚事务，将所有操作撤销                  |
| **备份数据库** | `mysqldump -u user -p dbname > backup.sql`         | 导出数据库到文件进行备份                  |
| **恢复数据库** | `mysql -u user -p dbname < backup.sql`             | 从备份文件中恢复数据库                    |
- show processlist 查看当前数据库的连接情况。


## 本质

- 建立数据库，本质上是创建一个目录。
![表在磁盘中的存储位置](/images/数据库-文件.png)

- 在数据库内建立表，本质就是在其数据库目录下，创建对应的文件。

并且以上操作是由`mysqld`来完成的,**并不由使用者直接修改，而是由数据库服务来操作**。

## 查看运行状态的一些命令
- netstat -nltp
- ps -axj | grep mysql