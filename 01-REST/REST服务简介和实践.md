# REST服务简介和实践

## 1 资源识别
REST服务中的API将不再以执行了什么动作为中心，而是以资源为中心。一些对资源的通用操作有添加，取得，修改，删除，以及对符合特定条件的资源进行列表操作。

### 1.1 识别方法
（关注动作后的宾语）我们对某个操作不要再关注它所执行的动作，而是关心它所操作的宾语。通常情况下，该宾语就会是REST系统中的资源。
动作可能并不存在着它所操作的宾语，变化的实体实际上就是一种资源。
在抽象资源的过程中，我们需要按照自顶向下的方式，即首先辨识出系统中的最主要资源，然后再辨识这些主要资源的子资源
如果一个资源是主资源，那么其可以被不同的资源实例包含引用而不会产生歧义。而如果一个资源是子资源，那么被不同的资源实例引用可能会产生歧义。

### 1.2 判断REST服务所定义的资源是否合理
考虑对该资源的CRUD是否有意义，从而验证资源的定义是否合理。
检查资源是否需要除CRUD之外的动词来操作。该方法用来检查资源中是否还有子资源没有被抽象。
资源是否是被整体使用，创建和删除。该方法用来探测是否一个子资源应该是一个主资源。如果在删除一个资源的时候，其子资源还可以被其它资源重用，那么该子资源实际上具有较高的重用性，应该是一个主资源。

## 2 资源的URL设计

在HTTP中，一个URL主要由以下几个部分组成：
协议。即HTTP以及HTTPS。
主机名和端口。如www.egoods.com:8421
资源的相对路径。如/api/categories。
请求参数。即由问号开始的由键值对组成的字符串：?page=1&page_size=20

在为一个资源设计其所对应的URL时，我们需要着重考虑第三部分和第四部分组成。

API必须有版本的概念
url中大小写不敏感，不要出现⼤写字⺟
使-不是使_做url路径中的字符串连接
有一份文档

### 2.1 方法步骤

明确所有的资源都应该置于举个例子如/api这样一个相对路径下之后，为资源定义对应的URL
例如egoods网站中的产品分类就是一个主资源，我们会为其分配如下URL：/api/categories
注：每类主资源都将拥有一个特定于该类资源的URL。这些URL就对应着相应资源实例的集合
某个主资源类型中的特定实例，那么我们就需要在该类主资源所对应的URL之后添加该实例的ID。如egoods网站中的食品分类的ID为1：/api/categories/1
一个资源实例中还可能拥有子资源，包含一个子资源的集合：/api/items/23456/shipments
仅仅包含一个子资源集合中的一个子资源： /api/items/23456/shipments/87256
仅仅包含一个子资源且子资源只有一个： /api/items/23456/discount

### 2.2 资源在URL中是单数还是复数

如果一个URL所对应的资源是使用复数表示的，那么该类型的资源可能有多个。对该URL发送Get请求可能返回该资源的一个列表。
反之，如果一个URL所对应的资源是使用单数表示的，那么该类型的资源将只有一个，因此对该URL发送Get请求将只返回该资源的一个实例。

### 2.3 使用合适的动词



1. GET 查询
2. POST 新增
3. PUT 更新
4. DELETE 删除
5. patch 更新部分资源

**CRUD：**

> 在一个资源的生命周期之内常常会发生一系列通用事件（CRUD）。
> 一开始，一个资源并不存在。只有用户或REST服务创建了该资源以后其才存在，也即是上面所列出的通用事件中的C，Create。
> 在一个资源创建完毕以后，用户可能会从服务端请求该资源的表示，也就是上面所列出的通用事件的R，Retrieve。
> 在特定情况下，用户可能决定要更新该资源，因此会使用上面的通用事件中的U，即Update来更新资源。
> 而在资源不再需要的时候，用户可能需要通过通用事件D，即Delete来删除该资源。同时用户有时也需要列出属于特定类型资源的资源实例，即通过List操作来得到属于特定类型的资源的列表。

POST动词会在目标URI之下创建一个新的子资源。例如在向服务端发送下面的请求时，REST系统将创建一个新的分类

```
POST /api/categories
Host: www.egoods.com
Authorization: Basic xxxxxxxxxxxxxxxxxxx
Accept: application/json

{
   "label" : "Electronics",
   ……
}
```

PUT则是根据请求创建或修改特定位置的资源。此时向服务端发送的请求的目标URI需要包含所处理资源的ID：

```
POST /api/categories/8fa866a1-735a-4a56-b69c-d7e79896015e
Host: www.egoods.com
Authorization: Basic xxxxxxxxxxxxxxxxxxx
Accept: application/json

{
   "label" : "Electronics",
   ……
}
```

怎么区分是用post还是put？

> 首先就是资源的ID是如何生成的。如果希望客户端在创建资源的时候显式地指定该资源的ID，那么就需要使用PUT。而在由服务端为该资源自动赋予ID的时候，我们就需要在创建资源时使用POST。
> 在决定使用PUT创建资源的时候，防止资源URI与其它资源所具有的URI重复的任务需要由客户端来保证。在这种情况下，客户端常常使用GUID/UUID作为将资源的ID。但是到底使用GUID/UUID还是由服务端来生成ID不仅仅和REST有关，更会对数据库性能等多个方面产生影响。因此在决定使用它们之前要仔细地考虑清楚。

同时需要注意的是，因为REST要求客户只可以通过服务端返回结果中所包含的信息来得到下一步操作所需要的信息，因此客户端仅仅可以决定资源的ID，而URI中的其它部分则需要从之前得到的响应中取得。

使用put还是post需要考虑的问题？

1. PUT的等幂性是否对REST系统的设计有所帮助？由于在同一个URI上调用两次PUT所得到的结果相同。因此用户在没有接到PUT请求响应时可以放心地重复发送该响应。这在网络丢包较为严重时是一个非常好的功能。反过来，在同一个URI上调用两次POST将可能创建两个独立的子资源。
2. 否将资源的创建和更新归结为一个API可以简化用户对REST服务的使用？用户可以通过PUT动词来同时完成创建和更新一个资源这两种不同的任务。这样的好处在于简化了REST服务所提供的接口，但是反过来也让一个API执行了两种不同的任务，在一定程度上违反了API设计时每个API都需要有明确的意义这一原则。

除此之外，另外一对类似的动词则是PUT和PATCH。两者之间的不同则在于PUT是对整个资源的更新，而PATCH则是对部分资源的更新。而该动词的局限性则在于对该动词的支持程度。毕竟在某些类库中并没有提供原生的对PATCH动词的支持。

### 2.4 使用标准的状态码

到底如何组织这些响应代码需要用户根据所编写的项目决定，尤其是该产品的使用者来决定。在定义一个平台时，尽量使用更多的HTTP响应代码，因为用户极有可能通过该平台编写自己的第三方软件。而在为一个普通的产品定义REST API时，将响应代码定得非常专业可能反而导致易用性的下降。

IANA提供了一张各个状态码所对应的RFC协议的列表，从而可以很容易地找到各个状态码所对应的RFC协议以及其所在的章节。该列表的地址为：http://www.iana.org/assignments/http-status-codes/http-status-codes.xhtml

### 2.5 选择适当的表示结构

在一个基于HTTP的REST服务中，我们可以使用JSON，也可以使用XML，甚至是自定义的MIME类型来表示资源。这些表现形式常常是等效的。
客户端和服务端对所使用负载类型的协商通常都按照协议所规定的标准协商过程来完成。例如对于一个基于HTTP的REST服务，我们就需要使用Accept头来标示客户端所可以接受的负载类型。

1. GET /api/categories
2. Host: www.egoods.com
3. Authorization: Basic xxxxxxxxxxxxxxxxxxx
4. Accept: application/json

而在服务端支持的情况下，返回的响应就将使用该MIME类型组织其负载：
1. HTTP/1.1 200 OK
2. Content-Type: application/json
3. Content-Length: xxx

### 2.6 负载的自描述性

REST系统中所传递的各个消息的负载需要提供足够的用于操作该资源的信息，如如何对资源进行添加，删除以及修改等操作，并可以根据负载中所包含的对其它各资源的引用来访问各个资源。这也对负载的自描述性提出了更高的要求。

**如何自定义资源的json**

> 在一个基于HTTP的REST系统中，我们常常在资源的引用中包含一定量的描述信息。这主要因为两点：
> 提高性能。在一个对资源的引用中添加了用于显示的属性后，客户端页面可以避免再次通过url发送请求得到资源的具体描述，以得到用于显示的信息。
> 自描述性的要求。如果一个资源中包含了一个对其它资源进行引用的数组，那么用户就需要通过该标签来决定到底访问哪个被引用的资源。
> 有时候，一个资源可能并不支持特定用户执行某个操作。例如一个管理员所创建的资源可能对普通用户只读。在这种情况下，我们需要禁止普通用户对该资源的修改和删除。为了能明确地告知用户他所具有的权限，我们需要一个能显式地标示用户可以在一个资源上所执行操作的组成。在REST响应中，这种组成被称为Hypermedia Controls。例如对于一个普通用户，其从egoods中所返回的分类列表将如下所示：


我们通过actions域显式地标示了用户可以在各个类别上所能执行的操作。

``` json
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: xxx
 
[
   {
      "label" : "Food",
      "uri" : "/api/categories/1",
      "actions" : ["GET", "PUT", "DELETE"]
   }, {
      "label" : "Clothes",
      "uri" : "/api/categories/2",
      "actions" : ["GET", "PUT", "DELETE"]
   }
   ...
   {
      "label" : "Electronics",
      "uri" : "/api/categories/25",
      "actions" : ["GET", "PUT", "DELETE"]
   }
]

```


### 2.7 版本管理

如果一个资源由不同的URI标示其不同的表现形式，那么用户将无法通过一个响应中所标示的URI得到其它URI所指向的表示形式。而且在URI中添加了有关版本的信息也就标示着其可能会随着时间的推移发生变化。

**方法一：**

一种使用独立URI的方法是基于Accept头。

在一个请求中，我们常常标明了Accept头，以标示客户端希望得到的表现形式。在该头中，用户可以添加所请求的资源的版本信息：

```
GET /api/categories/1
Host: www.egoods.com
Authorization: Basic xxxxxxxxxxxxxxxxxxx
Accept: application/vnd.ambergarden.egoods-v3+json

```


而在接收到该请求之后，服务端将返回该资源的第三个版本

```
HTTP/1.1 200 OK
Content-Type: application/vnd.ambergarden.egoods-v3+json
Content-Length: xxx

{
   "uri" : "/api/categories/1",
   "label" : "Food",
   ……
}

```

**方法二：**

⼀种基于重定向的解决⽅案也被提出。该⽅案允许⼀个REST系统提供多个版本的API，并在URI中标明版本号：

```
/api/v2/categories
/api/v1/categories

```


这样⽤⼾可以选择使⽤特定版本的REST API来实现客⼾端功能。由于其使⽤固定版本的API，因此并不存在着⼀个资源有多种表⽰，进⽽违反了HATEOAS约束的问题。

## 3 详细RESTful API接口设计参考
http://www.ruanyifeng.com/blog/2018/10/restful-api-best-practices.html



[REST服务简介和实践](https://blog.csdn.net/u_hcy2000/article/details/122216769)