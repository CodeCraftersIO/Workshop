theme: Fira, 3

![](http://codecrafters.io/static/logocc.b146854e.png)

---

## Build **Better**
## Apps **Faster**

---
## **What we'll cover**

- Size classes & Autolayout. ☠️
- Automatic Cell Sizing 📐
- Stateful ViewControllers. ⏳
- Networking and Promises. 🎁
- Forms 📖
- CoreData ⚙️
- Soft Skills 👔

---

## And of course...
## **Tests**

---

##We'll build:

---

##A Wallet

---

##A Crypto Wallet?

---

##A ~~Crypto~~ Wallet

---

##A  **GIF** Wallet

---

![left](https://youtu.be/tCjinXH4QT8)

# GIf Wallet

- Store GIFs in a handy way to share
- Integration with Giphy to find new GIFs
- iPad support

---

###Schedule:

|   **Fri 9**  |    **Sat 10**   |   **Fri 16** |    **Sat 17**	|
| :-----------: | :-----------: | :-----------: | :-----------: |
| Autolayout        | Stateful VC |    Networking    | CoreData    |
| Size Classes 		| Forms 		 |   Promises   | UI Tests   |
| Self-Sizing Cells |  👨‍💻			 |    👩🏽‍💻 | Soft Skills   |


---

#...But first

---

## [Bootstrapping](https://www.youtube.com/playlist?list=PL3Hgz5X2fCsFI72e8OcAnNJYRf71vBtub)

![](https://www.surepayroll.com/globalassets/blog-article-images/bootstrapping-benefits-vs-venture-capital.jpg)

---

#Goals of Boostrapping:

- Separation of concerns
- Dependency Management
- Schemes for Development and AppStore
- Tests setup
- Continuous Integration

![](https://www.surepayroll.com/globalassets/blog-article-images/bootstrapping-benefits-vs-venture-capital.jpg)

---

![inline](https://i.imgur.com/sTcBJgh.png)

---

![inline](https://i.imgur.com/mX01MVT.jpg)


---

##Separation of concerns

UI and Model layers should be separated into modules.

- Easier testing
- Faster compilation
- Reusability

---

`UIKit` → UI Module
`Foundation` → Foundation Module

---

##What's a module?

---
A Module[^1]:

- Encapsulates code and data.
- Has an interface (API)
- Easily deployed

[^1]: Modular programming from [Wikipedia](https://en.wikipedia.org/wiki/Modular_programming).

---

`UIKit` → UI Module → App
`Foundation` → Foundation Module → Framework

---

##However...

---

![inline](https://i.imgur.com/1gTA9p3.png)

---

![inline](https://i.imgur.com/uMkcolO.png)

---

In this cases:

`UIKit` → UI Module → Framework
`Foundation` → Foundation Module → Framework

---

![left fit](https://i.imgur.com/DXENrh8.png)

## App Target
- Cells and Views
- ViewController
- Interactors

## Framework Target
 
- Form validation
- Data store
- APIClient

---

##Dependency Management

---

##Dependency Management

The de-facto standard is Cocoapods

However, Carthage is also a challenger.

---

![inline](https://i.imgur.com/7fC04CZ.png)

---

##Tradeoffs:

- Add `/Pods` to repo.
- Xcode integration.
- Too many frameworks.

---

##Schemes for Development and AppStore

---

![inline](https://i.imgur.com/2ayPd87.png)

---

## We shouldn't use the two target approach

- It 2x `xcodeproj` file's size.
- It leads to errors when adding new files.
- It feels dirty.

---

## Instead, we should use schemes
- With schemes we can get the same result.
- We can parametrize what we need using `xcconfig`
- It's predictable.

---

#XCConfig

![left](https://i.imgur.com/STnHZkM.png)

- Allows us to parametrize: Bundle ID, Entitlements, Compilation Flags...

- Can be set per scheme.

- Integrates with Cocoapods.

---

![left fit](https://i.imgur.com/mEuorPk.png)
![right fit](https://i.imgur.com/23um8jn.png)

---

![fit](https://i.imgur.com/SLohiYw.png)

---

#Tests setup

---

#Tests setup

- Tests should not launch your app's logic.
- Integrate with Snapshot testing.
- Separate App Tests from Model tests

---
```swift
@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate, UISplitViewControllerDelegate {

    var window: UIWindow?

    func application(
    _ application: UIApplication,
     didFinishLaunchingWithOptions options: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {

        self.window = UIWindow(frame: UIScreen.main.bounds)
        defer { self.window?.makeKeyAndVisible() }

        guard NSClassFromString("XCTest") == nil else {
            self.window?.rootViewController = UIViewController()
            return true
        }

        self.window?.rootViewController = ...
        return true
    }
}

```

---

#Continous integration

---

#Continous integration

Several options:
- Buddybuild
- Bitrise
- CircleCI
- Travis

---

#Why Bitrise?

---

#Why Bitrise?
- **Multiplatform**
- Easy setup
- Github integration for PRs
- Apple Developer integration
- Price scales linearly

---

![fit](https://i.imgur.com/kxpVMA0.png)

---

![fit](https://i.imgur.com/Cay3j8i.png)

---

![fit](https://i.imgur.com/j0yzj7F.png)

---

# Architecture

- This is just a fancy word to explain how you manage complexity regarding:
- Presenting and dismissing `UIViewController` instances.
- Passing data.

---

#ViewController Data

- MVVM allows us to quickly mock data required by a ViewController to display it's contents.

---

[.background-color: #FFFFFF]

![left fit](https://i.imgur.com/Ls5nQb0.png)
![right fit](https://i.imgur.com/KU66U5Y.jpg)

---

#Data Retrieval

- A type called `Interactor` will create mock/real VM and pass it to the VC.
- Interactors will be injected into the ViewControllers in the `init`

---

```swift
class GIFWalletViewController: UIViewController {

    let interactor: GIFWalletInteractorType
    
    init(interactor: GIFWalletInteractorType) {
        self.interactor = interactor
        super.init(nibName: nil, bundle: nil)
    }
}

protocol GIFWalletInteractorType {
    func fetchData(handler: @escaping WalletDataHandler)
}
```
---

#ViewController Creation

- It'll also be handled by `Interactor`.
- However, in a complex app this would not scale correctly, and this class should be split into two: `Repository` and `Wireframe`.

---

#Let's go!

---

![fit](https://i.imgur.com/zbEmgkb.png)