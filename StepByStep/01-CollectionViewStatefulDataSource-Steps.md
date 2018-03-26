StatefulDataSource

Step 0 (finished at 63e7226):
- Show VC and UICollectionViewCell code as-is, there'll be time to dig in later
- Show how the collectionView is created and FlowLayout is created
- Explain the differences between dataSource, delegate, and layoutDelegate (SLIDES!)

Step 1 (finished at 3531555):
- Begin with ListState<T> and show how it should look
- Start creating CollectionViewStatefulDataSource with a not-generic ListState
- Register cells and dequeue them
- Build and run, it should show the cells, but empty

Step 2 (finished at 00a38ae):

- Try to do cell.configureFor(model) and see that the cell doesn't have that method
- Create GIFViewModelConfigurable without associatedType
- Try to add GIFViewModelConfigurable to the init and see that it fails
- Add it to the generic definition of the DataSource
- Add the register/dequeue dance 
- Show that now cell.configureFor now works
- Fix compilation issues on GIFWalletVC
- Show that it works now

Step 3 (d131412):

- Explain that this is not reusable, and we should strive for reusable code
- Go to ViewModelConfigurable and add the associatedtype
- Show that cell.configureFor doesn't work because models don't match
- Change the ListState to go from a specified model to use the Cell's model
- Profit

Step 3.X with students:

- Create extension to collectionView to avoid stringly typed register/dequeue (296d186)
- Add states for loading/empty/error (07bbf5c)
	- Close all the loop using an interactor to mock the network request (fc758bd)

- Make them choose between:
	- Pull to Refresh
	- Reorder
	- State customization 
