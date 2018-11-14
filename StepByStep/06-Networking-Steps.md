Start at `5893d67`:

#Environment and Endpoint

- Create Environment, Endpoint `1b0b4a6`

```swift
public protocol Endpoint {

    /// The path for the request
    var path: String { get }

    /// The HTTPMethod for the request
    var method: HTTPMethod { get }

    /// Optional parameters for the request
    var parameters: [String : Any]? { get }

    /// The HTTP headers to be sent
    var httpHeaderFields: [String : String]? { get }
}

public enum HTTPMethod: String {
    case GET, POST, PUT, DELETE, OPTIONS, HEAD, PATCH, TRACE, CONNECT
}

extension Endpoint {
    public var method: HTTPMethod {
        return .GET
    }

    public var parameters: [String : Any]? {
        return nil
    }

    public var httpHeaderFields: [String : String]? {
        return nil
    }
}

public protocol Environment {
    var baseURL: URL { get }
    var shouldAllowInsecureConnections: Bool { get }
}

public extension Environment {
    var shouldAllowInsecureConnections: Bool {
        return false
    }
}
```

- Add tests

```
class HTTPBINAPITests: XCTestCase {
    func testIPEndpoint() {
        let getIP = HTTPBINAPI.ip
        XCTAssert(getIP.path == "/ip")
        XCTAssert(getIP.method == .GET)
    }
}

```

- Start writing Unit Tests against them using HTTPBin as an example

```swift
private enum HTTPBINHosts: Environment {
    case production
    case development

    fileprivate var baseURL: URL {
        switch self {
        case .production:
            return URL(string: "https://httpbin.org")!
        case .development:
            return URL(string: "https://dev.httpbin.org")!
        }
    }
}

private enum HTTPBINAPI: Endpoint {
    case ip
    case orderPizza

    var path: String {
        switch self {
        case .orderPizza:
            return "/forms/post"
        case .ip:
            return "/ip"
        }
    }

    var method: HTTPMethod {
        switch self {
        case .orderPizza:
            return .POST
        default:
            return .GET
        }
    }
}
```

#APIClient

- Create APIClient to wire both of them (1c06f2a)

```swift
public class APIClient {

    let environment: Environment

    public init(environment: Environment) {
        self.environment = environment
    }

    public func performRequest(forEndpoint endpoint: Endpoint) {

    }
}

class APIClientTests: XCTestCase {

    var apiClient: APIClient!

    override func setUp() {
        super.setUp()
        apiClient = APIClient(environment: HTTPBINHosts.production)
    }

    func testGET() {
        apiClient.performRequest(forEndpoint: HTTPBINAPI.ip)
    }
}
```

- Integrate with `URLSession` with `createURLRequest(endpoint: Endpoint) throws -> URLRequest`  (5183d01)

```swift
public extension URLRequest {
	init(fromEndpoint endpoint: Endpoint, andEnvironment environment: Environment) throws {
	    guard let url = URL(string: endpoint.path, relativeTo: environment.baseURL) else {
	        throw APIClient.Error.malformedURL
	    }
	        
	    self.init(url: url)
	        
	    httpMethod = endpoint.method.rawValue
	    allHTTPHeaderFields = endpoint.httpHeaderFields
	    //Not included in repo
	    setValue("User-Agent", forHTTPHeaderField: "GifWallet - iOS")
	    
	    if let parameters = endpoint.parameters {
	        do {
	            let requestData = try JSONSerialization.data(withJSONObject: parameters, options: .prettyPrinted)
	            httpBody = requestData
	            setValue("application/json", forHTTPHeaderField: "Content-Type")
	        } catch {
	            throw APIClient.Error.malformedParameters
	        }
        }
    }
}

// in APIClient
enum Error: Swift.Error {
    case malformedURL
    case malformedParameters
    case malformedResponse
}
```


```swift
public func performRequest(forEndpoint endpoint: Endpoint, handler: @escaping (Data?, Swift.Error?) -> ()) {
    let urlRequest: URLRequest
    do {
        urlRequest = try URLRequest(fromEndpoint: endpoint, andEnvironment: environment)
    } catch let error {
        handler(nil, error)
        return
    }
    
    let task = urlSession.dataTask(with: urlRequest) { (data, _, error) in
        guard error == nil else {
            handler(nil, error)
            return
        }
        
        guard let data = data else {
            handler(nil, Error.unknown)
            return
        }
        
        handler(data, nil)
    }
    task.resume()
}
```

#Parsing

- Refactor HTTPBin a bit (`da4d9e9`)
	- Create an empty enum and put both Endpoint and Environment there.
- Create the Response
- Create the generic parser explaining how a generic func is created

```swift
func parseResponse<T: Decodable>(data: Data) throws -> T {
    do {
        return try jsonDecoder.decode(T.self, from: data)
    } catch {
        throw Error.malformedJSONResponse
    }
}
```

- Parse the Response using a Test `6aef436`

```swift
    func testParseIPResponse() throws {
        let json =
            """
{
  "origin": "80.34.92.76"
}
"""
                .data(using: .utf8)!
        guard let response: HTTPBin.Responses.IP = try? apiClient.parseResponse(data: json) else {
            XCTFail("Response threw error")
            return
        }
        XCTAssert(response.origin == "80.34.92.76")
    }
```
 
- Add response parsing in APIClient.
	- Add JSONDecoder() and attempt to parse the response.
	- See that it fails, explain that there is a bug in the User-Agent, fix it by:
	- Adding code that checks if the response is 200
	- Data is not nil and bytes.count > 0
	- Make sure test passes.

#Refactor APIClient

- Subclass APIClient into HTTPBinAPIClient and tie the Endpoint and the response type in a fetchIPAddress method. `26cc540`

```swift
class HTTPBinAPIClient: APIClient {
    
    func fetchIPAddress(handler: @escaping(HTTPBin.Responses.IP?, Swift.Error?) -> Void) {
        self.performRequest(forEndpoint: HTTPBin.API.ip) { (data, error) in
            guard error == nil else {
                handler(nil, error!)
                return
            }
            guard
                let _data = data,
                let response: HTTPBin.Responses.IP = try? self.parseResponse(data: _data) else {
                    handler(nil, Error.malformedJSONResponse)
                    return
            }
            
            handler(response, nil)
        }
    }
}
```

#Refactor Parsing

- Further clean it by creating `performRequestAndParseResponse` (8e51b01)

```swift
public func performRequestAndParseResponse<T: Decodable>(forEndpoint endpoint: Endpoint, handler: @escaping (T?, Swift.Error?) -> Void) {
    self.performRequest(forEndpoint: endpoint) { (data, error) in
        guard error == nil else {
            self.delegateQueue.async { handler(nil, error!) }
            return
        }
        guard let data = data else {
            self.delegateQueue.async { handler(nil, Error.malformedResponse) }
            return
        }

        let response: T
        do {
            response = try self.parseResponse(data: data)
        } catch let error {
            self.delegateQueue.async { handler(nil, error) }
            return
        }

        handler(response, nil)
    }
}
```

then fetchIPAddress turns out to be just: 

```swift
func fetchIPAddress(handler: @escaping(HTTPBin.Responses.IP?, Swift.Error?) -> Void) {
    self.performRequestAndParseResponse(forEndpoint: HTTPBin.API.ip) { (ip: HTTPBin.Responses.IP?, error) in
        handler(ip, error)
    }
}
```

- Explain how `Endpoint` can't include `associatedType` without making it unusable. Explain difference with `ViewModelConfigurable`.
![](https://i.imgur.com/AdX085S.png)
- Introduce the `Request` type to even more clean interface (`eecc5ca`)

```swift
public struct Request<ResponseType>{
    let endpoint: Endpoint
}
```

- further enhance Request taking the environment into it.

```swift
public struct Request<ResponseType> {
    let endpoint: Endpoint
    let environment: Environment
}
```

- show how performRequestAnd... turns into simply perform<T>
- show that a lot of duplicated code can be taken out in APIClient.perform()
- refactor the `URLRequest.init<T>(fromRequest: Request<T>)`

#GIPHY API

- Now, to the Giphy API (students)
	- Implement the search method
	- Implement the trening method
	- Finished at `01e64b3`

- Go back to slides