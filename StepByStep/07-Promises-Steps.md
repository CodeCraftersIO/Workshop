Start at (3bc527f)

- Add Async as our Promises Library
`pod 'BNRDeferred', '4.0.0-beta.2'"`

- Create async versions of these functions:

``` 
    func parseResponse<T: Decodable>(data: Data, handler: @escaping (T?, Swift.Error?) -> Void) {
        do {
            let parsedResponse: T = try parseResponse(data: data)
            handler(parsedResponse, nil)
        } catch let error {
            handler(nil, error)
        }
    }
```


``` 
public extension URLRequest {
    static func createURLRequest<T>(request: Request<T>, handler: @escaping (URLRequest?, Swift.Error?) -> Void) {
        do {
            let request = try URLRequest.init(fromRequest: request)
            handler(request, nil)
        } catch let error {
            handler(nil, error)
        }
    }
}
```

- Try to make it work. Show the chaos.
- Start implementing promises
- Wrap every handler as promise in WalletKit
	- `b780dc0` 
- Make them migrate the Unit Tests to Task<T>