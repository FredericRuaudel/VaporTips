# Converting Optional Future into Future of Optional

[back to index][1]

## The problem

Let say you have a `User` model that can have a pet or not.
You'll create a model like this with an optional parent relationship toward your `Pet` model:
```swift
struct User : PostgreSQLModel {
	var id: Int?
	var name: String
	var petId: Pet.ID?
	var pet: Parent<User, Pet>? {
		return parent(\User.petId)
	}
}

struct Pet: PostgreSQLModel {
	var id: Int?
	var name: String
}
```

Now what happens when you want to get the pet of a user?  
You'll probably like to write a method like below that should return a `Future` of optional `Pet`:

```swift

func getPet(for user: User, on connection: DatabaseConnectable) -> Future<Pet?> {
	return user.pet.get(on: connection) // Future<Pet>? 💥🤪
}

```

Unfortunately, this code will return an optional `Future` of `Pet` 🙈

So how can we solve that ?! 🤔

## The solution
To solve this problem, we need a little bit of optional juggling.
We need to solve both cases of our `Optional`:
- When we have a valid `Future`, we want to return its content embedded as an optional
- When we have nothing, we want to create a new `Future` that contains `nil`

Do to this, we can create this little extension on `Optional`:

```swift
extension Optional {
	func asFuture<T>(on worker: Worker) -> Future<T?> where Wrapped == EventLoopFuture<T> {
		switch(self) {
		case let .some(future):
			return future.map { .some($0) }
		case .none:
			return Future<T?>.map(on: worker, { nil })
		}
	}
}
```

Then, you can fix your method above like this:

```swift
func getPet(for user: User, on connection: DatabaseConnectable) -> Future<Pet?> {
	return user.pet // get the optional parent relationship
		.map({ // unwrap the optional relationship
			$0.get(on: connection) // return a Future<Pet> that will be wrapped into a Future<Pet>?
		}).asFuture(on: connection) // convert the resulting Future<Pet>? into Future<Pet?>
}
```

That's all folks! 🎉

[back to index][2]

[1]:	../README.md
[2]:	../README.md