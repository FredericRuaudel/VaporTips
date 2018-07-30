# How to easily maintain your Linux tests

[back to index][1]

## The problem
When testing on Linux platforms, the XCTest framework doesn't have access to the Objective-C runtime to analyse your test suites code and find all the test methods. 
So you'll usually find in guides about testing server side code on Linux, that you have to maintain a bunch of `allTests` static vars in all your test suite classes alongside a file named `LinuxMain.swift` that must gather them all in a `XCTMain` call like this:

```swift
// Tests/AppTests/AppTests.swift
import App
import XCTest

final class AppTests: XCTestCase {
	func testNothing() throws {
		// add your tests here
		XCTAssert(true)
	}

	static let allTests = [
		("testNothing", testNothing),
	]
}


// Tests/LinuxMain.swift
import XCTest
@testable import AppTests

XCTMain([
	testCase(AppTests.allTests),
])
```

The problem is that you need to maintain this list in each of your test suites' `allTests` vars and the list of these `allTests` vars in the `LinuxMain.swift` file. That makes a lots of reasons to fail sooner or later 🙈.

## The Solution

Since **Swift 4.1**, there is a waaaay better solution to deal with this!

### SPM to the rescue
Now, we can use a new **Swift Package Manager** command:

```bash
swift test --generate-linuxmain
```

This command will generate two things:

First, in each of your subfolders inside your `Tests` folder, it will create a `XCTestManifests.swift` holding a `__allTests` static var with all your tests listed inside.

Second, it will generate your `LinuxMain.swift` referencing all these `__allTests` vars.

### Caveats
This SPM command **still requires the Objective-C runtime** so you need to _run it on a Mac_ before running your tests on Linux. So this is not a perfect answer for those who develop directly on a Linux platform. It is more targeted to those who develop on Mac and run their Linux tests using a Docker image.

Another caveat is that **this command doesn't update** existing `LinuxMain.swift` and `XCTestManifests.swift` files, so you need to remove them all, and then running the command again if you need to update your tests.

### Automating the update process
To **solve the test update caveat**, we can easily create a script that will do the update for us. We can also add in this script the docker command we use to run our tests on Linux so we can be sure that all our tests are up to date before running them:

```bash
#!/bin/bash

# Check if we are on a Mac first
if [ `uname` = "Darwin" ]
then
	# Remove all generated files
	find ./Tests -name XCTestManifests.swift -exec rm {} \;
	find ./Tests -name LinuxMain.swift -exec rm {} \;
	# Re-generate them
	swift test --generate-linuxmain
else
	echo
	echo "Running on Linux plateform: can't update test suite!"
	echo
fi

# Put here your docker command that will run your tests on Linux
docker build --tag "linux_tests" . && docker run --rm "linux_tests"
```

**⚠️ Caution:** This script will first remove all `XCTestManifests.swift` and `LinuxMain.swift` files that are present in the `Tests` folder and subfolders from the path you run this script. Be sure to never edit them manually!

## References
- [ Keeping XCTest in sync on Linux by Ole Begemann ][2]

[back to index][3]

[1]:	../README.md
[2]:	https://oleb.net/blog/2017/03/keeping-xctest-in-sync/
[3]:	../README.md