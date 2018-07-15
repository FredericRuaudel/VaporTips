# CORS Management ![][image-1]

To allow React frontend to access the backend API, we need to add CORS middleware in the vapor `configure.swift` file:

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

**Note:** This is a copy/paste example found on the web. Maybe we'll need to adjust the parameters. That said, it has been tested and this example allows a React website to access the vapor API routes.

[image-1]:	img/vapor3_20.jpg