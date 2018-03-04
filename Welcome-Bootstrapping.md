![](https://i.imgur.com/bMN1pjb.png)

---

## Build **better**
## apps **faster**

---
## **What we'll cover**

- Stateful ViewControllers. ⏳
- Size classes & Autolayout. ☠️
- Networking and Promises. 🎁
- Forms 📖
- CoreData ⚙️
- Soft Skills 👔

---

## And of course...
## **tests**

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

_Video of the app_

---

###Schedule:

|   **Friday**  |    **Saturday**   |   **Sunday**	|
| ----------- | :-----------: | -----------: |
| Stateful VC        |    Networking    |    CoreData    |
| Autolayout        |   Promises    |   UITests   |
| Size Classes        |   Forms    |   Soft Skills   |


---

#...But first

---

## Bootstrapping
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

However...

---

![inline](https://i.imgur.com/1gTA9p3.png)

---

![inline](https://i.imgur.com/uMkcolO.png)

---

In this cases:

`UIKit` → UI Module → Framework
`Foundation` → Foundation Module → Framework

---

#_Demo_

^ Create Framework target
Restructure file system
Git commit push.

---

##Dependency Management

The de-facto standard is Cocoapods

However, Carthage is also a challenger.

---

![inline](https://i.imgur.com/saB2GNN.png)

---

##Tradeoffs:

- Add `/Pods` to repo.
- Xcode integration.

---

#_Demo_
^ Create Podfile
Add SDWebImage to app Target
Add Alamofire to Framework Target
Git commit push.

---

##Schemes for Development and AppStore

---

![inline](https://i.imgur.com/2ayPd87.png)

---

### We shouldn't use the two target approach

- It 2x `xcodeproj` file's size.
- It leads to errors when adding new files.
- It feels dirty.

---
### Instead, we should use schemes
- With schemes we can get the same result.
- We can parametrize what we need using `xcconfig`
- It's predictable.

---
#_Demo_
^ Duplicate scheme
Modify schemes to always do the same things
Create xcconfig
Modify build configuration
Parametrize name
Parametrize in code
Git commit push.

---

#Tests setup
- Tests should not launch your app's logic.
- Integrate with Snapshot testing.
- Separate App Tests from Model tests

---
#_Demo_
^ Add `XCTestCase` check
Add snapshot tests
Show how one scheme builds only one test bundle
Git commit push.

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

#_Demo_
^ Create bitrise account from github
Add Repo
Git commit push.

