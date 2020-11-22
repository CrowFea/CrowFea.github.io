---
title: iOS丨Alamofire 01
date: 2020-11-21 16:21:12
categories:
    - iOS
tags: 
    - iOS
mathjax: true
---

### 单例
alamofire本身是一个单例进行管理的。在`Alamofrie.swift`中并没有看见任何class，而是直接返回了session的单例。

实现单例最简洁的方式是直接使用类常量。
```swift
public static let `default` = Session()
```
swift中类常量都是在`dispatch_once`这个线程中完成的，保证了原子性。因此是线程安全的。
> The lazy initializer for a global variable (also for static members of structs and enums) is run the first time that global is accessed, and is launched as dispatch_once to make sure that the initialization is atomic. This enables a cool way to use dispatch_once in your code: just declare a global variable with an initializer and mark it private.  

### 命名空间和链式处理
```swift
Alamofire.request("https://httpbin.org/get")
    .validate(statusCode: 200..<300) //返回值验证
    .responseData { response in //解析返回的数据
        switch response.result {
        case .success:
            print("Validation Successful")
        case .failure(let error):
            print(error)
        }
    }
```
当使用大多个连续的方法时，如果设计的不够合理，可能会导致一个闭包套着另一个闭包的情况。并且每种情况的内部或许都要`guard`挡一下。

链式的连续调用十分的优雅，同样的这种写法可以参考[GitHub - mxcl/PromiseKit: Promises for Swift & ObjC.](https://github.com/mxcl/PromiseKit)

具体怎么样去实现一个链式调用呢，可以看下面的代码。对某个类做扩展，给出扩展的方法。传入的参数是一个闭包类型，返回self。
```swift
    @discardableResult
    public func validate(_ validation: @escaping Validation) -> Self {
        let validator: () -> Void = { [unowned self] in
            guard self.error == nil, let response = self.response else { return }

            let result = validation(self.request, response)

            if case let .failure(error) = result {
                self.error = error.asAFError(or: .responseValidationFailed(reason: .customValidationFailed(error: error)))
            }

            self.eventMonitor?.request(self,
                                       didValidateRequest: self.request,
                                       response: response,
                                       withResult: result)
        }

        $validators.write { $0.append(validator) }

        return self
    }
```

### URL处理
处理URL。在发起请求的时候总是需要带上请求的参数。alamofire在这里的处理可以作为参考。他将所有可以被传入的URL信息抽象为`URLConvertible`，抽出一个协议，在这个协议下支持将不同的传入对象转换为URL的方法。

```swift
public protocol URLConvertible {
    /// Returns a `URL` from the conforming instance or throws.
    ///
    /// - Returns: The `URL` created from the instance.
    /// - Throws:  Any error thrown while creating the `URL`.
    func asURL() throws -> URL
}
```

接下来只需要让可能传入的类型对这个协议做扩展就可以了。并且在这些扩展的方法中还可以额外的做一些操作。比如进行valid的判断和检测。

```swift
extension String: URLConvertible {
    /// Returns a `URL` if `self` can be used to initialize a `URL` instance, otherwise throws.
    ///
    /// - Returns: The `URL` initialized with `self`.
    /// - Throws:  An `AFError.invalidURL` instance.
    public func asURL() throws -> URL {
        guard let url = URL(string: self) else { throw AFError.invalidURL(url: self) }

        return url
    }
}

extension URL: URLConvertible {
    /// Returns `self`.
    public func asURL() throws -> URL { self }
}

```

### 发起请求
从`session.swift`中暴露出来的接口来看，使用alamofire发起请求会包含以下几个步骤：
- 包装传入的信息，形成`DataRequest`
- 执行该请求

当然每一个步骤包含的内容会更多。我们可以拆开逐步来看：
```swift
open func request(_ convertible: URLConvertible,
                      method: HTTPMethod = .get,
                      parameters: Parameters? = nil,
                      encoding: ParameterEncoding = URLEncoding.default,
                      headers: HTTPHeaders? = nil,
                      interceptor: RequestInterceptor? = nil,
                      requestModifier: RequestModifier? = nil) -> DataRequest {
        let convertible = RequestConvertible(url: convertible,
                                             method: method,
                                             parameters: parameters,
                                             encoding: encoding,
                                             headers: headers,
                                             requestModifier: requestModifier)

        return request(convertible, interceptor: interceptor)
    }
```
这是request的最顶层方法，他返回一个`DataRequest`实体，但实际上这个实体内部包含的不只有请求。请求根据类型不同细分为`DataRequest`、`DownloadRequest`等。但是每一种request都继承自Request这个类。内部包含的属性和方法是传入的`urlconvertiable`,`data`,验证有效的valid方法，didRecive方法，更新下载状态方法等。

除此以外，还可以看到在后面有对这几种request的extension，在其中有一个属性是responseString。一旦网络请求回来之后，就会立即调用`StringResponseSerializer`进行解析。
```swift
@discardableResult
    public func responseString(queue: DispatchQueue = .main,
                               dataPreprocessor: DataPreprocessor = StringResponseSerializer.defaultDataPreprocessor,
                               encoding: String.Encoding? = nil,
                               emptyResponseCodes: Set<Int> = StringResponseSerializer.defaultEmptyResponseCodes,
                               emptyRequestMethods: Set<HTTPMethod> = StringResponseSerializer.defaultEmptyRequestMethods,
                               completionHandler: @escaping (AFDataResponse<String>) -> Void) -> Self {
        response(queue: queue,
                 responseSerializer: StringResponseSerializer(dataPreprocessor: dataPreprocessor,
                                                              encoding: encoding,
                                                              emptyResponseCodes: emptyResponseCodes,
                                                              emptyRequestMethods: emptyRequestMethods),
                 completionHandler: completionHandler)
    }
```

根据传入的url、queue、adapter生成对应的task实例。同时对对应的task实例创建delegate。接下来只要按需进行task的resume方法即可。

