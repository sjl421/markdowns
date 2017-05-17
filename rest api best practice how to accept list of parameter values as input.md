# rest api best practice how to accept list of parameter values as input

如果是需要删除一个条目，可以直接将需要删除的条目的id放进url里面，比如`http://example.com/posts/2016`，但是如果需要再一次请求里面删除多个条目，应该如何设计比较合理呢？我现在想到的是以下两种方法：

1. 用逗号分隔放进url里面：`http://example.com/posts/2016,2017`；
2. 将需要删除的一系列id放进请求体里面，但是似乎没有这样的标准。

## 3个回答

答案对人有帮助，有参考价值2答案没帮助，是错误的答案，答非所问

你这个方法是可以的，不过不属于标准的DELETE RESTful请求。当然有一些框架是允许delete伴随body发送的，这样你可以把所有的IDs一次放进去然后发送DELETE请求。 另外一种写法是分成2步完成，第一步发送POST请求，集合所有要删除的IDs然后返回一个header,然后在利用这个header调用DELETE请求。具体步骤如下:

```
发送POST请求，集中所有的IDs (可以存到Redis或者普通数据库)
http://example.com/posts/deletes

成功后可以返回一个唯一的头文件：

HTTP/1.1 201 created, and a Location header to:
http://example.com/posts/deletes/KJHJS675

然后可以利用Ajax直接发送DELETE请求:
DELETE http://example.com/posts/deletes/KJHJS675

这样就可以在不暴露IDs的情况下更加安全的删除相关条目。
```



## 参考答案：

http://stackoverflow.com/questions/2602043/rest-api-best-practice-how-to-accept-list-of-parameter-values-as-input



## A Step Back

First and foremost, REST describes a URI as a universally unique ID. Far too many people get caught up on the structure of URIs and which URIs are more "restful" than others. **This argument is as ludicrous as saying naming someone "Bob" is better than naming him "Joe" – both names get the job of "identifying a person" done.** A URI is nothing more than a *universally unique* name.

So in REST's eyes arguing about whether `?id=["101404","7267261"]` is more restful than `?id=101404,7267261` or `\Product\101404,7267261` is somewhat futile.

Now, having said that, many times how URIs are constructed can usually serve as a good indicator for other issues in a RESTful service. There are a couple of red flags in your URIs and question in general.

## Suggestions

1. Multiple URIs for the same resource and `Content-Location`

   > We may want to accept both styles but does that flexibility actually cause more confusion and head aches (maintainability, documentation, etc.)?

   URIs identify resources. Each resource should have **one** canonical URI. This does not mean that you can't have two URIs point to the same resource *but* there are well defined ways to go about doing it. If you do decide to use both the JSON and list based formats (or any other format) you need to decide which of these formats is the main *canonical* URI. All responses to other URIs that point to the same "resource" should include the [`Content-Location` header](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.14).

   Sticking with the name analogy, having multiple URIs is like having nicknames for people. It is perfectly acceptable and often times quite handy, however if I'm using a nickname I still probably want to know their full name – the "official" way to refer to that person. This way when someone mentions someone by their full name, "Nichloas Telsa", I know they are talking about the same person I refer to as "Nick".

2. "Search" in your URI

   > A more complex case is when we want to offer more complex inputs. For example, if we want to allow multiple filters on search...

   A general rule of thumb of mine is, if your URI contains a verb, it may be an indication that something is off. URI's identify a resource, however they should not indicate *what* we're doing to that resource. That's the job of HTTP or in restful terms, our "uniform interface".

   To beat the name analogy dead, using a verb in a URI is like changing someone's name when you want to interact with them. If I'm interacting with Bob, Bob's name doesn't become "BobHi" when I want to say Hi to him. Similarly, when we want to "search" Products, our URI structure shouldn't change from "/Product/..." to "/Search/...".

## Answering Your Initial Question

1. Regarding `["101404","7267261"]` vs `101404,7267261`: My suggestion here is to avoid the JSON syntax for simplicity's sake (i.e. don't require your users do URL encoding when you don't really have to). It will make your API a tad more usable. Better yet, as others have recommended, go with the standard `application/x-www-form-urlencoded` format as it will probably be most familiar to your end users (e.g. `?id[]=101404&id[]=7267261`). It may not be "pretty", but Pretty URIs does not necessary mean Usable URIs. However, to reiterate my initial point though, ultimately when speaking about REST, it doesn't matter. Don't dwell too heavily on it.
2. Your complex search URI example can be solved in very much the same way as your product example. I would recommend going the `application/x-www-form-urlencoded` format again as it is already a standard that many are familiar with. Also, I would recommend merging the two.

Your URI...

```
/Search?term=pumas&filters={"productType":["Clothing","Bags"],"color":["Black","Red"]}    

```

Your URI after being URI encoded...

```
/Search?term=pumas&filters=%7B%22productType%22%3A%5B%22Clothing%22%2C%22Bags%22%5D%2C%22color%22%3A%5B%22Black%22%2C%22Red%22%5D%7D

```

Can be transformed to...

```
/Product?term=pumas&productType[]=Clothing&productType[]=Bags&color[]=Black&color[]=Red

```

Aside from avoiding the requirement of URL encoding and making things look a bit more standard, it now homogenizes the API a bit. The user knows that if they want to retrieve a Product or List of Products (both are considered a single "resource" in RESTful terms), they are interested in `/Product/...` URIs.