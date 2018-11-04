# Automatic Sizing for `UICollectionViewCell`s

---

# History

---

# History: `UITableView`

- iOS was designed for 3.5' screens.
- Subclass of `UIScrollView`
	- Table Views only scrolled vertically
	- `contentSize` was always limited to the table view's width 
- *Performance* was key
 
![left, fit](https://oleb.net/media/appointmentlist-screenshot-iphone.png)

---
# `UITableView`'s points of customization

1. `UITableViewDataSource`
	- This configures the *what* is shown on-screen
2. `UITableViewDelegate` 	
	- This configures the *how* is shown on-screen
	- This *notifies* of user input

---

# UITableView's Layout

- Cells width match the tableView's width.
- Cells are shown one below the other, in rows.
- Every cell's height must be calculated and returned using `tableView:heightForRowAtIndexPath:`.
	- This is done for *every* row in the tableView.

---

![fit](https://i.imgur.com/8ASs3d0.png)

---

![fit](https://media.giphy.com/media/Bcpspr9LTSvss/giphy.gif)

---

# Enter `UITableViewAutomaticDimension`

![left fit] (https://media.giphy.com/media/pqFbHWj0vRqmY/giphy.gif)

---

# `UITableViewAutomaticDimension`

- As long as your subviews are *pinned* to the edges
- Constraints will have to be pinned to `contentView`, not the cell.
- UIKit will do a fake layout pass to calculate the cell's height
	- Profit 🤑

![right, fit](https://i.imgur.com/tjKsI9t.png)

---

$$
cellHeight=f(cellWidth,cellContent)
$$

---

#What about `UICollectionView`?

---

#Differences between `UICollectionView` and `UITableView`

- Collections can scroll on both axis.
- Layout is completely customizable.
- Layout can be swapped at run time.
- Built with big screens and complex interactions in mind.

---

![fit](https://ksassets.timeincuk.net/wp/uploads/sites/54/2012/08/Pinterest-ipad-app-1-1620x954.png)

---

![fit](https://koenig-media.raywenderlich.com/uploads/2015/05/final-scrolling.gif)

---

![fit](https://media.giphy.com/media/26mkhMYkitO7DoJuU/giphy.gif)

---

# `UICollectionView`'s points of customization


1. `UICollectionViewDataSource`
	- This configures the *what* is shown on-screen
2. `UICollectionViewDelegate` 	
	- This *notifies* of user input
3. `UICollectionViewLayout`
	- This configures the *how* is shown on-screen

---

#`UICollectionViewLayout`

- Abstract class that defines:
	- `UICollectionViewLayoutAttributes` for each cell
	- The collectionView's content size

---

#`UICollectionViewLayoutAttributes`
```
@available(iOS 6.0, *)
open class UICollectionViewLayoutAttributes : NSObject, NSCopying, UIDynamicItem {    
    open var frame: CGRect
    open var center: CGPoint
    open var size: CGSize
    open var transform3D: CATransform3D
    open var bounds: CGRect
    open var transform: CGAffineTransform
   	open var alpha: CGFloat
    open var zIndex: Int
    open var isHidden: Bool
    open var indexPath: IndexPath
    open var representedElementCategory: UICollectionView.ElementCategory { get }
    open var representedElementKind: String? { get }
}
```
---

#`UICollectionViewLayout`

- Most commonly, you use `UICollectionViewFlowLayout`.

![right fit](https://media.giphy.com/media/yYZOdU9PjcIDK/giphy.gif)

---

#`UICollectionViewFlowLayout`

> A concrete layout object that organizes items into a grid with optional header and footer views for each section.

---

#`UICollectionViewFlowLayout`

- This gives class tons of customization points:
	- Scroll Direction
	- *Item Size*
	- Item Spacing
	- Line Spacing 
	- *Estimated Item Size* (iOS 8+)

---

#WTF is the difference about `itemSize` and `estimatedItemSize`?

![left fit](https://media.giphy.com/media/aZ3LDBs1ExsE8/giphy.gif)

---

#`itemSize`
- Final size of the cell on the layout.

---

#`itemSize`
- Final size of the cell on the layout.
- That's it 🤡

---

#`estimatedItemSize`

- Hints to `UICollectionViewFlowLayout` about the probable size of a cell.
	- Enables first layout pass without querying doing the math for every cell before they're visible.

---

#`estimatedItemSize`

- The actual size is retrieved calling `preferredLayoutAttributesFittingAttributes:` on the *cell*.
- The collectionView's layout object will call this method passing appropiate `UICollectionViewLayoutAttributes`.

---

![fit](https://i.imgur.com/L2CArik.png)

---

![fit](https://i.imgur.com/B4Mjwlg.png)

---

#Where is `UICollectionViewFlowLayoutAutomaticSize`?

---

#This math is a lot harder:

$$
cellHeight=f(cellWidth,cellContent)
$$


---

# First conclusions:

- Now you know why these terms are used interchangeably: 
	- `Automatic cell sizing`
	- `Flow layout automatic sizing` 


---

#Demo

---

#Takeaways:

---

#Takeaways:

- For your layout code, please:
	- Use `UIStackView`
		- And set it to `.fill` & `.fill` 
	- Use `layoutMargins`

---

| What do I have to build? | What do I use? |
| --- | --- | 
| Form | `UITableView` |
| Literally anything else | `UICollectionView` |

---

#Documentation:

- A Tour of UICollectionView, [WWDC 2018 - Session 225](https://developer.apple.com/videos/play/wwdc2018/225/)
- UICollectionView Custom Layout Tutorial: Pinterest, [raywenderlich.com](https://www.raywenderlich.com/392-uicollectionview-custom-layout-tutorial-pinterest)