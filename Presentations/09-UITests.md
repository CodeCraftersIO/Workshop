theme: Work, 1


#UITests

---

# Tests

During this workshop, we've built:

- Unit Tests
- Snapshot Tests

--- 

#However...

![inline](https://i.imgur.com/KBw4SIO.png)

---

![fit](https://i.imgur.com/Pw5IZDw.png)

---

# This is what UITests are for

---

#User Interface Testing

- UI testing gives you the ability to find and interact with the UI of your app in order to validate the properties and state of the UI elements.

- Built with `XCTest` and `Accessibility`

---

![fit](https://martinfowler.com/bliki/images/testPyramid/test-pyramid.png)

---

#APIs

UI tests are based on the implementation of three new classes:

- `XCUIApplication`
- `XCUIElement`
- `XCUIElementQuery`

---

#APIs

Best way to start it via recording

---

#Demo

---

#Takeaways

- UITests are executed on a different executable, not on your app.
	- Don't have access to your app's code.
	- Test preparation is done differently
- Elements are accessed via Accesibility queries
	- Slow 🐌. 

---

#Takeaways

- Ideal to run nightly or on every PR.
	- Definetly not for every build.
	- Run them on a different scheme.
- You'll get accessibility for free.
- You'll improve your coverage.

---

![fit](https://i.imgur.com/nT7ZjRm.png)

---

#Takeaways

- Environment variables can be passed using `launchArguments`

```swift
let app = XCUIApplication()
app.launchArguments.append("UITests")
app.launch()
...
if ProcessInfo.processInfo.arguments.contains("UITests") {
}

```
---

#Takeaways

- Use this to create mock Interactors instead

```swift
let presenter: GIFWalletPresenterType = {
	if ProcessInfo.processInfo.arguments.contains("UITests") {
		return GIFWalletViewController.MockDataPresenter()
            } else {
		return GIFWalletViewController.Presenter(dataStore: dataStore)
	}
}()
```
