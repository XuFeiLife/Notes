[1. 介绍](#1.%20介绍)
[2. 官方文档](#2.%20官方文档)
[3. 核心功能](#3.%20核心功能)
[4. 安装使用](#4.%20安装使用)

# 1. 介绍

BFG Repo-Cleaner 是一个用 Scala 编写的、专门用于清理 Git 仓库历史的命令行工具。
它可以简单、快速地从 Git 提交历史中永久删除不需要的大文件（如 jar 包、视频、机密信息等）是 git filter-branch 命令的一个更高效、更用户友好的替代品。

# 2. 官方文档

[官方文档](https://rtyley.github.io/bfg-repo-cleaner)
# 3. 核心功能
1. 不小心提交了一个巨大的视频文件、编译产物（如 .jar, .class）或数据库备份，导致仓库体积暴增。
即使后续提交中删除了它，这个文件依然存在于历史记录中，每个人克隆时都需要下载。
2. 清除敏感信息：不小心将密码、API 密钥、私钥等敏感数据提交到了仓库。你需要彻底地从所有历史记录中抹去它们。
3. 清理特定文件：批量删除所有历史中的 .gitignore 文件、临时文件等。 

# 4. 安装使用
## 4.1 安装
从官网下载.jar文件，然后使用java命令执行

## 4.2 使用流程
 克隆一个裸仓库，生成一个my-repo.git 文件夹
```bash
git clone --mirror git@example.com:your-user/my-repo.git
```

 删除所有大于100M的文件
```bash
java -jar bfg-1.15.0.jar --strip-blobs-bigger-than 100M my-repo.git
```

删除特定文件(password.txt)
```bash
java -jar bfg-1.15.0.jar delete-files password.txt my-repo.git
```

 清理敏感信息，比如你需要清除你的密码信息，假设你的密码是abc123456,替换成security，定义replace.txt，内容如下:
```
abc123456==>security
```

```bash
java -jar bfg-1.15.0.jar --replace-text repl.txt  my-repo.git
```

 清理残留数据并强制推送
```
cd my-repo.git
git reflog expire --expire=now --all
git gc --aggressive --prune=now
git push origin --force
```

