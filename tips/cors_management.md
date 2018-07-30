# CORS Management ![][image-1]

[back to index][1]

To allow a React (or similiar) frontend to access the backend API, you need to setup a CORS middleware in your vapor `configure.swift` file:

```swift
let corsMiddleware = CORSMiddleware(
	configuration: CORSMiddleware.Configuration.init(
		allowedOrigin: .all,
		allowedMethods: [
			.POST,
			.GET,
			.PUT,
			.OPTIONS,
			.DELETE,
			.PATCH
		],
		allowedHeaders: [
			.accept, 
			.authorization, 
			.contentType, 
			.origin, 
			.xRequestedWith
		]))
middlewares.use(corsMiddleware)
```

**Note:** This is the default parameters expanded so you can see them without digging into the code. Maybe you'll need to adjust the parameters. That said, it has been tested and this example allows a default React website to access the vapor API routes.

## References
- [CORSMiddleware.swift on Github][2]

[back to index][3]

[1]:	../README.md
[2]:	https://github.com/vapor/vapor/blob/master/Sources/Vapor/Middleware/CORSMiddleware.swift
[3]:	../README.md

[image-1]:	img/vapor3_20.jpg "compatible with Vapor 3"