Start at db9d480:

- Create Environment, Endpoint
- Start writing Unit Tests against them using HTTPBin as an example
- Create APIClient to wire both of them (1c06f2a)
- Integrate with `URLSession` with `createURLRequest(endpoint: Endpoint) throws -> URLRequest`  (0646a74)
- Refactor HTTPBin a bit (2178cb2)
- Start the parsing section. Explain how `Endpoint` can't include `associatedType` without making it unusable. Explain difference with `ViewModelConfigurable`.
- Subclass APIClient into HTTPBinAPIClient and tie the Endpoint and the response type in a fetchIPAddress method.
- Add JSONDecoder() and attempt to parse the response. See that it fails
- Add a unit tests to verify that
- See that the data is 0, go back to `APIClient.performRequestForEndpoint` and see that the response is returning a 400
- Fix the "User-Agent" and see that it starts working.
- Clean it up by adding a generic func to APIClient to parse the responses. Explain how creating a Parser for every response makes no sense (ae8ac1a)
- Further clean it by creating `performRequestAndParseResponse` (8e51b01)
- Introduce the `Request` type to even more clean interface (3956246)
- Now, to the Giphy API
	- Implement the search method 