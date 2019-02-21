# PathSource REST API 开发指导手册
## PathSource 后端团队出品

<div style="font-size:150%">
文档编辑: Fenix Wang
</div>

# PathSource REST API 开发指导手册
## 1 目的
本文档旨在帮助开发人员快速、准确的开发符合规范的REST API接口, 并且帮助前端开发人员通过一致的方式使用REST API来实现业务。

## 2 目录

- [3 开发符合标准的API准则](#3-开发符合标准的API准则)
- [4 使用RESTful风格的url地址和提交方式](#4-使用RESTful风格的url地址和提交方式)
- [5 启用SSL](#5-启用SSL)
- [6 文档化](#6-文档化)
- [7 版本化](#7-版本化)
- [8 针对查询结果的过滤、排序和查询](#8-针对查询结果的过滤、排序和查询)
- [9 选择性的返回字段](#9-选择性的返回字段)
- [10 更新和创建操作应该返回资源描述](#10-更新和创建操作应该返回资源描述)
- [11 HATEOAS](#11-HATEOAS)
- [12 使用JSON作为返回数据格式](#12-使用JSON作为返回数据格式)
- [13 snake_case还是camelCase](#13-snake_case还是camelCase)
- [14 针对通过浏览器查看API返回的结果进行格式化并启用gzip压缩功能](#14-针对通过浏览器查看API返回的结果进行格式化并启用gzip压缩功能)
- [15 默认情况下不要封装返回结果除非有必要](#15-默认情况下不要封装返回结果除非有必要)
- [16 JSON编码的POST,PUT和PATCH请求体](#16-JSON编码的POST,PUT和PATCH请求体)
- [17 分页](#17-分页)
- [18 附加相关数据字段](#18-附加相关数据字段)
- [19 重写HTTP方法](#19-重写HTTP方法)
- [20 请求限制](#20-请求限制)
- [21 鉴权](#21-鉴权)
- [22 缓存](#22-缓存)
- [23 错误处理](#23-错误处理)
- [24 HTTP状态码](#24-HTTP状态码)

## 3 开发符合标准的API准则
- 应该符合web标准
- 对于开发人员足够友好并且能够通过浏览器浏览API
- API应保持简单、直观和一致
- API的开发应保证足够的灵活性
- API的开发应遵循高效性,易于维护

## 4 使用RESTful风格的url地址和提交方式
REST的核心理念是通过不同的动词将逻辑资源的操作进行分离。这些用于操作资源的动词包括(GET, POST, PUT, PATCH, DELETE),每个动词在REST规范中代表了不同的操作场景,具体内容会在下面的内容提及。

所操作的资源在命名时必须使用名词, 例如:user, token, occupation等。

通过不同的操作动词来区分请求的目的,例如下面的例子:

- `GET /users` - 获取用户的列表
- `GET /users/324` - 获取用户ID为324的用户
- `POST /users` - 创建(注册)用户
- `PUT /users/324` - 更新用户ID为324的用户信息
- `PATCH /users/324` - 更新用户ID为324的部分信息
- `DELETE /users/324` - 删除用户ID为324的信息

**资源名称使用单数形式还是复数形式?**

一律使用复数形式。

**Sever 目录下的 model 和 json 使用单数形式还是复数形式?**(Senan 注)

一律使用单数形式。

**如何处理模型之前的关系?**

我们考虑一下学校和专业之间的关系, 如下所示:

- `GET /schools/12/majors` - 获取ID为12的学校包含的所有专业
- `GET /schools/12/majors/5` - 获取ID为12的学校, 专业编号为5的信息
- `POST /schools/12/majors` - 为ID为12的学校创建一个新的专业
- `PUT /schools/12/majors/5` - 更新学校ID为12,专业ID为5的专业信息
- `PATCH /schools/12/majors/5` - 更新学校ID为12,专业ID为5的部分专业信息
- `DELETE /schools/12/majors/5` - 删除学校ID为12,专业ID为5的专业信息

**有哪些情况不适用于上述规则?**
  1. 例如有一个叫做`activate`的操作使用PATCH提交方式来更新某个模型的`activated`字段信息。
  2. 例如GitHub允许你使用`PUT /gists/:id/star`来给某个代码片段打星, 使用`DELETE /gists/:id/star`来取消某个代码片段的打星状态。这里使用了类似关系表示法来更新gist模型的打星状态。
  3. 例如搜索操作`GET /search`, 在这种场景下我们无法准确地将该操作映射到某个模型, 因此这样定义REST API地址是可接受的。

## 5 启用SSL

为所有的REST API启用SSL, **没有例外!**

## 6 文档化

使用Swagger Editor将REST API文档化并且可以通过浏览器查看API定义和调用API查看输出结果。

## 7 版本化

应该始终版本化REST API, 版本化可以帮助项目进行快速迭代。

通常有两种提供版本化的方式, 一种是将版本号通过header传递,另外一种是在url地址中加入版本部分`api.pathsource.com/v1/users`。考虑到可读性, 这里我们使用第二种方式表示版本。

## 8 针对查询结果的过滤、排序和查询

尽量保持短的资源请求地址。复杂的过滤、排序和高级查询操作可以通过**简短的基地址+查询参数**来实现, 如下所示:

**过滤:** 使用唯一的请求参数来实现过滤。例如: 当我们请求用户列表时`/users`, 可能需要根据参数`online`来筛选出目前在线的用户, 那么我们可以使用`GET /users?online=true`来获取在线用户。这里, online就是一个用于筛选用户的请求参数。

**排序:** 和过滤的情况类似, 使用一些标示符来决定排序规则, `-`减号表示降序排序, 如果需要针对多列进行排序, 那么可以使用`,`来分隔各个字段。例如:

- `GET /users?sort=-vip_expiration_datetime` - 按会员过期时间降序排序所有获取的用户列表
- `GET /users?sort=-vip_expiration_datetime,created_at` - 先按会员过期时间降序排列再按创建时间升序排列来获取用户列表

**搜索:** 有时我们需要做一些全文检索的功能, 通常我们会定义名称为`q`的查询参数, 我们可以组合多个请求参数来完成一次全文检索的查询, 例如:
- `GET /articles?sort=-post_datetime` - 按照发布时间降序排序来获取所有文章
- `GET /articles?state=delete&sort=-post_datetime` - 获取所有已删除的文章并按发布时间将序排列
- `GET /articles?q=The&20Best&20Job&state=normal&sort=-post_datetime` - 获取所有文章含有"The Best Job"关键词并且状态为normal的文章按发布时间降序排列

**为常用的查询取别名**

如果某些查询非常被使用到, 可以考虑将其定义为一个单独的路由地址, 例如我们经常会去查询最近热门的职业, 那么可以考虑定义请求`GET /jobs/recently_hot`

## 9 选择性的返回字段

不需要总是返回完整的数据结构, 只需要返回当前业务所需字段的最小集合,这样可以减小网络数据交换的大小, 提高API调用的效率。

使用字段名作为请求参数来获取特定字段的数据, 例如:

`GET /users?fields=id,name,age,birthday&online=true&sort=-created_at`

## 10 更新和创建操作应该返回资源描述

PUT, POST, PATCH操作有时会更新那些并没有出现在请求参数的字段(例如类似created_at或updated_at这样的时间戳)。为了避免重复提交同样的数据, 在更新和创建操作时, API调用总是应该返回被更新或被创建的资源。

在创建操作中我们使用POST提交方式, 在返回中使用http状态201表示创建成功([HTTP 201 STATUS CODE](http://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html#sec9.5))并且把刚刚创建的资源的地址放在返回的头信息中([Location Header](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.30))

## 11 HATEOAS

HATEOAS的全称是"超媒体应用程序状态引擎"(**H**ypermedia **a**s **t**he **E**ngine **o**f **A**pplication **S**tate)。

我们看下面的例子:

```json
{
  "id": 711,
  "schoolName": "Post University",
  "streetAddress": "800 Country Club Road",
  "zip": "06723",
  "curriculums": [
    {
      "curriculumId": 581651,
      "curriculumName": "A.S. in Accounting",
      "details": [
        {
          "uri": "/api/v1/schools/711/curriculums/23"
        }
      ]
    }
  ]
}
```
## 12 使用JSON作为返回数据格式

**不要使用XML!**

![JSON和XML流行度比较](http://www.vinaysahni.com/images/201305-xml-vs-json-api.png)

如果一些第三方合作伙伴的API返回的数据格式是XML, 那么请确保设计REST API路由地址时始终以.xml结尾, 例如: `https://api.cosmos.com/articles.xml`

## 13 snake_case还是camelCase

一个通用准则是, 如果使用JSON作为数据交换格式, 那么应该使用camelCase的命名规则, 从使用的语言层面来讲, 如果使用C#, Java或javascript那么使用camelCase, 如果使用Python, Ruby则应使用snake_case。

## 14 针对通过浏览器查看API返回的结果进行格式化并启用gzip压缩功能

默认情况下通过浏览器调用API查看返回的结果去除了多余的空格以保证较小的网络传输字节数, 但相比未启用gzip压缩来说, 去除了多余空格带来的坏处远远大于好处, 所以在开发API时, 默认应返回对浏览器有好的JSON呈现, 我们可以通过下面的例子了解他们之间的差异:

```
$ curl https://api.github.com/users/veesahni > with-whitespace.txt
$ ruby -r json -e 'puts JSON JSON.parse(STDIN.read)' < with-whitespace.txt > without-whitespace.txt
$ gzip -c with-whitespace.txt > with-whitespace.txt.gz 
$ gzip -c without-whitespace.txt > without-whitespace.txt.gz
```
在上面四个请求返回的结果得出的文件大小:

- without-whitespace.txt - 1252字节
- with-whitespace.txt - 1369字节
- without-whitespace.txt.gz - 496字节
- with-whitespace.txt.gz - 509字节

所以, **必须针对返回的数据做gzip压缩!**

## 15 默认情况下不要封装返回结果除非有必要

先看一个例子, 许多API会像如下方式返回结果:

```json
{
  "data" : {
    "id" : 123,
    "name" : "John"
  }
}
```
### **什么情况下需要封装返回结果?**
- 需要使用jsonp实现跨域请求时
- 客户端无法拿到HTTP HEADERS时

## 16 JSON编码的POST,PUT和PATCH请求体

API接口实现必须接受通过JSON编码的POST, PUT和PATCH请求并且把`Content-Type`设置为`application/json`, 当接口拿到的请求对象不是json时应抛出错误编码415(Unsupported Media Type)。

## 17 分页

通过[Link Header](http://tools.ietf.org/html/rfc5988#page-6)来实现分页:

```
Link: <https://api.github.com/user/repos?page=3&per_page=100>; rel="next", <https://api.github.com/user/repos?page=50&per_page=100>; rel="last"
```

还可以参考facebook的[基于游标的分页实现](https://developers.facebook.com/docs/reference/api/pagination/)和[GitHub的分页实现](http://developer.github.com/v3/#pagination)

上述分页实现不一定满足所有情况的需求, 如果我们需要获取数据总数count, 那么可以自定义一个叫做`X-Total-Count`的header来保存总条数。

## 18 附加相关数据字段

有时我们为了实现业务逻辑需要调用多个API接口来获取相关数据, 这里增加了网络请求的负担并且使客户端的逻辑复杂化。因此, 可以考虑使用请求参数`embed`或`expand`将关联的数据字段"组装"到现有模型上, 例如:

`GET /articles/34?embed=author.name,last_comment_user`

调用后的返回结果可能会是如下的结构:

```json
{
  "id" : 34,
  "title" : "How To Design REST API?",
  "publish_datetime" : "April 29 2016",
  "author" : {
    "name" : "Fenix Wang"
  },
  "last_comment_user": {
   "id" : 42,
   "name" : "Smith Brown"
  }
}
```

上述实现方式仅是建议或是一种实现方案, 具体的实现方案还需根据特定的业务复杂度决定。

## 19 重写HTTP方法

一些HTTP客户端仅能够支持GET和POST请求, 为了兼容这种情况, API需要提供某种方式来重写HTTP方法, 这里可以通过设置请求头的`X-HTTP-Method-Override`来实现重写。

重写HTTP方法仅被POST请求所接受, **绝对不要针对GET请求做HTTP方法重写**, 这可能在不经意的情况下修改掉数据。详细讨论可以参考stackexchange的这篇[帖子](http://programmers.stackexchange.com/questions/188860/why-shouldnt-a-get-request-change-data-on-the-server)

## 20 请求限制

为了防止恶意请求, REST API需要对请求加以限制, 当某个客户端在指定市场范围内发出了超过限制的请求, 那么应该返回[429错误](http://tools.ietf.org/html/rfc6585#section-4)。

尽管如此, 考虑到用户体验, 应该在每次调用返回的头部信息中加入如下信息:

- `X-Rate-Limit-Limit` - 当前时间周期允许的调用总次数
- `X-Rate-Limit-Remaning` - 当前时间周期剩余的调用次数
- `X-Rate-Limit-Reset` - 当前时间周期剩余秒数

## 21 鉴权

RESTful API是无状态的。这意味着请求鉴权不应该依赖cookies或sessions。因此, 每次请求应基于某种鉴权凭证。

之前提到过, REST API应始终启用SSL, 鉴权凭证可以通过随机生成的access token填入HTTP基本授权的用户名字段。

针对第三方鉴权,通常会使用到OAuth 2来实现, 具体实现可参考各个服务提供商的API文档。

## 22 缓存

HTTP本身提供了一套内建的缓存框架。这里要做的仅仅是在response headers中包含一些额外的header信息并且去验证request headers里面的信息。

有两种方式来验证数据是否缓存[ETag](http://en.wikipedia.org/wiki/HTTP_ETag)和[Last-Modified](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.29)

**ETag:** 当创建请求时, http头的ETag键会包含一个哈希值。在请求同一个资源时,只要返回的资源发生变化,那么ETag的值相应的也会发生变化。如果请求头中的`If-None-Match`的值和响应头中的ETag值相同, 那么API应返回状态码`304 Not Modified`。

**Last-Modified:** 和ETag类似, 只是Last-Modified使用了时间戳机制, 在相应头中的`Last-Modified`会和请求头中的`If-Modified-Since`进行验证来决定是否从缓存中获取资源而不是重新加载。

## 23 错误处理

和其他资源一样, 错误应设计为结构化的资源对象。不论API返回的正确与否,都应该保证返回对应的http状态码。错误情况的http状态码主要分为400系列和500系列, 其中400系列的错误码主要是由于客户端的请求导致的错误, 而500系列错误码主要是由于服务器端导致的错误。

在设计错误对象数据结构时, 通常需要一个错误编码, 一个错误消息和一个错误详情。如下所示:
```json
{
  "code" : 1234,
  "message" : "Something bad happened :(",
  "description" : "More details about the error here"
}
```
如果是验证错误, 则应根据具体的错误将字段进行分解, 如下所示:
```json
{
  "code" : 1024,
  "message" : "Validation Failed",
  "errors" : [
    {
      "code" : 5432,
      "field" : "first_name",
      "message" : "First name cannot be blank"
    },
    {
       "code" : 5622,
       "field" : "password",
       "message" : "Password must be greater than 8 characters"
    }
  ]
}
```

## 24 HTTP状态码

HTTP协议定义了一组有实际意义的状态码,大部分的API调用可以直接使用这些状态码,特定的业务代码需要由开发人员来自行定义, 常见的http状态码如下所示:

- `200 OK` - 表示GET, PUT, PATCH, DELETE请求成功获得响应, POST请求也可以返回200, 不过不能用于创建资源, 因为创建资源的成功返回由单独的状态码表示。
- `201 Created` - 使用POST请求创建资源的成功返回。应该结合[Location header](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.30)一起使用。
- `204 No Content` - 表示无内容的成功返回, 比如执行的删除的请求
- `304 Not Modified` - 当从缓存返回数据是应返回此状态码
- `400 Bad Request` - 当请求对象不正确时会返回该错误码, 比如无法解析请求的json对象或者json对象中的数据类型不正确。
- `401 Unauthorized` - 当没有提供或没有提供正确的鉴权凭证时会返回该错误码
- `403 Forbidden` - 当鉴权成功但该用户没有权限访问该资源时应返回该错误码
- `404 Not Found` - 当请求了不存在的资源时应返回该错误码
- `405 Method Not Allowed` - 当被请求的HTTP方法不被支持时应返回该错误码, 比如并没有针对路由地址`http://api.cosmos.com/users`实现`DELETE`操作时发起了`DELETE`操作则会返回该错误码
- `410 Gone` - 当特定的路由地址不再可用时应返回该错误码, 比如某个REST API已经废弃时,调用该接口应返回该错误码
- `415 Unsupported Media Type` - 当请求的资源类型和实际的资源类型不一致时会返回该错误码, 比如资源应返回xml的资源但在请求时`Content-Type`设置为了`application/json`则会返回该错误码
- `422 Unprocessable Entity` - 用于验证错误, 当后端代码验证请求参数不合法时应返回该错误码
- `429 Too Many Request` - 由于达到了请求限制阈值时应返回该错误码

