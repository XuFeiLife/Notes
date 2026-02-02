# 1. psql 内置命令（\ 开头）

## 1.1 数据库相关
```bash
\l            -- 查看所有数据库
\c db_name    -- 切换数据库
\conninfo     -- 当前连接信息

```

## 1.2 用户 角色
```bash
\du           -- 查看所有用户（角色）
\du+          -- 查看用户详情

```

## 1.3 表 schema
```bash
\dt           -- 当前 schema 下的表
\dt *.*       -- 所有 schema 下的表
\dn           -- 查看 schema

```
## 1.4 表结构
```
\d table_name
\d+ table_name   -- 含存储、索引等信息

```
## 1.5 索引 视图
```
\di           -- 索引
\dv           -- 视图
\dm           -- 物化视图

```

# 2. 数据库管理 SQL（非常常用）

## 2.1 创建删除数据库
```
CREATE DATABASE testdb;
DROP DATABASE testdb;

```

## 2.2 设置编码（建库时）
```
CREATE DATABASE testdb
WITH ENCODING='UTF8';

```

# 3. 用户 权限

## 3.1 创建用户
```
CREATE USER app_user WITH PASSWORD '123456';

```

## 3.2 授权数据库
```
GRANT ALL PRIVILEGES ON DATABASE testdb TO app_user;

```

## 3.3 切换用户
```
psql -U app_user -d testdb

```

