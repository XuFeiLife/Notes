# 1. 拉取镜像
```bash
docker pull postgres:17

```

# 2. 启动容器
```bash
docker run -d \
  --name postgres17 \
  -e POSTGRES_USER=postgres \
  -e POSTGRES_PASSWORD=123456 \
  -e POSTGRES_DB=testdb \
  -p 5432:5432 \
  -v pgdata:/var/lib/postgresql/data \
  postgres:17

```

关键参数 👇

- `-e POSTGRES_USER`：用户名
- `-e POSTGRES_PASSWORD`：密码
- `-e POSTGRES_DB`：默认数据库
- `-v pgdata:/var/lib/postgresql/data`：**数据持久化（非常重要）**

# 3. 验证启动是否成功
```bash
docker ps
docker logs postgres17
```

# 4. 进入容器测试
```bash
docker exec -it postgres17 bash
psql -U postgres
select current_database();
```