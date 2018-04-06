Start at (3bc527f)

- Add Async as our Promises Library
`pod 'Async', :git => "git@github.com:CodeCraftersIO/async.git"`

- Create `func createURLRequest(endpoint: Endpoint, handler: @escaping (URLRequest?, Swift.Error?) -> Void)`
- Create `parseResponse<T: Decodable>(data: Data, handler: @escaping (T?, Swift.Error?) -> Void)`
- Try to make it work. Show the chaos
- Start implementing promises
- Wrap every handler as promise in WalletKit