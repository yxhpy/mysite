# [RESTful规范建议](https://www.cnblogs.com/Erik_Xu/p/9011013.html)

# RESTful概述

RESTful是目前最流行的一种互联网软件架构。它结构清晰、符合标准、易于理解、扩展方便，所以正得到越来越多网站的采用。

REST是**Re**presentational **S**tate **T**ransfer的缩写，是Roy Thomas Fielding在他2000年的博士论文中提出的。其提出的设计概念和准则为：

\1. 网络上的所有事物都可以抽象为资源

\2. 每个资源都应该有唯一的标识（identifier），对资源的操作不会改变标识

\3. 所有的操作都是无状态的

\4. 使用标准方法（GET、POST、PUT、PATCH、DELETE）操作资源

 

# RESTful使用

| **HTTP方法** | **URI**                             | **描述**                 | **幂等** | **安全** |
| ------------ | ----------------------------------- | ------------------------ | -------- | -------- |
| GET          | /api/members                        | 获取成员列表             | 是       | 是       |
| GET          | /api/members/{id}                   | 获取指定成员             | 是       | 是       |
| POST         | /api/members                        | 创建一个成员             | 否       | 否       |
| PUT          | /api/members/{id}                   | 更新成员所有信息         | 是       | 否       |
| PATCH        | /api/members/{id}                   | 更新成员部分信息         | 是       | 否       |
| DELETE       | /api/members/{id}                   | 删除指定成员             | 是       | 否       |
| **HTTP方法** | **URI**                             | **描述**                 | **幂等** | **安全** |
| GET          | /api/groups                         | 获取群组列表             | 是       | 是       |
| GET          | /api/groups/{id}                    | 获取指定群组             | 是       | 是       |
| POST         | /api/groups                         | 创建一个群组             | 否       | 否       |
| PUT          | /api/groups/{id}                    | 更新群组所有信息         | 是       | 否       |
| PATCH        | /api/groups/{id}                    | 更新群组部分信息         | 是       | 否       |
| DELETE       | /api/groups/{id}                    | 删除指定群组             | 是       | 否       |
| GET          | /api/groups/{id}/members            | 获取指定群组下的成员     | 是       | 是       |
| GET          | /api/groups/{id}/members/{memberId} | 获取指定群组下的指定成员 | 是       | 是       |

幂等性：同一个RESTful接口的多次访问，得到的资源状态是相同的。

安全性：对该RESTful接口访问，不会使服务端资源的状态发生改变。

 

# 规范建议

\1. API尽量采用通过安全通道的HTTPS协议（https）。

　　

\2. 请求体与响应体统一通过json格式来承载，json使用Camel的命名规则，媒体类型需设置为“application/json”。

示例：

Request

　　Accept: application/json

　　Content-Type: application/json

 

Response

　　Content-Type: application/json

 

\3. 请求体与响应体统一采用UTF-8编码格式，时间统一使用UTC格式：yyyy-MM-dd'T'HH:mm:ss[.SSS]'Z'。

 

\4. URI模版：/{domain}/{service or module}/api/{version}/{resource}，URI应全为小写字母，短语单词使用“-”分隔。

| **名称**          | **说明**                               | **示例**                                             |
| ----------------- | -------------------------------------- | ---------------------------------------------------- |
| domain            | 领域名称，不需要区分领域时，可以不指定 | education(教育领域) finance(金融领域) game(游戏领域) |
| service or module | 服务或模块名                           | account(账户模块) order(订单服务) storage(库存服务)  |
| version           | 版本号                                 | v1 v2 v3                                             |
| resource          | 服务或模块内资源                       | users(用户) products（产品） members（成员）         |

 

\5. 资源增、删、改、查外的操作，采用模板：/{domain}/{service or module}/api/{version}/{resource}/action/{action}。

示例：/common/account/api/v1/users/action/login

 

# 响应消息建议

\1. 获取资源列表成功返回**200**，响应消息体中包含记录总条数、当前页码、每页记录，以及对应的资源。

示例：

Response Body

{

　　"total": xxx,

　　"pageIndex": xxx,

　　"pageSize":xxx,

　　"records":[

　　　　{ "id": xxx, "name":"xxx" },

　　　　{ "id": xxx, "name":"xxx" }

　　]

}

 

\2. 获取指定资源成功返回**200**，响应消息体中包含该资源的信息。

 

\3. 创建资源成功返回**201**，并在响应消息头中包含定位该资源的地址。

示例：

Response Headers

{
　　"pragma": "no-cache",
　　"server": "xxx",
　　"content-type": "application/json; charset=utf-8",
　　"location": "https://xxx/api/users/xxx", //资源访问地址
　　"content-length": "xxx"
}

 

\4. 资源更新成功返回**200**，并在响应消息中体返回更新后的资源内容。

 

\5. 资源删除成功返回**204**，响应消息体无内容。

 

\6. 针对**400 Bad Request**客户端错误，可以在响应消息体中扩展状态码。

示例：

Response Code

400 Bad Request

Response Body

{

　　"code": 400001,

　　"message": "用户名或密码错误"

}

 

\----------------------------------------------------------

Response Code

400 Bad Request

Response Body

{

　　"code": 400002,

　　"message": "邮箱已存在"

}

 

\----------------------------------------------------------

Response Code

400 Bad Request

Response Body

{

　　"code": 400003,

　　"message": "邮箱地址错误"

}

 

\7. 针对**5XX**的服务端错误，只在响应消息体中提供简单提示，不可打印错误日志信息。

示例：

Response Body

{

　　"message": "内部错误，请稍后再试或联系管理员"

}

 

\7. 其他客户端错误的响应码，只在响应消息体中提供相应提示。

示例：

Response Code

401 Unauthorized

Response Body

{

　　"message": "用户未登录"

}

 

\----------------------------------------------------------

Response Code

403 Forbidden

Response Body

{

　　"message": "权限不足"

}

 

\----------------------------------------------------------

Response Code

404 Not Found

Response Body

{

　　"message": "请求资源不存在或已被删除"

}

 

# 常见响应码

| **响应码**                | **说明**                                                     |
| ------------------------- | ------------------------------------------------------------ |
| 200 OK                    | 请求已成功                                                   |
| 201 Created               | 资源已创建                                                   |
| 204 No Content            | 请求已成功，但无返回内容                                     |
| 304 Not Modified          | 缓存有效                                                     |
| 400 Bad Request           | 语义有误，当前请求无法被服务器理解，请求参数错误             |
| 401 Unauthorized          | 当前请求需要用户认证（登录）                                 |
| 403 Forbidden             | 用户已认证（登录），但权限不足                               |
| 404 Not Found             | 请求源未在服务器上被发现                                     |
| 405 Method Not Allowed    | 请求方法不能被用于请求相应的资源，如使用PUT方法访问只接受POST方法的API |
| 500 Internal Server Error | 服务端内部错误                                               |
| 502 Bad Gateway           | 网关错误                                                     |
| 504 Gateway Timeout       | 网关超时                                                     |