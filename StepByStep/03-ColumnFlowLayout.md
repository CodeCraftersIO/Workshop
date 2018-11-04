#ColumnFlowLayout

- Checkout `12c2a8d`
	- Show the code around
	- Run it and show the cells 50x50
- Add `collectionViewLayout.estimatedItemSize = UICollectionViewFlowLayout.automaticSize` and show that it still sucks
- Now let's override `preferredLayoutAttributesFittingAttributes`
	- Start by dumping this code in:

	```
override func preferredLayoutAttributesFitting(_ layoutAttributes: UICollectionViewLayoutAttributes) -> UICollectionViewLayoutAttributes {
        let estimatedSize = contentView.systemLayoutSizeFitting(
            CGSize(width: layoutAttributes.frame.size.width, height: UIView.layoutFittingCompressedSize.height),
            withHorizontalFittingPriority: .required,
            verticalFittingPriority: .fittingSizeLevel
        )
        layoutAttributes.frame.size = estimatedSize
        return layoutAttributes
    }
``` 
	- Then go back to ViewController.swift and add:
	
```
flowLayout.estimatedItemSize = CGSize(width: view.frame.size.width, height: 200)

```
- Point out that's it's a good starting point, but lot's of improvements:
	- The available width may be less if we want to add padding for example
		- For example: `flowLayout.sectionInset = UIEdgeInsets(uniform: 30)`

- See that now it's not showing anything because `collectionView.frame` is not ready, add that code in `viewDidLayoutSubviews` and show that it kind of looks decent now.
	- Note the tension that we're feeling between the VC lifecycle and the UICollectionView API.

- Subclass `UICollectionViewFlowLayout`:
	- Note that we're overriding prepare because that's called whenever the layout is invalidated
	- It's much more natural to add the estimatedItemSize here instead.

```
class ColumnFlowLayout: UICollectionViewFlowLayout {
    override func prepare() {
        super.prepare()
        guard let cv = collectionView else { return }
        self.estimatedItemSize = CGSize(width: cv.bounds.size.width, height: 200)
    }
}
```

- Now let's play with the width available:

```
let availableWidth = cv.bounds.inset(by: cv.layoutMargins).size.width
        
let minColumnWidth = CGFloat(200)
let maxNumColumns = Int(availableWidth / minColumnWidth)
let cellWidth = (availableWidth / CGFloat(maxNumColumns)).rounded(.down)
self.estimatedItemSize = CGSize(width: cellWidth, height: 200)
self.sectionInsetReference = .fromSafeArea
```

- Also point out that we still rely on the cell's implementation of `preferredLayoutAttributesFitting`

- Go back to slides and explain what what we need to implement from `UICollectionViewLayout`:
	- `prepare()`
	- `collectionViewContentSize`
	- `layoutAttributesForElements(in:)`
	- `layoutAttributesForItem(at:)`

- Checkout `e26a77c` and tour the code.
- Add snapthot tests `9078954`.
- Run on iPad and see that it works without chaning a line of code.

	