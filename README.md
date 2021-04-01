# RxSwift-mvvm
一个基于RxSwift + Moya + mvvm架构的简单App（内容不断在丰富、优化中）

app包含了而且生成、识别、扫描、以及RxSwift的使用

对（tableView、CollectionView的使用）加入了大量注释内容，可以更加清晰了解RxSwift + Moya + mvvm的使用。

新增了对应用暗黑模式的简单适配，可以更有效的使用暗黑模式

同时增加了，对网络请求更为复杂的处理，包含 -> API超时时常、设置统一Token、设置SSL证书、设置当前请求过程等
-----------------------------------------------------------------------------------------------------------------------------------------------------------------
// MARK: - 第一种，直接就是一个请求的作用，其他都是默认设置
var weiNetworkTool = MoyaProvider<NetworkTool>()

// MARK: - 第二种，设置了多种功能超时时常、设置统一Token、设置SSL证书、设置当前请求过程等，如果程序对请求有要求，那么就要使用第二种
// MARK: ⬇️ ⬇️ ⬇️ ⬇️ ⬇️ ⬇️ ⬇️ ⬇️ ⬇️ ⬇️ ⬇️ 第二种方式 ⬇️ ⬇️ ⬇️ ⬇️ ⬇️ ⬇️ ⬇️ ⬇️ ⬇️ ⬇️
 
 //把session当参数传进去就行了
 let kProvider = MoyaProvider<NetworkTool>(endpointClosure: myEndpointClosure, requestClosure: requestClosure, session: session, plugins: [networkPlugin], trackInflights: false)

 // 设置 SSL
 let session : Session = {
    //证书数据
    func certificate() -> SecCertificate? {
        let filePath = Bundle.main.path(forResource: "存在Xcode中证书的文件名", ofType: "cer")
        if filePath == nil {
            return nil
        }
        let data = try! Data(contentsOf: URL(fileURLWithPath: filePath ?? ""))
        let certificate = SecCertificateCreateWithData(nil, data as CFData)!
        return certificate
    }
    guard let certificate = certificate() else {
        return Session()
    }
    let trusPolicy = PinnedCertificatesTrustEvaluator(certificates: [certificate], acceptSelfSignedCertificates: false, performDefaultValidation: true, validateHost: true)
    let trustManager = ServerTrustManager(evaluators: ["你证书的域名，比如www.baidu.com或者baidu.com" : trusPolicy])
    let configuration = URLSessionConfiguration.af.default
    return Session(configuration: configuration, serverTrustManager: trustManager)
 }()

 ///网络请求的基本设置,这里可以拿到是具体的哪个网络请求，可以在这里做一些设置
 private let myEndpointClosure = { (target: NetworkTool) -> Endpoint in
     ///这里把endpoint重新构造一遍主要为了解决网络请求地址里面含有? 时无法解析的bug https://github.com/Moya/Moya/issues/1198
     let url = target.baseURL.absoluteString + target.path
     var task = target.task
     /*
      如果需要在每个请求中都添加类似token参数的参数请取消注释下面代码
      👇👇👇👇👇👇👇👇👇👇👇👇👇👇👇👇👇👇👇👇👇👇👇👇👇
      */
 //    let additionalParameters = ["token":"888888"]
 //    let defaultEncoding = URLEncoding.default
 //    switch target.task {
 //        ///在你需要添加的请求方式中做修改就行，不用的case 可以删掉。。
 //    case .requestPlain:
 //        task = .requestParameters(parameters: additionalParameters, encoding: defaultEncoding)
 //    case .requestParameters(var parameters, let encoding):
 //        additionalParameters.forEach { parameters[$0.key] = $0.value }
 //        task = .requestParameters(parameters: parameters, encoding: encoding)
 //    default:
 //        break
 //    }
     /*
      👆👆👆👆👆👆👆👆👆👆👆👆👆👆👆👆👆👆👆👆👆👆👆👆👆
      如果需要在每个请求中都添加类似token参数的参数请取消注释上面代码
      */
     var endpoint = Endpoint(
         url: url,
         sampleResponseClosure: { .networkResponse(200, target.sampleData) },
         method: target.method,
         task: task,
         httpHeaderFields: target.headers
     )
     requestTimeOut = 30//每次请求都会调用endpointClosure 到这里设置超时时长 也可单独每个接口设置
     switch target {
     case .getHomeList:
         return endpoint
     default:
         return endpoint
     }
 }

 ///网络请求的设置
 private let requestClosure = { (endpoint: Endpoint, done: MoyaProvider.RequestResultClosure) in
     do {
         var request = try endpoint.urlRequest()
         //设置请求时长
         request.timeoutInterval = requestTimeOut
         // 打印请求参数
         if let requestData = request.httpBody {
             print("\(request.url!)"+"\n"+"\(request.httpMethod ?? "")"+"发送参数"+"\(String(data: request.httpBody!, encoding: String.Encoding.utf8) ?? "")")
         }else{
             print("\(request.url!)"+"\(String(describing: request.httpMethod))")
         }
         done(.success(request))
     } catch {
         done(.failure(MoyaError.underlying(error, nil)))
     }
 }

 /// NetworkActivityPlugin插件用来监听网络请求，界面上做相应的展示
 ///但这里我没怎么用这个。。。 loading的逻辑直接放在网络处理里面了
 private let networkPlugin = NetworkActivityPlugin.init { (changeType, targetType) in
     print("networkPlugin \(changeType)")
     //targetType 是当前请求的基本信息
     switch(changeType){
     case .began:
         print("开始请求网络")
     case .ended:
         print("结束")
     }
 }

// MARK: ⬆️ ⬆️ ⬆️ ⬆️ ⬆️ ⬆️ ⬆️ ⬆️ ⬆️ ⬆️ ⬆️ 第二种方式 ⬆️ ⬆️ ⬆️ ⬆️ ⬆️ ⬆️ ⬆️ ⬆️ ⬆️ ⬆️
// MARK: -


RxSwift：
1、Subject
2、Variable
3、doOn
4、Dispose
5、bindto
6、Deiver
7、变化操作
8、Scheduler
......

---------------------------------------------------------------------------------------------------------------------------------------------------------------

![截屏2021-03-26 下午5 07 07](https://user-images.githubusercontent.com/32358366/112608919-df9e7400-8e55-11eb-9b3d-fa371b98ef03.png)


---------------------------------------------------------------------------------------------------------------------------------------------------------------

多线程
GCD、Runloop、Runtime、NSOperation、NSOperationQueue等


![截屏2021-03-26 下午5 07 36](https://user-images.githubusercontent.com/32358366/112608958-eaf19f80-8e55-11eb-8507-f98cffd283ee.png)
![截屏2021-03-26 下午5 07 54](https://user-images.githubusercontent.com/32358366/112608989-f5139e00-8e55-11eb-8dff-4e48f3e234bf.png)


---------------------------------------------------------------------------------------------------------------------------------------------------------------
增加了类似今日头条、网易新闻的列表展示效果

![截屏2021-03-26 下午5 07 27](https://user-images.githubusercontent.com/32358366/112609041-0492e700-8e56-11eb-8cc8-3b687a4649cb.png)


---------------------------------------------------------------------------------------------------------------------------------------------------------------

二维码的识别、生成、扫描等

![截屏2021-03-26 下午5 07 44](https://user-images.githubusercontent.com/32358366/112609097-14123000-8e56-11eb-8732-c02bf49a2d9e.png)

当前demo后期功能会不断更新
