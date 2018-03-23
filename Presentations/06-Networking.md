#Networking

---

![fit](https://i.imgur.com/nvLwbWi.png)

---

![fit](https://i.imgur.com/Q1n6QtN.png)

---

```objc
ConfigurationLogic *logic = [ConfigurationLogic currentConfiguration];

[NetworkFetcher  callService:@"/users"
					    host:[logic baseURL]
					  method:GET
					postData:nil
				  bodyInJSON:NO
					hasCache:YES
			 timeoutInterval:[logic servicesTimeout]
		      successHandler:successHandler
			  failureHandler:failureHandler];
```
---

```swift
func lookStockFor(symbol: String, handler: @escaping Handler) {
	let parameters = ["input": symbol]
	let request = networkFetcher.request(
		.GET,
		"http://dev.markitondemand.com/api/v2/lookup.json"
		parameters: parameters,
		encoding: .URL
	)
	
	request.responseJSON { result in
		guard result.error == nil {
			handler(nil, result.error)
			return
		}
		let symbols: [StockSymbols] = JSONParse(result.json)
		handler(symbols, nil)
	
	}
}
```
---

#Objectives:

1. Composability
2. Sensitive Default Values
3. Readability
4. Self Documented

---

![right](https://media.giphy.com/media/12NUbkX6p4xOO4/giphy.gif)

# More protocols

---

```swift
public protocol Endpoint {

    /// The path for the request
    var path: String { get }

    /// The HTTPMethod for the request
    var method: HTTPMethod { get }

    /// Optional parameters for the request
    var parameters: [String : Any]? { get }

    /// How the parameters should be encoded
    var parameterEncoding: HTTPParameterEncoding { get }

    /// The HTTP headers to be sent
    var httpHeaderFields: [String : String]? { get }
}
```
---

```swift
//  This is the default implementation for Endpoint 
extension Endpoint {
    public var method: HTTPMethod {
        return .GET
    }
    
    public var parameters: [String : AnyObject]? {
        return nil
    }
    
    public var parameterEncoding: HTTPParameterEncoding {
        return .url
    }
    
    public var httpHeaderFields: [String : String]? {
        return nil
    }
}
```
---

#To Xcode!

![left](https://media.giphy.com/media/5GoVLqeAOo6PK/giphy.gif)

---
#What we've built:

---