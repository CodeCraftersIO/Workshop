# AutolayoutTests

- Checkout `5e20420`. Show `AutolayoutTestsVC` and how the system can figure out the size using `intrinsicContentSize` using iPhone SE:

- Show how the layout system can figure out intrinsicContentSize for views with enough constraints:
	- Add a label with margins:

```
class RedView: UIView {
    override init(frame: CGRect) {
        super.init(frame: frame)
        translatesAutoresizingMaskIntoConstraints = false
        backgroundColor = .red
        let label = UILabel(frame: .zero)
        label.translatesAutoresizingMaskIntoConstraints = false
        label.text = "Code Crafters"
        self.addSubview(label)

        NSLayoutConstraint.activate([
            label.leadingAnchor.constraint(equalTo: self.leadingAnchor, constant: 5),
            label.trailingAnchor.constraint(equalTo: self.trailingAnchor, constant: -5),
            label.topAnchor.constraint(equalTo: self.topAnchor, constant: 5),
            label.bottomAnchor.constraint(equalTo: self.bottomAnchor, constant: -5)
            ])
    }

    required init?(coder aDecoder: NSCoder) { fatalError("init(coder:) has not been implemented") }
}

```
- Safe Areas:
	- Start by adding the RedView to the top of the view and showing how it grows. 
- Show how safeAreaInsets are propagated into subviews by printing it:

```swift
override func viewDidAppear(_ animated: Bool) {
  super.viewDidAppear(animated)
  print(view.safeAreaInsets)
  print(redView.safeAreaInsets)
}
``` 
- Remove the RedView and add a GradientView as background
- Re-add the RedView to the top. Run the app on iPhoneX, show that it grows even more. 


#DetailVC

Checkout `0a24ec1` and show the core around:

- Show the VM of the details.
- Explain the interactor part and the async
- Explain the mock load changes


## All constraints approach
- Let's add the GIFImageView
- Notice that if you forget translatesAutoresizingMaskIntoConstraints shit happens. 
- Create `addAutolayoutView`
- Add `pod 'TagListView', '~> 1.3.2'`
- Create all the view using just constraints (`c57389e`)

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
scrollView.pinToSuperview()
let containerView = UIView()
scrollView.addAutolayoutView(containerView)
containerView.pinToSuperview()

// Add Subviews
containerView.addAutolayoutView(self.imageView)
containerView.addAutolayoutView(detailsStackView)
```


add pinToSuperview()

```swift
    public func pinToSuperview(withEdges edges: UIEdgeInsets = UIEdgeInsets(top: 0, left: 0, bottom: 0, right: 0)) {
        guard let superView = superview else { return }
        translatesAutoresizingMaskIntoConstraints = false
        NSLayoutConstraint.activate([
            leadingAnchor.constraint(equalTo: superView.leadingAnchor, constant: edges.left),
            trailingAnchor.constraint(equalTo: superView.trailingAnchor, constant: -edges.right),
            topAnchor.constraint(equalTo: superView.topAnchor, constant: edges.top),
            bottomAnchor.constraint(equalTo: superView.bottomAnchor, constant: -edges.bottom)
            ])
    }
```

- Rememeber to add:

```swift
let layoutMargins = containerView
...
stackView.bottomAnchor.constraint(equalTo: containerView.bottomAnchor),
containerView.widthAnchor.constraint(equalTo: scrollView.widthAnchor),
```

## Full stackView approach

- Having a container view, a stackView and an imageView feels dirty. We could have everything inside a stackView and use that as the containerView
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

- Refactor UILabel creation (1ff0141)

```swift
extension UILabel {

    public static func autolayoutLabelWith(textStyle style: UIFont.TextStyle) -> UILabel {
        let label = UILabel()
        label.translatesAutoresizingMaskIntoConstraints = false
        label.font = UIFont.preferredFont(forTextStyle: style)
        label.numberOfLines = 0
        return label
    }
}
```
- Fix imageView aspect ratio (f7e11bb)

```swift
self.imageView.sd_setImage(with: vm.url) { (image, _, _, _) in
  guard let image = image else { return }
  let aspectRatio = image.size.width/image.size.height
  self.imageView.widthAnchor.constraint(equalTo: self.imageView.heightAnchor, multiplier: aspectRatio).isActive = true
}
```

- For Students: Create a `ScrollableStackView` subclass

## Rotation:

![](https://i.imgur.com/GxhBrst.jpg)

- Start with basic support:

```swift
    override func viewWillTransition(to size: CGSize, with coordinator: UIViewControllerTransitionCoordinator) {
        super.viewWillTransition(to: size, with: coordinator)
        coordinator.animate(alongsideTransition: { (context) in
            self.configureStackViews(forContainerViewSize: size)
        }, completion: nil)
    }
    
    private func configureStackViews(forContainerViewSize size: CGSize) {
        if size.width > size.height {
            self.containerStackView.axis = .horizontal
        } else {
            self.containerStackView.axis = .vertical
        }
    }
```

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

// Iin the regular constraints setting
        topSpacingView.heightAnchor.constraint(equalTo: bottomSpacingView.heightAnchor),


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


// In configureForViewModel, after the image is retrieved (remove prev line setting the same constraint)
  let aspectRatioConstraint = self.imageView.widthAnchor.constraint(equalTo: self.imageView.heightAnchor, multiplier: aspectRatio)
  aspectRatioConstraint.priority = .defaultHigh
  aspectRatioConstraint.isActive = true


```

- All ready on `bc02ff4`

#Snapshot tests

- Start with `efce8ed` and show the code around.
- Show that the snapshot tests are not passing due to the image being downloaded, taking time to do so.
- Change the downloading of the image to use our custom wrapper from the UIImageView extension instead.
- Show that the imageView is all shrunk since there is no aspectRatio
	- Create one at 1:1 and then set it on the download block in order to make the test more real

```swift 
self.imageAspectRatioConstraint = self.imageView.widthAnchor.constraint(equalTo: self.imageView.heightAnchor, multiplier: 1)
```
