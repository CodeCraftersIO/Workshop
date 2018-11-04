Start at db9d480:

#Environment and Endpoint

- Create Environment, Endpoint

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

- Integrate with `URLSession` with `createURLRequest(endpoint: Endpoint) throws -> URLRequest`  (0646a74)

```swift
    private func createURLRequest(endpoint: Endpoint) throws -> URLRequest {
        guard let URL = URL(string: endpoint.path, relativeTo: self.environment.baseURL) else {
            throw Error.malformedURL
        }

        var urlRequest = URLRequest(url: URL)
        urlRequest.httpMethod = endpoint.method.rawValue
        urlRequest.allHTTPHeaderFields = endpoint.httpHeaderFields
        urlRequest.setValue("User-Agent", forHTTPHeaderField: "GifWallet - iOS")
        if let parameters = endpoint.parameters {
            do {
                let requestData = try JSONSerialization.data(withJSONObject: parameters, options: .prettyPrinted)
                urlRequest.httpBody = requestData
                urlRequest.setValue("application/json", forHTTPHeaderField: "Content-Type")
            } catch {
                throw Error.malformedParameters
            }
        }
        return urlRequest
    }

    enum Error: Swift.Error {
        case malformedURL
        case malformedParameters
        case malformedResponse
    }
```

#Refactor APIClient

- Refactor HTTPBin a bit (2178cb2)
	- Create an empty enum and put both Endpoint and Environment there.
- Create the Response
- Subclass APIClient into HTTPBinAPIClient and tie the Endpoint and the response type in a fetchIPAddress method.

```swift
class HTTPBinAPIClient: APIClient {

    func fetchIPAddress(handler: @escaping(HTTPBin.Responses.IP?, Swift.Error?) -> Void) {
        self.performRequest(forEndpoint: HTTPBin.API.ip) { (data, error) in

}
```

#Parsing

- Parse the Response using a Test

```swift
    func testParseIPResponse() throws {
        let json =
"""
{
  "origin": "80.34.92.76"
}
"""
            .data(using: .utf8)!
        let decoder = JSONDecoder()
        let response = try decoder.decode(HTTPBin.Responses.IP.self, from: json)
        XCTAssert(response.origin == "80.34.92.76")
    }
```
 
- Add response parsing in HTTPBinAPIClient.
	- Add JSONDecoder() and attempt to parse the response.
	- See that it fails, explain that there is a bug in the User-Agent, fix it by:
	- Adding code that checks if the response is 200
	- Data is not nil and bytes.count > 0
	- Make sure test passes.

#Refactor Parsing

- Clean it up by adding a generic func to APIClient to parse the responses. Explain how creating a Parser for every response makes no sense (ae8ac1a)

```swift
    public func parseResponse<T: Decodable>(data: Data) throws -> T {
        do {
            return try self.jsonDecoder.decode(T.self, from: data)
        }
        catch {
            throw Error.malformedJSONResponse
        }
    }
```

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
- Explain how `Endpoint` can't include `associatedType` without making it unusable. Explain difference with `ViewModelConfigurable`.
![](https://i.imgur.com/AdX085S.png)
- Introduce the `Request` type to even more clean interface (3956246)

```swift
public struct Request<ResponseType>{
    let endpoint: Endpoint
}

```

#GIPHY API

- Now, to the Giphy API (students)
	- Implement the search method
	- Implement the trening method
	- Finished at f492df8

- Go back to slides