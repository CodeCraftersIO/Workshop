theme: Huerta

#**Auto**Layout

---

#Problem to solve:

Laying out views when there are changes in:

	- Screen size
	- Rotations
	- Subview content
	- Internationalization
	- Dynamic Type
	- Size class

---

![fit](https://developer.apple.com/library/content/documentation/UserExperience/Conceptual/AutolayoutPG/Art/layout_views_2x.png)

---

![fit](https://developer.apple.com/library/content/documentation/UserExperience/Conceptual/AutolayoutPG/Art/layout_constraints_2x.png)

---

###*The logic used to design a set of constraints to create specific behaviors is very different from the logic used to write procedural or object-oriented code.*[^1]

[^1]: Understanding Auto Layout. [developer.apple.com](https://developer.apple.com/library/content/documentation/UserExperience/Conceptual/AutolayoutPG/).

---

##Let's back up a bit...

---

![inline 100%](https://i.imgur.com/3tynz2a.png)

---

##"...the **constraint**-based system"

---

##"...the ~~constraint~~ **equation**-based system"

---

##AutoLayout is just an **equation system** where the variables are the **view's frames**

---

> Equation == Constraint

---

##A constraint is just an equation that expresses on an axis either:

- The distance between two sibling views
- The size of a view

---

#If we think about it:

```swift
view.leadingAnchor.constraint(
  equalTo: superView.leadingAnchor, 
  constant: 10
)
```

---
#Can be translated to[^2]:

$$
view.left == superView.left + 10
$$

[^2]: [Cartography](https://github.com/robb/Cartography).


---
##How many equations do we need?

Each view has to have a defined `frame`, which means:

- _origin_: `x`, `y`
- _size_: `width`, `height`

---

###You add enough constraints until the layout system can solve **all 4** variables for every view.

---

![inline 100%](https://i.imgur.com/3tynz2a.png)

---

![inline 100%](https://i.imgur.com/4QrKrmS.png)

---

![fit](https://i.imgur.com/vzOKzjX.png)

---

![inline 100%](https://i.imgur.com/PhUam6k.png)

---

```
Unable to simultaneously satisfy constraints
	Probably at least one of the constraints in the 
	following list is one you don't want. 

Will attempt to recover by breaking constraint 
<NSLayoutConstraint:0x698e70 UIView: 0x698e43.width == 30   (active)>

```

---

##How to add constraints (responsibly):

1. A `UIView` should provide constraints for it's subviews positions
2. A `UIView` should provide it's size:
	- Using width and height constraints
	- Using `intrinsicContentSize`

---
##How to create constraints:

```swift
let widthConstraint = redView.widthAnchor.constraint(equalToConstant: 30)
let heightConstraint = redView.heightAnchor.constraint(equalToConstant: 30)
```

And then:

```swift
NSLayoutConstraint.activate([
  widthConstraint, 
  heightConstraint
])
```

---

#What happens next?

---

##Layout cycle

Adding a constraint will indirectly call `setNeedsLayout` to a view, triggering a layout pass where the layout system will:

- Finds the top-most view (normally visible's `UIViewController`'s view)
- Resolve the top-most view hierarhy using the screen size.
	- Positions and sizes for the top-most view's subviews will be resolved.
- The layout system will recursively resolve the subviews hierarcy based on the size of all the subviews.

---
##Layout cycle: customization

The layout system will call:

-`viewWillLayoutSubviews` on `UIViewController` 

-`layoutSubviews` on `UIView`

The resolved frame for the current view will be passed as an argument for customization.

**Don't** call `setNeedsLayout` during these methods.

---

#Demo

^ Create a sample UIViewController.
Create a sample UIView
Add it to view hierarchy using constraints
Remove size constraints, use intrinsic contentSize
The add a label, play with it for a bit

---

#Layout Guides

Defines a rectangular region where layout can occurr safely in their owning view’s coordinate system.

![inline](https://useyourloaf.com/assets/images/2017/2017-04-28-001.png)

---

#Layout Guides

```swift
let layoutGuide: UILayoutGuide = ...
let constraints = [
	label.leadingAnchor.constraint(equalTo: layoutGuide.leadingAnchor),
	label.trailingAnchor.constraint(equalTo: layoutGuide.trailingAnchor),
	label.topAnchor.constraint(equalTo: layoutGuide.topAnchor),
	label.bottomAnchor.constraint(equalTo: layoutGuide.bottomAnchor)
]
NSLayoutConstraint.activate(constraints)

```
---

#Layout Guides

There are three types:

- Margin.
- SafeArea.
- Custom.

---

#Layout **Margins** Guides

Every `UIView` has one to reflect the `layoutMargins` property.

```swift
self.layoutMargins = UIEdgeInsets(top: 5, left: 5, bottom: 5, right: 5)
let constraints = [
	label.leadingAnchor.constraint(equalTo: self.layoutMarginsGuide.leadingAnchor),
	label.trailingAnchor.constraint(equalTo: self.layoutMarginsGuide.trailingAnchor),
	label.topAnchor.constraint(equalTo: self.layoutMarginsGuide.topAnchor),
	label.bottomAnchor.constraint(equalTo: self.layoutMarginsGuide.bottomAnchor)
]
NSLayoutConstraint.activate(constraints)

```

**Note:** you can't modify a `UIViewController`'s view's margins.

---

#Safe Area Layout Guide

![inline](https://docs-assets.developer.apple.com/published/dbcc36bfb3/e5aca39a-f9a2-4ab8-9f45-08fd95fb845c.png)

---
#Safe Area Layout Guide

Prevents showing content underneath:
- `UINavigationBar`
- `UITabBar`
- `UIStatusBar`
- iPhone X home indicator.


✅ Automatically set for us on `UIViewController`
👀 Can be modified with `additionalSafeAreaInsets`

---

#Safe Area Layout Guide

- All `UIControl`s in your view must respect this


![left fit](https://i.imgur.com/0QLeB9O.jpg)

---

#Custom guides:

![inline](https://useyourloaf.com/assets/images/2016/2016-02-27-001.png)

---

#Custom guides[^3]:
![inline](https://useyourloaf.com/assets/images/2016/2016-02-27-002.png)

[^3]: Goodbye Spacer Views Hello Layout Guides. [useyourloaf.com](https://useyourloaf.com/blog/goodbye-spacer-views-hello-layout-guides/).

---

#Demo

---

#Best practices:


---

# View layout

- From top to bottom on the view hierarchy.

- Create constraints and activate them all at once using `NSLayoutConstraint.activate()`.

- Don't create more constraints than you need.

- Rely on `intrinsicContentSize` for sizes when possible.

---
# `intrinsicContentSize`

- Your custom view's should not override this method
	- Instead, create constraints that will make the system infer it for you.

![inline](https://i.stack.imgur.com/CTUPi.png)

---

# `UILabel.intrinsicContentSize`

- ⛔️ *Never* constraint the width/height

- ✅ Set `numberOfLines` to 0 and leading and trailing constraints instead of width.
- ☑️ Also, set `contentHuggingPriority` and `contentCompressionResistancePriority` appropiately.
 
---

# `UIImageView.intrinsicContentSize`

- It will default to the image's size
- Create a constraint for:
	- The `aspectRatio` 
	- The max allowed `aspectRatio`
	- The width or height.

---

# `UIStackView`

- Use `UIStackView` as much as possible.

- Remember that they don't have an `intrinsicContentSize`.

	- However, if the system can infer a width (vertical) or height (horizontal) for this stackView, then it uses the **fitting size** for this stackView:

```swift
let size: CGSize = stackView.systemLayoutSizeFitting(
    CGSize(width: 100, height: 0),
    withHorizontalFittingPriority: .required,
    verticalFittingPriority: .fittingSizeLevel
)
```

---
# Debugging

- Xcode's Visual Hierarchy debugger is your friend.

![inline](https://developer.apple.com/library/content/documentation/DeveloperTools/Conceptual/debugging_with_xcode/Art/dwx-sw-dvh-1_2x.png)

---

# Debugging

- Remember to turn of `translatesAutoresizingMaskIntoConstraints`.
- If a constraint is being broken by the engine, and the layout looks correct:
	- The engine did the right thing now, but this doesn't mean that it will do the same thing on other OS versions. 
	- Set that constraint's priority lower to optimize the performance. 

---

# Debugging

- Content sizes are also represented in the layout engine

![inline](https://i.imgur.com/Rb4jgbT.png)
