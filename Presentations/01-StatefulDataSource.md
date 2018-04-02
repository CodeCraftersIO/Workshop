theme: Work, 1

##*Stateful* DataSource

---

##*Stateful* `UICollectionView` DataSource


---

![](https://i.imgur.com/ucK4lA2.png)

---

![](https://i.imgur.com/rO2oPg7.png)

---

##Swift == Stateless
#🤔

---

In Swift you can be more *explicit* about the state 

---
## ...back in Objective-C

---

[.background-color: #495061]
![fit](https://i.imgur.com/B9qPkR7.png)

---

## what if...

---

```swift

private func fetchData() {
  self.collectionView.state = .loading
  presenter.fetchLocationAndUsers { (result) in
    switch result {
    case .failure(let error):
        self.collectionView.state = .failure(error)
    case .success(let users):
        self.collectionView.state = .loaded(users)
    }
  }
}

```

---

# However, last time I tried to do `collectionView.state` this happened:

```
Value of type 'UICollectionView' has no member 'state'
```

# So what, then?

---

# Apple provides 3 point of customizations:

1. Subclassing
2. `UICollectionViewFlowLayout`
3. `UICollectionViewDataSource`

---

# Apple provides 3 point of customizations:

1. ~~Subclassing~~
2. ~~`UICollectionViewFlowLayout`~~
3. `UICollectionViewDataSource` ✅

---

#Objectives:
- Easier to use than `UICollectionViewDataSource`.
- Map one model object to one cell kind at **compile time**.
- Support for loading, error and empty state.

---

![fit](https://i.imgur.com/ltyGFuo.png)

---

[.background-color: #495061]
![fit](https://memegenerator.net/img/instances/42555568/talk-is-cheap-show-me-the-code.jpg)

---

#Now, Snapshot Tests

---

#FB Snapshot Tests

- Allow us to refactor view code with ease.
- Tests results are analized in the problem domain: *Pixels*
	- $$f(vm,ss) = screenshot.png$$
- Built by Facebook, currently maintained by Uber.

---

#How it works:

- Set expectations
	- Create `UIView`/`UIViewController`
	- Inject dependencies
- Run the method
	- Layout the View
	- Generate a PNG for the View 
- Compare to expectation
	- Run a diff 

---

```swift
class ProfileViewControllerTests: BSWSnapshotTest {

    func testSampleLayout() {
        let viewModel = ProfileViewModel.sampleVM()
        let detailVC = ProfileViewController(
          viewModel: viewModel
        )
        let navController = UINavigationController(
          rootViewController: detailVC
        )
        waitABitAndVerify(viewController: navController)
    }
}
```
---

![fit](https://www.objc.io/images/issue-15/snapshots-reference-59b0b96b.png)

---

![fit](https://i.pinimg.com/originals/bd/92/a0/bd92a01c5c6c48833a97a504d4046b75.jpg)

---

#Takeaways

- Be aware of the tradeoffs before writing any abstraction.
	- Bad abstractions are very, very expensive.
- Don't be afraid to *refactor*.
	- Cover yourself here with Tests.

---
#Takeaways

- Use Swift's extensions to declare types.
	- Don't litter the global namespace.
	- Improves compile time and code completion.
	- Very useful with types like VM or Cells that are only used within a class.