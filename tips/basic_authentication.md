# Basic Authentication with Vapor ![][image-1]

[back to index][1]

To manage login, you can use the Basic Authentication protocol (RFC 7617) which is supported by Vapor 3 by default

## Principle

you have a username and a password that you combined like so:

```html
 admin:password
```

then, you base64-encode them:

```html
 YWRtaW46cGFzc3dvcmQK==
```

and you submit it in your HTTP Header like so:

```html
 Authorization: Basic YWRtaW46cGFzc3dvcmQK==
```

## How to implement using Vapor framework

### Setup
 
First, you need to add the `Authentication` package in your SPM config (`github: vapor/auth`)

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

### Model Conformance

Assuming you have a `User` model object with at least two properties `username` and `password`, you need to make this model conforms to `BasicAuthenticatable` protocol by defining the keypath of the two properties:

```swift
extension User: BasicAuthenticatable {
  static let usernameKey: UsernameKey = \User.username
  static let passwordKey: PasswordKey = \User.password
}
```

### Routing

In your routing code, you need to create two middlewares that will be responsible for intercepting each request and check if it is correctly authorized:

```swift
let basicAuthMiddleware =
  User.basicAuthMiddleware(using: BCryptDigest())
let guardAuthMiddleware = User.guardAuthMiddleware()
```

*Note:* the basic auth middleware will using **BCrypt** hashing to verify the password.
These lines can be added directly next to your routes definitions.

Then you need to create a group of routes managed by those two middlewares:

```swift
let protectedRoutes = router.grouped(
  basicAuthMiddleware,
  guardAuthMiddleware)
// you can now use protectedRoutes to manage your routes that require authentication with basic Auth
protectedRoutes.post("login", use: loginHandler)
```

### User creation

When creating your user, you need to hash your password. You can simple use the standard **BCrypt** hashing like that:

```swift
import Crypto

let user = getUser(...)
user.password = try BCrypt.hash(user.password)
user.save(on: dbconnection)
```

## References

* [Server Side Swift with Vapor" ebook, section III, chapter 18)][2]

[back to index][3]

[1]:	../README.md
[2]:	https://store.raywenderlich.com/products/server-side-swift-with-vapor
[3]:	../README.md

[image-1]:	img/vapor3_20.jpg "compatible with Vapor 3"