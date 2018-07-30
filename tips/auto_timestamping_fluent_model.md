# How to auto-Timestamp your fluent models ![][image-1]

It is common to add timestamps in your database table schema like `createdAt` and `updatedAt` to track modifications in your data. But dealing with it each time you need to create a model or updating it, could be cumbersome.

Using **Vapor 3** and **Fluent**, this is very easy to add those fields and ask **Fluent** to automatically manage them for you! 🎉

## Setup your model
Let's say you have a `Person` model like this:

```swift
struct Person: PostgreSQLModel {
	var id: Int?
	var firstName: String
	var lastName: String
}
```

Here is all you have to do to auto-timestamp it:

- First add the two timestamp properties:

```swift
struct Person: PostgreSQLModel {
	var id: Int?
	var firstName: String
	var lastName: String
	// Timestamp
	var createdAt: Date?
	var updatedAt: Date?
}
```

**Note:** you can name them differently if you prefer.

- Second, tell **Fluent** which one they are, using `keyPath` definition like below

```swift
extension Person {
	static let createdAtKey: TimestampKey? = \Person.createdAt
	static let updatedAtKey: TimestampKey? = \Person.updatedAt
}
```

## How to make it easier to manage?
If you start to have lots of models, it can quickly become tedious to add all these `keyPath` definitions which are always the same. And as you usually want to use the same names for these properties through out all your schemas, you can easily define a new protocol to ease the process.

So, let's introduce this new protocol that we can call `AutoTimestampable`! 🤓

```swift
import Fluent

protocol AutoTimestampable: Model {    
	var createdAt: Date? { get set }
	var updatedAt: Date? { get set }
}

extension AutoTimestampable {
	static var createdAtKey: TimestampKey? { return \Self.createdAt }
	static var updatedAtKey: TimestampKey? { return \Self.updatedAt }
}
```

**Note:** If you prefer, replace the timestamp properties in the protocol definition and in the returned value of each default `keyPath` implementations. For example, if you prefer naming them `creationDate` and `lastUpdateDate`, you'll get this instead:

```swift
import Fluent

protocol AutoTimestampable: Model {    
	var creationDate: Date? { get set }
	var lastUpdateDate: Date? { get set }
}

extension AutoTimestampable {
	static var createdAtKey: TimestampKey? { return \Self.creationDate }
	static var updatedAtKey: TimestampKey? { return \Self.lastUpdateDate }
}
```

**Tips:** Once you've added this, you can leverage **Xcode** fix-it to help you adding the missing pieces in each of your models. Simply add the `AutoTimestampable` protocol at the definition level (not in an extension) to have **Xcode** automatically adding the timestamp properties for you:

![][image-2]

Once you see the error, click on it and hit the `Fix` button, or simply hit `Ctrl+Opt+Cmd+F`:

![][image-3]

**Note:** if you prefer adding it in an extension, the fixit will create computed properties stubs instead since you can't add stored properties in extensions, which is probably not what you're expecting 🙈. 

![][image-4]

If you prefer this style of protocol conformance, you'll have to create the properties yourself in the main definition.

[image-1]:	img/vapor3_20.jpg "compatible with Vapor 3"
[image-2]:	img/Autotimestampable-fixit-error.png
[image-3]:	img/Autotimestampable-fixit-done.png
[image-4]:	img/Autotimestampable-fixit-extension.png