
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

![fit](https://memegenerator.net/img/instances/42555568/talk-is-cheap-show-me-the-code.jpg)