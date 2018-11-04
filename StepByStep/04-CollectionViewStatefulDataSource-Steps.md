StatefulDataSource

Step 1 (finished at 43db272):

- Begin creating ListState<T> and show how it should look:

```swift
public enum ListState<T> {
    case loading
    case loaded(data: [T])
    case failure(error: Error)
}
```

- Start creating CollectionViewStatefulDataSource with a not-generic ListState:

```swift
class CollectionViewStatefulDataSource: NSObject, UICollectionViewDataSource {

    init(state: ListState<GIFWalletViewController.VM>,
         collectionView: UICollectionView,
         cellType: UICollectionViewCell.Type) { }
}
```

- Register cells and dequeue them
- Build and run, it should show the cells, but empty. On the VC

```
dataSource = CollectionViewStatefulDataSource(
  state: .loaded(data: MockLoader.mockCellVM()),
  collectionView: collectionView,
  cellType: GifCell.self
)
```
Step 2 (finished at e586421):

- Try to do cell.configureFor(model) and see that the cell doesn't have that method
- Create GIFViewModelConfigurable without associatedType

```swift
protocol GIFViewModelConfigurable {
    func configureFor(vm: GIFWalletViewController.VM)
}
```

- Try to add GIFViewModelConfigurable to the init and see that it fails:

![](https://i.imgur.com/EzTeiQf.png)

- Add it to the generic definition of the DataSource

```swift
class CollectionViewStatefulDataSource<Cell: GIFViewModelConfigurable & UICollectionViewCell>: NSObject, UICollectionViewDataSource { }
```

- Add the register/dequeue dance 
- Show that now cell.configureFor now works
- Fix compilation issues on GIFWalletVC
- Show that it works now

Step 3 (d131412):

- Explain that this is not reusable, and we should strive for reusable code
- Go to ViewModelConfigurable and add the associatedtype

```swift
protocol ViewModelConfigurable {
    associatedtype VM
    func configureFor(vm: VM)
}

```
- Show that cell.configureFor doesn't work because models don't match
- Change the ListState to go from a specified model to use the Cell's model
- Profit

Step 3.X with students:

- Create extension to collectionView to avoid stringly typed register/dequeue (296d186)
- Add states for loading(a49f2f7)
- Make them choose between:
	- Empty and error states 
	- Pull to Refresh
	- Reorder
	- State customization 
	- Snapshot tests