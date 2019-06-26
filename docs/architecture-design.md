# 架构设计

## 逻辑架构

![ch6-08-frontend-backend](architecture-design/ch6-08-frontend-backend.png)

本应用的逻辑架构由三层模型构成：

### 表示层

- 使用微信小程序作为应用的表示层，主要服务于任务参与者，除了系统的基本功能之外，主要提供消息和通知功能，便于用户及时收到通知和信息。此外还支持一般的跑腿信息任务发布。
- 使用Web作为应用的表示层，主要服务于任务发布者，除了系统的基本功能之外，主要提供问卷任务的编辑与统计功能。

### 业务逻辑层

使用Golang作为服务端语言，构建HTTP服务器通过API请求处理业务逻辑，对从表示层过来的请求进行校验与处理，然后对数据持久层进行操作(读取/写入数据)并处理，最后返回给表示层进行数据的展示。

### 数据持久层

- 使用 MongoDB 作为数据库，存储业务数据
- 使用 Redis 作为缓存，用于持久化会话以及缓存一些需要重复访问的信息。
- 使用 腾讯云的对象存储服务(COS) 保存图片/任务附件等文件

## 代码结构

## ECB 关系

- Entity：代表系统数据，如：用户、任务、问卷等
- Boundary：与用户的接口，如：UI界面、网关、代理等
- Controller：连接 Boundary 和 Entity 的媒介，编排来自 Boundary 的命令的执行

### Entity

- 在[API文档](http://xm.zhenly.cn//docs/swagger/?url=https://raw.githubusercontent.com/TimeForCoin/Dashboard/master/design/api.yaml#/)的底部定义了应用表示层可以获得的所有实体数据结构
- 在服务端的`Model`层定义了数据持久层所存储的所有的实体数据结构

### Boundary

- 通过微信小程序提供用户界面
- 通过 Web 页面提供用户界面
- 通过 Nginx 反向代理提供静态页面文件和API请求转发

### Controller

- 在服务端的`Controller`层中接受来自 Boundary 的命令，校验请求的合法性然后调用 `Service` 层中的指令进行业务逻辑的执行。
- 在服务端的`Service`层中接受来自 `Controller` 层的指令，根据其传入的合法参数和业务逻辑调用`Model`层对数据进行查询或更新。
