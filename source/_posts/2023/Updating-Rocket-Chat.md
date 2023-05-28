---
title: 更新 Rocket.Chat
date: 2023-05-28 19:46:31
updated: 2023-05-28 19:46:31
tags: [Docker, IM, MongoDB, Docker Compose]
categories: Docker
---

本文将对使用Docker Compose部署的Rocket.Chat[项目](https://github.com/filefi/RocketChatDeployment)进行升级。

根据官网文档，官方不建议跨大版本升级。理想状态下，甚至不应该跨越2个小版本进行升级。

{% note default [官方文档](https://docs.rocket.chat/deploy/updating-rocket.chat)原文 %}
- For a successful update, you must not skip any major version. That is, say you want to move from version 1.x.x to say 4.x.x, you need to traverse chronologically 1.x.x -> 2.x.x -> 3.x.x -> 4.x.x. Ideally, it's even better to make more granular steps, and not skip more than two minor versions at a time.

- Upgrading to v5 requires you to be on at least 4.x.

{% endnote %}

<!-- more -->

# 更新使用Docker部署的 Rocket.Chat

## 更新到最新版Rocket.Chat
拉取最新版的Docker镜像：

```
docker pull registry.rocket.chat/rocketchat/rocket.chat:latest
```

停止并删除当前运行中的容器，然后重新运行新容器：

```
docker compose stop rocketchat
docker compose rm rocketchat
docker compose up -d rocketchat
```

## 更新到指定版本Rocket.Chat

拉取指定版本的Docker镜像：

```
docker pull registry.rocket.chat/rocketchat/rocket.chat:latest
```

要更新到指定版本的Rocket.Chat，需要修改部署目录中的`.env`配置文件，或者`compose.yml`文件：

- 修改部署目录中的`.env`配置文件，将`RELEASE`变量的值修改为预期的Rocket.Chat版本号：

```
RELEASE=<desired version>
```

- 如果未使用`.env`配置文件，则可以直接修改`compose.yml`文件中`rocketchat`服务的`image`值所对应的Docker镜像tag改为预期的特定版本号：

```
services:
  rocketchat:
    image:registry.rocket.chat/rocketchat/rocket.chat:<desired version>
```

停止并删除当前运行中的容器，然后重新运行新容器：
```
docker compose stop rocketchat
docker compose rm rocketchat
docker compose up -d rocketchat
```

## 更新到指定版本的MongoDB

更新MongoDB前应提前对数据库进行备份，以便更新失败后回退。

### 使用`mongodump`备份MongoDB

> - `mongodump` allows you to create backups from standalone, replica sets or sharded cluster deployments.
> - As from MongoDB server 4.4, you are required to install the mongodump utility separately. Read more at the MongoDB Database tools docs https://www.mongodb.com/docs/database-tools/mongodump


使用`mongodump`命令备份远程MongoDB实例，将命令的参数修改为`compose.yml`中的对应值：

```mongodump --uri="mongodb://<host URL/IP>:<Port>" [additional options]```

### 使用`mongorestore`还原MongoDB

> - Make sure you drop first any existing rocketchat schema in your database with same name as the one you are restoring.
> - `mongorestore` allows you to load data from either a binary database dump created by mongodump or the standard input into MongoDB instance.
> - As from MongoDB server 4.4, you are required to install the mongorestore utility separately. Read more at the MongoDB Database tools docs https://www.mongodb.com/docs/database-tools/mongorestore/


根据部署目录中的`.env`配置文件和`compose.yml`设置相应的命令行参数：

```
mongorestore --uri="mongodb://<host URL/IP>:<Port>" /dump
```

### 更新MongoDB

MongoDB不允许跨大版本更新，必须按大版本逐个更新。假设当前RocketChat所使用的MongoDB版本为4.4，现在要将MongoDB更新到6.0，则需要先将4.4更新到5.0，然后再从5.0更新到6.0。

{% note info 前提条件%}
The 4.4 replica set must have `featureCompatibilityVersion` set to `"4.4"`. 

To ensure that all members of the replica set have `featureCompatibilityVersion` set to `"4.4"`, connect to each replica set member and check the `featureCompatibilityVersion`:

```js
db.adminCommand( { getParameter: 1, featureCompatibilityVersion: 1 } )
```
All members should return a result that includes `"featureCompatibilityVersion" : { "version" : "4.4" }`.

To set or update `featureCompatibilityVersion`, run the following command on the primary. A majority of the data-bearing members must be available:

```js
db.adminCommand( { setFeatureCompatibilityVersion: "4.4" } )
```

The 5.0 replica set must have `featureCompatibilityVersion` set to `"5.0"`.

To ensure that all members of the replica set have `featureCompatibilityVersion` set to `"5.0"`, connect to each replica set member and check the `featureCompatibilityVersion`:

```js
db.adminCommand( { getParameter: 1, featureCompatibilityVersion: 1 } )
```

All members should return a result that includes `"featureCompatibilityVersion" : { "version" : "5.0" }`.

To set or update featureCompatibilityVersion, run the following command on the primary. A majority of the data-bearing members must be available:

```js
db.adminCommand( { setFeatureCompatibilityVersion: "5.0" } )
```

{% endnote %}



以下操作将MongoDB 4.4 升级到 5.0，再从 5.0 更新到 6.0。具体步骤：

1. 拉取或加载指定版本的MongoDB镜像。要想将MongoDB 4.4 更新到 6.0，必须先更新到5.0，再从 5.0 更新到 6.0。所以我们需要首先拉取或加载 MongoDB 5.0 和 6.0 的Docker 镜像：

```
docker pull bitnami/mongodb:5.0
docker pull bitnami/mongodb:6.0
```


2. 进入容器`mongodb` shell：

```
docker exec -it mongodb bash
```

3. 进入MongoDB shell：
```
mongo
```

4. 检查当前MongoDB的`featureCompatibilityVersion`值。

```js
db.adminCommand( { getParameter: 1, featureCompatibilityVersion: 1 } )
```

5. 如果输出结果不为`"featureCompatibilityVersion" : { "version" : "4.4" }`，则手动设置其值为`4.4`：

```js
db.adminCommand( { setFeatureCompatibilityVersion: "4.4" } )
```

6. 修改部署目录中的`.env`配置文件的MongoDB版本变量（本文所使用的`compose.yml`部署文件中对应MongoDB版本的变量为`MONGODB_VERSION`）:

```
MONGODB_VERSION=5.0
```

7. 停止并删除当前运行`mongodb`容器，然后重新运行新容器：

```
docker compose stop mongodb
docker compose rm mongodb
docker compose up -d mongodb
```

8. 检验是否更新成功；
9. 继续。检查当前MongoDB的`featureCompatibilityVersion`值。

```js
db.adminCommand( { getParameter: 1, featureCompatibilityVersion: 1 } )
```

10. 如果输出结果不为`"featureCompatibilityVersion" : { "version" : "5.0" }`，则手动设置其值为`5.0`：

```js
db.adminCommand( { setFeatureCompatibilityVersion: "5.0" } )
```

11. 修改部署目录中的`.env`配置文件的MongoDB版本变量（本文所使用的`compose.yml`部署文件中对应MongoDB版本的变量为`MONGODB_VERSION`）:

```
MONGODB_VERSION=6.0
```

12. 停止并删除当前运行`mongodb`容器，然后重新运行新容器：

```
docker compose stop mongodb
docker compose rm mongodb
docker compose up -d mongodb
```

13. 检查是否更新成功。

