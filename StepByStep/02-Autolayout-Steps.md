#AutolayoutTests

- Create `AutolayoutTestsVC` and show how the system can figure out the size using intrinsicContentSize:

```swift
class AutoLayoutTestsViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        let redView = RedView(frame: .zero)
        view.addSubview(redView)

        // Specify origin
        redView.leadingAnchor.constraint(equalTo: self.view.leadingAnchor).isActive = true
        redView.topAnchor.constraint(equalTo: self.view.topAnchor).isActive = true
    }
}

class RedView: UIView {
    override init(frame: CGRect) {
        super.init(frame: frame)
        translatesAutoresizingMaskIntoConstraints = false
        backgroundColor = .red
    }

    override var intrinsicContentSize: CGSize {
        return CGSize(width: 50, height: 50)
    }

    required init?(coder aDecoder: NSCoder) { fatalError("init(coder:) has not been implemented") }
}
```

- Show how the layout system can figure out intrinsicContentSize for views with enough constraints:
	- Add a label with margins

- Show how safeAreaInsets are propagated into subviews by printing it:

```swift
override func viewDidAppear(_ animated: Bool) {
  super.viewDidAppear(animated)
  print(view.safeAreaInsets)
  print(redView.safeAreaInsets)
}
``` 

#DetailVC

Checkout `2815a7f` and show the core around:

- Show the VM of the details.
- Explain the interactor part and the async
- Explain the mock loade changes


## All constraints approach
- Let's add the GIFImageView
- Notice that if you forget translatesAutoresizingMaskIntoConstraints shit happens. 
- Create `addAutolayoutView`
- Add `pod 'TagListView', '~> 1.0'`
- Create all the view using just constraints (`7d0e691`)

- 
```swift
// Actually layout the view
self.view.addAutolayoutView(self.imageView)
self.view.addAutolayoutView(self.titleLabel)
self.view.addAutolayoutView(self.subtitleLabel)
self.view.addAutolayoutView(self.tagView)
let layoutMargins = self.view.safeAreaLayoutGuide
NSLayoutConstraint.activate([
	self.imageView.leadingAnchor.constraint(equalTo: layoutMargins.leadingAnchor),
	self.imageView.topAnchor.constraint(equalTo: layoutMargins.topAnchor),
	self.imageView.trailingAnchor.constraint(equalTo: layoutMargins.trailingAnchor),
	self.titleLabel.topAnchor.constraint(equalTo: imageView.bottomAnchor, constant: 5),
	self.titleLabel.leadingAnchor.constraint(equalTo: layoutMargins.leadingAnchor, constant: 5),
	self.titleLabel.trailingAnchor.constraint(equalTo: layoutMargins.trailingAnchor, constant: 5),
	self.subtitleLabel.topAnchor.constraint(equalTo: titleLabel.bottomAnchor, constant: 5),
	self.subtitleLabel.leadingAnchor.constraint(equalTo: layoutMargins.leadingAnchor, constant: 5),
	self.subtitleLabel.trailingAnchor.constraint(equalTo: layoutMargins.trailingAnchor, constant: 5),
	self.tagView.topAnchor.constraint(equalTo: subtitleLabel.bottomAnchor, constant: 5),
	self.tagView.leadingAnchor.constraint(equalTo: layoutMargins.leadingAnchor, constant: 5),
	self.tagView.trailingAnchor.constraint(equalTo: layoutMargins.trailingAnchor, constant: 5),
])
```

## Bottom StackView approach

- Rewrite details views (not the imageView) using StackView (0f3695b)

```swift
// Now the details' StackView
let stackView = UIStackView(arrangedSubviews: [self.titleLabel, self.subtitleLabel, self.tagView])
stackView.axis = .vertical
stackView.distribution = .fill
stackView.alignment = .fill
stackView.spacing = 10
stackView.layoutMargins = UIEdgeInsets(top: 5, left: 5, bottom: 5, right: 5)
stackView.isLayoutMarginsRelativeArrangement = true
self.view.addAutolayoutView(stackView)
```

## Adding a scrollView

- Place inside scrollView using UIView as containerView (1102fe3)

```swift
// Add UIScrollView & Container-View
let scrollView = UIScrollView()
scrollView.alwaysBounceVertical = true
self.view.addAutolayoutView(scrollView)
scrollView.pinToSuperviewSafeLayoutEdges()
let containerView = UIView()
scrollView.addAutolayoutView(containerView)
containerView.pinToSuperview()

// Add Subviews
containerView.addAutolayoutView(self.imageView)
containerView.addAutolayoutView(detailsStackView)
```

- Rememeber to add:

```swift
containerView.widthAnchor.constraint(equalTo: scrollView.widthAnchor),
```

## Full stackView approach

- Use stackView as containerView

```swift
private let containerStackView: UIStackView = {
	let stackView = UIStackView()
	stackView.axis = .vertical
	stackView.distribution = .fill
	stackView.alignment = .fill
	return stackView
}()

private let scrollView: UIScrollView = {
	let scrollView = UIScrollView()
	scrollView.alwaysBounceVertical = true
	return scrollView
}()
```

## Cleanup:

- Refactor UILabel creation (58eaf7a)

```swift
extension UILabel {

    public static func autolayoutLabelWith(textStyle style: UIFontTextStyle) -> UILabel {
        let label = UILabel()
        label.translatesAutoresizingMaskIntoConstraints = false
        label.font = UIFont.preferredFont(forTextStyle: style)
        label.numberOfLines = 0
        return label
    }
}
```
- Fix imageView aspect ratio (f0b9bc3)

```swift
self.imageView.sd_setImage(with: vm.url) { (image, _, _, _) in
  guard let image = image else { return }
  let aspectRatio = image.size.width/image.size.height
  self.imageView.widthAnchor.constraint(equalTo: self.imageView.heightAnchor, multiplier: aspectRatio).isActive = true
}
```

- Maybe create a `ScrollableStackView` subclass

## Rotation:

![](https://i.imgur.com/GxhBrst.jpg)

- Show that we'll need some spacing views:

```swift
    class func verticalSpacingView() -> UIView {
        let view = UIView()
        view.translatesAutoresizingMaskIntoConstraints = false
        view.setContentCompressionResistancePriority(.defaultLow, for: .vertical)
        return view
    }
```

- Add it to the top and to the bottom of the detailsStackView. 

- Create landscapeConstraints and portraitConstaints:

```swift
private var landscapeConstraints: [NSLayoutConstraint]!
private var portraitConstraints: [NSLayoutConstraint]!

...
        portraitConstraints = [
            topSpacingView.heightAnchor.constraint(lessThanOrEqualToConstant: 0),
        ]

        landscapeConstraints = [
            imageView.widthAnchor.constraint(lessThanOrEqualTo: detailsStackView.widthAnchor, multiplier: 2, constant: 0),
            containerStackView.heightAnchor.constraint(equalTo: scrollView.heightAnchor)
        ]
...

private func configureStackViews(forContainerViewSize size: CGSize) {
	if size.width > size.height {
		self.containerStackView.axis = .horizontal
		landscapeConstraints.forEach { $0.isActive = true }
		portraitConstraints.forEach { $0.isActive = false }
	} else {
		self.containerStackView.axis = .vertical
		landscapeConstraints.forEach { $0.isActive = false }
		portraitConstraints.forEach { $0.isActive = true }
	}
}

```

- All ready on d9b78dd

#Snapshot tests

- Show that the imageView is all shrunk since there is no aspectRatio
	- Create one at 1:1 and then set it on the download block in order to make the test more real

```swift 
self.imageAspectRatioConstraint = self.imageView.widthAnchor.constraint(equalTo: self.imageView.heightAnchor, multiplier: 1)
```