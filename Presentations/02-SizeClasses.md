theme: Zurich-Better


# Trait Collections

---

![fit](https://i.imgur.com/GF1n758.png)

[.background-color: #f2f2f2]

---

![fit](https://i.imgur.com/dSBsux8.png)

[.background-color: #f0f0f0]

---

![fit](https://i.imgur.com/DlwfqDq.png)

[.background-color: #FFFFFF]

---

![fit](https://i.blogs.es/b90771/iosmacosunified-2/1366_2000.jpg)

[.background-color: #d0d6e2]

---

![fit](https://2e8ram2s1li74atce18qz5y1-wpengine.netdna-ssl.com/wp-content/uploads/2018/06/Apple-Marzipan-iOS-macOS-AppKit-UIKit-Dice-1024x484.jpg)

---

#Trait collections allows us to adapt the interface, not only to screen sizes, but to different devices. 

![inline](https://i.imgur.com/RxlWLKJ.png)

---

![fit](https://developer.apple.com/library/archive/featuredarticles/ViewControllerPGforiPhoneOS/Art/VCPG_Adaptive-Model_13-1_2x.png)

[.background-color: #FFFFFF]


---

# Traits:

- `horizontalSizeClass`
- `verticalSizeClass`
- `displayScale`
- `displayGamut`
- `forceTouchCapability`
- `preferredContentSizeCategory`
- `userInterfaceIdiom`

Use these traits to customize your layout.

^ For example, horizontal size class to stack views horizontally or vertically, vertical size class to make the content fit the screen without scrolling, displayScale for pixel-level decisions, interfaceidiom to figure out if you're on a TV or Carplay

---

# Traits:

- All these properties are defined in `UITraitCollection`.

---

# Traits:

- The `traitCollection` can be accessed on any class that conforms to:

```objc

@protocol UITraitEnvironment <NSObject>

@property UITraitCollection *traitCollection;

- (void)traitCollectionDidChange:(UITraitCollection *)previousTraitCollection;

@end
```

---

#Traits:

All these classes conform to `UITraitEnvironment`

- `UIScreen`
- `UIWindow`
- `UIViewController`
- `UIPresentationController`
- `UIView`

---

#Traits:

- The traits should have a correct value when added to the view/viewController hierarchy, not at init time.

- You can override them using `setOverrideTraitCollection:forChildViewController:`

---

#Size Classes:

They are just one of the many traits available

---

#Size Classes:

```objc
typedef NS_ENUM(NSInteger, UIUserInterfaceSizeClass) {
    UIUserInterfaceSizeClassUnspecified = 0,
    UIUserInterfaceSizeClassCompact     = 1,
    UIUserInterfaceSizeClassRegular     = 2,
} NS_ENUM_AVAILABLE_IOS(8_0);
```
---

#Some recomendations:

---

#Never assume that a size class corresponds to the specific width or height of a view.

---

#Avoid using idiom information to make decisions about the layout or content of your interface.

---

# Be prepared for the `unspecified` value. 

---

# Override the traits if you want a different layout than what is provided by the system.

---

![inline](https://useyourloaf.com/assets/images/2015/SizeClasses-iPhone.png)

---

![inline](https://useyourloaf.com/assets/images/2015/SizeClasses-iPhone6Plus.png)

---

![inline](https://www.bignerdranch.com/assets/img/blog/2017/10/iphone7plus_landscape_mail.png)

[.background-color: #FFFFFF]

---

![inline](https://useyourloaf.com/assets/images/2015/SizeClasses-iPad.png)

---

#Adapting to change

Changes to trait collection can happen because:

- Was overriden by parent viewController
- Rotation
- Multitasking

---

![fit](https://i.imgur.com/FyaIlkB.png)

[.background-color: #FFFFFF]

---

![fit](https://i.imgur.com/DTBH9b5.png)

[.background-color: #FFFFFF]

---

#[fit]Demo

^Show how `traitCollectionDidChange` is propagated to a UIView and to UIViewController.
Then go to GIFWallet and add two or three columns to GIFWalletVC


---

#ViewController Presentation

---

#Navigation

- Don't explicitly use `UINavigationController`'s methods. [^1]
	- Instead use `showViewController:sender:` and `showDetailViewController:sender:`
	- These allow customization of the presentation for a given context overriding `targetViewControllerForAction:sender:`

[^1]: [targetViewControllerForAction:sender: is smarter than it seems](http://optshiftk.com/2014/08/targetviewcontrollerforaction-sender-smarter-than-it-seems/)


---
#Navigation

- Adopt `UISplitViewController` when appropiate.
	- Allows better integration when shown on `.regular` horizontal size classes. 

---

#Presentation

---

![inline](https://developer.apple.com/library/content/featuredarticles/ViewControllerPGforiPhoneOS/Art/VCPG_PresentationStyles%20_fig_8-1_2x.png)

[.background-color: #FFFFFF]

---

#Popovers

![inline](https://developer.apple.com/library/content/featuredarticles/ViewControllerPGforiPhoneOS/Art/VCPG_popover-in-regular-and-compact-views_13_3_2x.png)

---

#Popovers


- Use `UIModalPresentationPopover`
- Customize through `UIPopoverPresentationController`
	- Origin of the popover
	- `preferredContentSize`
	- Margins

---

#Recomendations:

---

#Work closely with your designer

---

#Think in terms of proportions, not pixels/points

---

#It's easier to test transitions on device