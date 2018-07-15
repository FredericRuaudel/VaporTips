# Basic Authentication with Vapor ![][image-1]

For the login, we can use the Basic Authentication protocol (RFC 7617)

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

_(for full details, you can refer to "Server Side Swift with Vapor" ebook, section III, chapter 18)_

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

*Note:* the basic auth middleware will using **BCrypt** hashing to verify the password

Then you need to create a group of routes managed by those two middlewares:

```swift
let protectedRoutes = router.grouped(
  basicAuthMiddleware,
  guardAuthMiddleware)
// you can now use protectedRoutes to manage protected routes with basic Auth
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


[image-1]:	img/vapor3_20.jpg