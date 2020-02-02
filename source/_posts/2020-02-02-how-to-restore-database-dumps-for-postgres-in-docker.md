---
title: 怎样将数据dump到docker中的postgres里
date: 2020-02-02 14:57:23
tags: [Docker, Postgres]
---

我在日常开发中使用的数据库是PostgreSQL，并且最近使用了Laradock的Docker环境。最近碰到了一个需要将数据库的dump文件倒入到Docker中的Postgres中的问题, 于是进行一下简单的记录。

其实这个问题的本质是将我们需要的dump文件copy到postgres的容器中，接着我们在容器内就可以使用`pg_restore`命令了。

<!--more-->

## 将dump文件拷贝到容器中

假设我们的dump文件叫`latest.dump` 那么我们可以使用下面的命令将这个文件拷贝到容器内的`/backups`文件夹下。

```
docker cp latest.dump laradock_postgres_1:/backups
```

>Laradock的默认容器名字是 laradock_postgres_1，你需要根据实际情况更换成你自己的容器名

## 执行pg_restore 命令

接下来我们就可以执行`pg_restore`命令了。

```
docker-compose exec postgres pg_restore --no-owner --no-acl --clean --if-exists -U default -d yourdb /backups
```

需要注意的是 docker-compose 需要在 laraock 的文件夹下执行，并且要确保对应的数据库存在这样我们就将数据库的dump文件导入到Docker中的Postgres中去了。
