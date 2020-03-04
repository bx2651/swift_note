Moya通信框架的使用

### 1. 安装Moya

在Podfile文件添加命令：

```
pod 'Moya', '~> 13.0'
```

命令行安装：

```
pod install
```

### 2. 项目导入

在需要使用的文件引入

```
import Moya
```

### 3.配置API文件

```
enum HomeApi {
    case currentWeather(cityName: String)
}

extension HomeApi: TargetType {
    var baseURL: URL {
    	 //因为这个接口肯定有数据，所以进行了强制解包
        return URL(string: "http://httpbin.org")!
    }
    
    
    var path: String {
        switch self {
        //这个路径感觉写成一个变量比较好，刚开始学swift不是很熟，留个坑以后填
        case .currentWeather(_): return "/get"
        }
    }
    
    var method: Moya.Method {
    	//请求的方法，get,post,put等
        return .get
    }
    
    var headers: [String: String]? {
    	//属性存储头部字段，它们将在请求中被发送。
        return ["Content-type": "application/json"]
    }
    
    var parameterEncoding: ParameterEncoding {
    	 //编码的格式，详细见下文
        return URLEncoding.default
    }
    
    var task: Task {
    	 //如何发送/接受数据，详细见下文
        switch self {
        case let .currentWeather(cityName):
            return .requestParameters(parameters:  ["pageSize":0,"pageNum":0], encoding: parameterEncoding)
        }
    }
    
    var sampleData: Data {
    	 //编码转义？不是很懂，留坑
        return Data()
    }
}

// MARK: - Helpers
private extension String {
    var urlEscaped: String {
        return addingPercentEncoding(withAllowedCharacters: .urlHostAllowed)!
    }
    
    var utf8Encoded: Data {
        return data(using: .utf8)!
    }
}

```

#### 编码格式
Moya有 URLEncoding, JSONEncoding, and PropertyListEncoding可以直接使用。您也可以自定义编码，只要遵循ParameterEncoding协议即可，通过源码我们可以看到：

```
public typealias ParameterEncoding = Alamofire.ParameterEncoding
public typealias JSONEncoding = Alamofire.JSONEncoding
public typealias URLEncoding = Alamofire.URLEncoding
public typealias PropertyListEncoding = Alamofire.PropertyListEncoding
```
Moya的编码其实是对Alamofire进行了一层封装，当我们使用的是Get方法时，我们可以直接使用ParameterEncoding或URLEncoding；当使用post请求时，我们需要使用JSONEncoding，这就意味着我们必须要传递参数给服务器，否则会报错。

#### 发送/接受数据

task 属性代表你如何发送/接受数据，并且允许你向它添加数据、文件和流到请求体中。这儿有几种.request 类型:

* .requestPlain 没有任何东西发送
* .requestData(_:) 可以发送 Data (useful for Encodable types in Swift 4)
* .requestJSONEncodable(_:)
* .requestParameters(parameters:encoding:) 发送指定编码的参数
* .requestCompositeData(bodyData:urlParameters:) & .requestCompositeParameters(bodyParameters:bodyEncoding:urlParameters) which allow you to combine url encoded parameters with another type (data / parameters)

#### sampleData

这是TargetType协议的一个必备属性。这个属性值可以用来后续的测试或者为开发者提供离线数据支持。这个属性值依赖于 self.

基础的设置已经设置好了，现在开始使用设置获取数据并打印

```
//协议可以自己起名
protocol HomeNetworkable {

	//将上一步配置好的内容传到尖括号中
    var provider: MoyaProvider<HomeApi> { get }
    
    //这个方括号里的Home，我暂时还不知道它有什么用，姑且先放着吧，是一个文件，代码贴到下一段
    func getCurrentWeather(city name: String, completion: @escaping([Home])->())
}

//结构体名字依然随便取
struct HomeService: HomeNetworkable {
    var provider = MoyaProvider<HomeApi>() //(plugins: [NetworkLoggerPlugin(verbose: true)])
    
    func getCurrentWeather(city name: String, completion: @escaping([Home])->()) {
        provider.request(.currentWeather(cityName: name)) { result in
            switch result {
            case let .success(moyaResponse):
                let data = moyaResponse.data
                let statusCode = moyaResponse.statusCode
                print(data)
                print(statusCode)
                do {
                		//这个是用来解码数据的，我也不知道为什么，直接打印上面的data,打印出来的是数据大小？所以从网上找了这个，可以成功从控制台打印出来数据
                    let json = try JSONSerialization.jsonObject(with: data, options: .mutableContainers)
                    print(json)
                } catch {
                    print(statusCode)
                }
                
            case let .failure(error):
                print("this is error,error = \(error.localizedDescription)")
            }
        }
    }
}
```

上面不知道啥用的Home,写在这里：

```
struct Home: Codable {

    let id: Int
    let name: String
}
```

上面的service最终被传到了下面这个类当中使用

```
import UIKit

protocol CityViewModelDelegate: class {
    func isDataLoading(_ loading: Bool)
    func didReceiveCurrentWeather(_ weather: [Home])
}


class CityVM{
    
    weak var delegate: CityViewModelDelegate?
    private var currentSearchNetworkTask: URLSessionDataTask?
    private let netService: HomeService
    
    private var currentCity: [Home]?
    
    
    init(service: HomeService) {
        netService = service
        
        ready()
    }
    
    func ready() {
        startFetching(withCity: "Moscow")
    }
    
    private func startFetching(withCity name: String) {
        currentSearchNetworkTask?.cancel()
        delegate?.isDataLoading(true)
        
        netService.getCurrentWeather(city: name) {
            [weak self] weather in
            guard let sSelf = self else { return }
            sSelf.finishFetching(weather)
        }
    }
    
    
    
    private func finishFetching(_ weather: [Home]) {
        delegate?.isDataLoading(false)
        currentCity = weather
        
        delegate?.didReceiveCurrentWeather(weather)
    }  
}
```
