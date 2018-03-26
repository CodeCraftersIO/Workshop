Start at 2815a7f:

- Let's add the GIFImageView
	- Notice that if you forget translatesAutoresizingMaskIntoConstraints shit happens. 
	- Create `addAutolayoutView`
- Create all the view using just constraints (7d0e691)
- Rewrite details using StackView (0f3695b)
- Place inside scrollView using UIView as containerView (1102fe3)
	- Use stackView as containerView
- Refactor UILabel creation (58eaf7a)
- Fix imageView aspect ratio (f0b9bc3)
- Add rotation support (d9b78dd)
# 