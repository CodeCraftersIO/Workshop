
- Debug how traitCollectionDidChange is propagated (4aadb2f)
	- Show how it is called on the ViewController and also on UIView subclass
	- Propose this problem https://i.imgur.com/klNG51U.png 
- Add more than one column on iPad using `willTransition(to newCollection: UITraitCollection, with coordinator: UIViewControllerTransitionCoordinator)` (9b94378)
	- Show how it looks like crap on iPad landscape
- Show that it's impossible using Apple's API to take into account both newSize and newCollection. Create wrapper to workaround it (45ebea6)
- Add a share button to DetailVC and use PopoverPresentation (2161086)
- Hide away any reference to UINavigationController using `self.show(vc, sender: nil)` and create a dismiss method as well (20e3860)  