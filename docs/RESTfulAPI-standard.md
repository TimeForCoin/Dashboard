# RESTful API 设计规范

| 版本 |时间| 内容 | 贡献者 |
| ----|-- | ---- | ------ |
| 1.0 | 2019-6-24 | 初始版本 | ZhenlyChen |

本系统API依照 RESTful API 的规范设计

[具体规则](http://www.ruanyifeng.com/blog/2014/05/restful_api.html)

[API文档](http://xm.zhenly.cn//docs/swagger/?url=https://raw.githubusercontent.com/TimeForCoin/Dashboard/master/design/api.yaml#/)

## 其他约束与规范

### HTTP 动词

```text
GET（SELECT）：从服务器取出资源（一项或多项）
POST（CREATE）：在服务器新建一个资源
PUT（UPDATE）：在服务器更新资源
DELETE（DELETE）：从服务器删除资源
```

PS：这里没有使用`PATCH`作为部分的资源更新是因为微信小程序原生的请求接口并不支持`PATCH`，因此统一使用`PUT`表示更新资源。

### 状态码

```text
200 OK - [GET]：服务器成功返回用户请求的数据，该操作是幂等的（Idempotent）
201 CREATED - [POST/PUT]：用户新建或修改数据成功。
204 NO CONTENT - [DELETE]：用户删除数据成功。
400 INVALID REQUEST - [POST/PUT]：用户发出的请求有错误，服务器没有进行新建或修改数据的操作。
401 Unauthorized - [*]：表示用户没有权限（令牌、用户名、密码错误）。
403 Forbidden - [*] 表示用户得到授权（与401错误相对），但是访问是被禁止的。
404 NOT FOUND - [*]：用户发出的请求针对的是不存在的记录，服务器没有进行操作，该操作是幂等的。
500 INTERNAL SERVER ERROR - [*]：服务器发生错误，用户将无法判断发出的请求是否成功。
```
