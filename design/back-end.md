# TimeForCoin 后端

## 技术选型

后端：Golang

路由框架：[iris](https://github.com/kataras/iris)

数据库：MongoDB

数据库驱动：[mongo-go-driver](https://github.com/mongodb/mongo-go-driver/)

缓存：Redis

缓存驱动：[go-redis](https://github.com/go-redis/redis)

日志库：[zerolog](https://github.com/rs/zerolog)

API 规范：RES
API

API 文档：Swagger

## 架构设计

图中箭头为依赖关系

![1553527547945](back-end/1553527547945.png)

其中`Controllers`, `Service`, `Models`再按照不同的业务划分为不同的模块

结合前端，整体的服务器架构大概如下：

![ch6-08-frontend-backend](back-end/ch6-08-frontend-backend.png)
