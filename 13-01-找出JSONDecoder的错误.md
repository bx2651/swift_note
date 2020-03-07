利用JSONDecoder，可以将API请求回来的数据转换成遵从Decodeable协议的模型数据：


我们先进行一个网络请求

```
        Alamofire.request("https://httpbin.org/get")
            .validate(contentType: ["application/json"])
            .responseData { response in
                switch response.result {
                case .success:
                    debugPrint(response)
                case let .failure(error):
                    print(error)
                }
        }
```

我们假设请求回来的数据长这样：

```
{
	"status":"success",
	"url":"https://httpbin.org/get"
}
```


根据请求结果，我们可以定义一个结构体，这个结构体遵从codable协议：

```
struct Msg: Codable {
   var status: String
   var message: URL
}
```

现在，我们就可以使用JSONDecoder对数据进行解析：

```
        Alamofire.request("https://httpbin.org/get")
            .validate(contentType: ["application/json"])
            .responseData { response in
                switch response.result {
                case .success:
                    debugPrint(response)
                    let data = try? JSONDecoder().decode(Msg.self,from:response)
                case let .failure(error):
                    print(error)
                }
        }
```

解析完成后，我们就可以对数据进行使用了。

但是这样会有一个问题：如果我们不小心犯了某个小错误，比如在结构体中将某个单词拼写错误，或者某个返回值是空值，那么JSONDecoder会解码失败，按照上面的写法，它**不会提示我们哪里错了，而是直接解析为Nil**,这种错误往往很难发现，所以最好能有一个办法能告诉我们哪里出错了，这个时候我们就需要用到do-catch搭配try一起使用：

```
        Alamofire.request("https://httpbin.org/get")
            .validate(contentType: ["application/json"])
            .responseData { response in
                switch response.result {
                case .success:
                    debugPrint(response)
                    do{
                    	let data = try JSONDecoder().decode(Msg.self,from:response)
                    }catch{
                    	print(error)
                    }
                case let .failure(error):
                    print(error)
                }
        }
```

当try抛出错误时，我们可以在catch里利用自动生成的常数error得到错误信息。

当然，我们也要知道，哪些情况会导致JSONDecoder解析错误：

1. property的名字拼错，和JSON数据里的key值不一样:比如，我们将message拼写成了massage:

```
struct Msg: Codable {
   var status: String
   var massage: URL
}

```
抛出错误：

```
keyNotFound(CodingKeys(stringValue: "massage", intValue: nil), Swift.DecodingError.Context(codingPath: [], debugDescription: "No value associated with key CodingKeys(stringValue: \"massage\", intValue: nil) (\"massage\").", underlyingError: nil))

```

2. JSON数据可能会为空:比如，服务器返回给我们的数据中，message值是空的，在这种时候，我们最好将结构体中的message定义为可选类型。
3. peoperty的数据类型错误：比如，status值本来应该是String,但是我们将它错误的写成了Int：

```
typeMismatch(Swift.Int, Swift.DecodingError.Context(codingPath: [CodingKeys(stringValue: "status", intValue: nil)], debugDescription: "Expected to decode Int but found a string/data instead.", underlyingError: nil))

```

tryMismatch表示property的类型错误，CodingKeys后的文字说明，有问题的property是status,DebugDescription进一步说明类型不合的原因，*Expected to decode Int but found a string/data instead*明确说明了String类型被定义成了Int类型。