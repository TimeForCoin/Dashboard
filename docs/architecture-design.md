# 架构设计

## 技术选型

- 服务端：Golang

选型理由：Golang 在近些年来在服务端比较流行，相对于其他语言，Golang 具有更高的开发效率以及并行的支持，可以很容易地构建高性能的服务端程序。Golang 是一种非常现代化的语言，吸收了不同领域的编程哲学，具有函数式编程的匿名函数和闭包，同样有面向消息编程的通道和goroutine，也有垃圾回收机制。最主要的是，Golang 是一种强类型的编译型语言，同时也有很好的跨平台特性。个人觉得相对于JavaScript、Python等主流语言，Golang的开发效率和运行效率都更胜一筹。

- 路由框架：[iris](https://github.com/kataras/iris)

选型理由：Iris是一个性能强劲并且功能齐全的Web框架，架构设计良好。其MVC的结构可以有效地解耦各个部分的业务并且自由加入中间件。

数据库：MongoDB (驱动：[mongo-go-driver](https://github.com/mongodb/mongo-go-driver/))

选型理由：这里选择了NoSQL的MongoDB作为应用的数据库，具有高性能和灵活的特点。相对于传统的关系性数据库，MongoDB可以很灵活地处理一些复杂的数据，以BSON的形式存储的数据同时也与前端的JSON十分适合，以层级式的结构保存数据，无论是查询还是写入，都比起表状的结构更加便捷。同时也具有比较优越的性能，在某些场景下其性能表现会超出MySQL许多。

缓存：Redis( 驱动：[go-redis](https://github.com/go-redis/redis))

选型理由：在内存数据库领域，Redis是非常流行并且性能优越的。并且相对于其他同类产品，Redis支持更加丰富的数据结构，如Set、List、Hash等，这些数据结构可以辅助我们实现性能优越的点赞、消息、推荐系统。其过期策略也可以辅助我们实现Session数据的存储以及一些登陆、认证等功能。

日志库：[zerolog](https://github.com/rs/zerolog)

选型理由：zerolog是一个高效的Golang日志库，相对于原生的日志库，提供了输出级别的控制，并且具有更好的性能。

API 规范：RESTful API

选型理由：REST风格的API是当今大多数服务端的实践方案，具有比较强的语义性，清晰易懂，其语义可以通过请求的类型和PATH表达，对于数据的操作也非常方便，非常适合一些中小型系统的开发。

API 文档：Swagger

选型理由：Swagger 可以帮助我们构建清晰的REST风格的API文档。通过编写yaml文件，可以生成可视化的文档，同时可以定义各种数据的 Schemas ，并且可以添加各种注释，对于前后端的联调和开发都非常方便。并且支持在界面中直接构造请求，便于接口的测试。

Web 端：Vue2.6 + Webpack

选型理由：Vue在前端三大框架之中，比较容易上手并且构建出比较优秀的Web应用，其单组件的形式使得团队合作开发变得简单。

Ajax 请求：axios

选型理由：axios为Vue社区中推荐的 Ajax 请求库，可以很简单地通过API与后端通信。

微信小程序：wxss + wxml + js

选型理由：因为第三方库当前还不是非常完善，这里使用了官方的架构对小程序进行开发。

## 逻辑架构

![ch6-08-frontend-backend](architecture-design/ch6-08-frontend-backend.png)

本应用的逻辑架构由三层模型构成：

### 表示层

- 使用微信小程序作为应用的表示层，主要服务于任务参与者，除了系统的基本功能之外，主要提供消息和通知功能，便于用户及时收到通知和信息。此外还支持一般的跑腿信息任务发布。
- 使用Web作为应用的表示层，主要服务于任务发布者，除了系统的基本功能之外，主要提供问卷任务的编辑与统计功能。

### 业务逻辑层

使用Golang作为服务端语言，构建HTTP服务器通过API请求处理业务逻辑，对从表示层过来的请求进行校验与处理，然后对数据持久层进行操作(读取/写入数据)并处理，最后返回给表示层进行数据的展示。

#### 模块划分

在业务逻辑层中，也就是服务端，我们将其划分为以下的结构

![architecture](architecture-design/architecture.png)

其中`Controllers`, `Service`, `Models`再按照不同的业务划分为不同的模块

- Controller 层按照API中的**路由**划分为 10 个模块，每个模块通过**依赖注入**持有对应的业务逻辑模块
  - Article 公告接口
  - Certification 用户认证接口
  - Comment 评论回复接口
  - File 文件管理接口
  - Message 消息通知接口
  - Questionnaire 问卷相关接口
  - Session 登陆会话接口
  - Task 任务相关接口
  - User 用户信息接口
- Service 层根据**业务的相关性**划分为7个模块，所有服务模块在数据模块初始化之后统一进行初始化，持有与业务相关的数据集合的模块
  - Article 公告服务
  - Comment 评论服务
  - File 文件服务
  - Message 消息服务
  - Questionnaire 问卷服务
  - Task 任务服务
  - User 用户服务
- Model 层根据**数据库的集合**划分为14个模块，具体的划分依据参考[数据库设计](../design/db-design.md)
  - Article 公告数据库
  - Comment 评论数据库
  - File 文件数据库
  - Log 日志数据库
  - Message 消息数据库
  - Questionnaire 问卷数据库
  - Set 集合数据库
  - System 系统变量数据库
  - Task 任务数据库
  - Task Status 任务状态数据库
  - User 用户数据库
  - Cache 临时数据缓存
  - Flow 个性化信息流队列
  - Statistics 统计数据

### 数据持久层

- 使用 MongoDB 作为数据库，存储业务数据
- 使用 Redis 作为缓存，用于持久化会话以及缓存一些需要重复访问的信息。
- 使用 腾讯云的对象存储服务(COS) 保存图片/任务附件等文件

### 其他外部服务

#### Violet

Violet 是 XMatrix 团队开发的基于OAuth 2.0 的用户授权系统。可以通过授权直接获取经过手机/邮箱认证的真实用户信息。

#### WeChat

在微信小程序端通过微信的服务获取用户的OpenID以及头像、昵称等用户信息。

#### Email

通过outlook的邮箱向用户的教育邮箱发送认证邮件认证学生信息，保证平台的安全性。



## 代码结构

### 服务端目录结构

```bash
.
├── app # 服务
│   ├── app.go # 应用主体(初始化服务)
│   ├── controllers # 控制路由、数据校验
│   │   ├── article.go
│   │   ├── certification.go
│   │   ├── comment.go
│   │   ├── controllers.go
│   │   ├── file.go
│   │   ├── message.go
│   │   ├── questionnaire.go
│   │   ├── session.go
│   │   ├── task.go
│   │   └── user.go
│   ├── libs
│   │   ├── cos.go # 云对象存储
│   │   ├── email.go # 邮件服务
│   │   ├── violet.go # OAuth2 授权
│   │   └── wechat.go # 微信 授权
│   ├── utils
│   │   ├── config.go # 应用配置
│   │   ├── error.go # 错误处理
│   │   ├── utils.go # 通用方法
│   │   └── verify.go # 数据校验
│   ├── models # 数据操作以及测试
│   │   ├── article.go
│   │   ├── cache.go
│   │   ├── cache_test.go
│   │   ├── comment.go
│   │   ├── comment_test.go
│   │   ├── file.go
│   │   ├── file_test.go
│   │   ├── flow.go
│   │   ├── log.go
│   │   ├── message.go
│   │   ├── message_test.go
│   │   ├── mongo.go
│   │   ├── mongo_test.go
│   │   ├── questionnaire.go
│   │   ├── redis.go
│   │   ├── redis_test.go
│   │   ├── set.go
│   │   ├── set_test.go
│   │   ├── statistics.go
│   │   ├── system.go
│   │   ├── system_test.go
│   │   ├── task.go
│   │   ├── task_status.go
│   │   ├── task_test.go
│   │   ├── user.go
│   │   └── user_test.go
│   └── services # 业务逻辑
│       ├── article.go
│       ├── comment.go
│       ├── file.go
│       ├── message.go
│       ├── questionnaire.go
│       ├── service.go
│       ├── task.go
│       └── user.go
├── config.yaml # 程序配置
├── docker-compose.yml # 容器编排
├── dockerfile # 容器部署
├── env.ps1 # 测试环境变量
├── go.mod # Golang 库管理
├── go.sum
├── main.go # 程序入口
└── test.ps1
```

### Web端

```bash
.
├── babel.config.js
├── package-lock.json
├── package.json
├── public
│   ├── favicon.ico
│   └── index.html
├── src # 页面代码
│   ├── App.vue # 页面入口
│   ├── assets # 素材
│   │   ├── HomePage
│   │   │   ├── errand.png
│   │   │   ├── notice.png
│   │   │   ├── page1-background.jpg
│   │   │   └── questionnaire.png
│   │   ├── MissionPage
│   │   │   ├── head.jpg
│   │   │   └── test.jpg
│   │   ├── logo.png
│   │   └── logo_vue.png
│   ├── components # 组件
│   │   ├── Discover
│   │   │   ├── MissionBlock.vue
│   │   │   └── MissionCardLong.vue
│   │   ├── HomePage
│   │   │   ├── Banner.vue
│   │   │   └── DisplayPage1.vue
│   │   ├── Mission
│   │   │   ├── CreateMission
│   │   │   │   ├── FileUploader.vue
│   │   │   │   ├── ImgUploader.vue
│   │   │   │   └── TagBlock.vue
│   │   │   ├── MissionCard.vue
│   │   │   └── MissionDetail
│   │   │       ├── FileList.vue
│   │   │       ├── ImgList.vue
│   │   │       └── PlayerList.vue
│   │   ├── NavBar.vue
│   │   ├── Presentation
│   │   │   ├── Choice.vue
│   │   │   ├── Fill.vue
│   │   │   └── Score.vue
│   │   └── Question
│   │       ├── Choice.vue
│   │       ├── Fill.vue
│   │       └── Score.vue
│   ├── main.js # 程序入口
│   ├── plugins # 插件
│   │   ├── ant-design-vue.js
│   │   └── axios.js
│   ├── router.js # 路由
│   ├── services # API 服务
│   │   ├── modules
│   │   │   ├── file.js
│   │   │   ├── questionnaire.js
│   │   │   ├── task.js
│   │   │   └── user.js
│   │   └── service.js
│   ├── store # Vuex 存储
│   │   ├── modules
│   │   │   └── user.js
│   │   └── store.js
│   ├── utils # 工具
│   │   ├── modules
│   │   │   └── verify.js
│   │   └── utils.js
│   └── views # 视图页面
│       ├── About.vue
│       ├── Discover.vue
│       ├── Home.vue
│       ├── Mission.vue
│       ├── MissionCenter
│       │   ├── MissionInformation.vue
│       │   └── MissionTypeChoice.vue
│       ├── MissionDetail.vue
│       ├── Presentation.vue
│       ├── Questionnaire.vue
│       └── User.vue
├── violet.config.js
└── vue.config.js
```

### 微信小程序

```bash
.
├── README.md
├── app.js # 程序入口
├── app.json
├── app.wxss
├── images # 图片资源
│   └── ...
├── miniprogram_npm
│   └── moment
│       └── ...
├── package-lock.json
├── package.json
├── pages # 页面
│   ├── AddItem
│   │   ├── AddItem.js
│   │   ├── AddItem.json
│   │   ├── AddItem.wxml
│   │   └── AddItem.wxss
│   ├── AddedItems
│   │   ├── AddedItems.js
│   │   ├── AddedItems.json
│   │   ├── AddedItems.wxml
│   │   └── AddedItems.wxss
│   ├── CollectList
│   │   ├── CollectList.js
│   │   ├── CollectList.json
│   │   ├── CollectList.wxml
│   │   └── CollectList.wxss
│   ├── Comment
│   │   ├── Comment.js
│   │   ├── Comment.json
│   │   ├── Comment.wxml
│   │   └── Comment.wxss
│   ├── Detail
│   │   ├── Detail.js
│   │   ├── Detail.json
│   │   ├── Detail.wxml
│   │   └── Detail.wxss
│   ├── Message
│   │   ├── Message.js
│   │   ├── Message.json
│   │   ├── Message.wxml
│   │   └── Message.wxss
│   ├── MessageDetail
│   │   ├── MessageDetail.js
│   │   ├── MessageDetail.json
│   │   ├── MessageDetail.wxml
│   │   └── MessageDetail.wxss
│   ├── ParticipateTask
│   │   ├── ParticipateTask.js
│   │   ├── ParticipateTask.json
│   │   ├── ParticipateTask.wxml
│   │   └── ParticipateTask.wxss
│   ├── Questionnaire
│   │   ├── Questionnaire.js
│   │   ├── Questionnaire.json
│   │   ├── Questionnaire.wxml
│   │   └── Questionnaire.wxss
│   ├── SearchResult
│   │   ├── SearchResult.js
│   │   ├── SearchResult.json
│   │   ├── SearchResult.wxml
│   │   └── SearchResult.wxss
│   ├── index
│   │   ├── index.js
│   │   ├── index.json
│   │   ├── index.wxml
│   │   └── index.wxss
│   └── userInfo
│       ├── userInfo.js
│       ├── userInfo.json
│       ├── userInfo.wxml
│       └── userInfo.wxss
├── project.config.json
├── services # 请求封装
│   └── server.js
├── sitemap.json
└── utils # 通用工具方法
    └── util.js

```

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

## 软件设计技术

### Object-Oriented Programming

在Golang中，通过`struct`可以定义对象，

