# Token Authentication with Vapor ![][image-1]

[back to index][1]

For limiting access to your sensitive API endpoints you can use **AccessToken** authentication

## Principle

You log in on a `/login` endpoint first using [Basic Authentication][2] and this endpoint returns you an access token that you can use with other endpoints using simple HTTP Headers like this:

```html
Authorization: Bearer XXXXXX-TOKEN-XXXXXXX
```

## How to implement using Vapor framework

### Setup

You need to add the `Authentication` package in your SPM config (`github: vapor/auth`)

```swift
.package(url: "https://github.com/vapor/auth.git",
		 from: "2.0.0-rc")
```

```swift
dependencies: [...,
			   "Authentication"]
```

### Configuration

In your `configure.swift` file, add the following line:

```swift
try services.register(AuthenticationProvider())
```

And in any file where you add some auth code, don't forget to import the module:

```swift
import Authentication
```

### Model
You need to create a `Token` model first:

```swift
final class Token: Content {
  var token: String
  var userID: User.ID

  init(token: String, userID: User.ID) {
	self.token = token
	self.userID = userID
  }
}
```

### Token creation
To create the token, you can use the `CryptoRandom` API like this:

```swift
extension Token {
  static func generate(for user: User) throws -> Token {
	let random = try CryptoRandom().generateData(count: 16)
	return try Token(
	  token: random.base64EncodedString(),
	  userID: user.requireID())
  }
}
```

This code will generate a `Token` object with a 16 bytes token. You'll use this method to create a token to return from your `/login` endpoint.

### Model Conformance
To use your `Token` model for token authentication, you need to conforms to the `Authentication.Token` and `BearerAuthenticatable` protocols:

```swift
extension Token: Authentication.Token {
  static let userIDKey: UserIDKey = \Token.userID
  typealias UserType = User
}

extension Token: BearerAuthenticatable {
  static let tokenKey: TokenKey = \Token.token
}
```

The first one will setup the correspondance between your `User` model and the `userID` properties of your `Token` model.

The second will tell the middleware which property of your `Token` model is storing the access token.

### Routing
In your routing code, you need to create two middlewares that will be responsible for intercepting each request and check if it is correctly authorized:

```swift
let tokenAuthMiddleware =
  User.tokenAuthMiddleware()
let guardAuthMiddleware = User.guardAuthMiddleware()
```

Then you need to create a group of routes managed by those two middlewares:

```swift
let protectedRoutes = router.grouped(
  tokenAuthMiddleware,
  guardAuthMiddleware)
// you can now use protectedRoutes to manage your routes that require authentication with token Auth
protectedRoutes.post("api/endpoint", use: apiEndpointHandler)
```

### Retrieving an authenticated user
In your authenticated routes, when you need to retrieve the user associated with the given token, you can simply add the following code:

```swift
let user = try request.requireAuthenticated(User.self)
```

## References

* [Server Side Swift with Vapor" ebook, section III, chapter 18)][3]

[back to index][4]

[1]:	../README.md
[2]:	basic_authentication.md
[3]:	https://store.raywenderlich.com/products/server-side-swift-with-vapor
[4]:	../README.md

[image-1]:	img/vapor3_20.jpg "compatible with Vapor 3"