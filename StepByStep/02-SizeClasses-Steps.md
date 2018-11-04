
#Autolayout Tests
- Debug how `traitCollectionDidChange` is propagated (4aadb2f)
	- Show how it is called on the ViewController and also on UIView subclass
	- Propose this problem https://i.imgur.com/klNG51U.png 

- Add a share button to DetailVC and use PopoverPresentation (3f29cb1)

```swift
    @objc func shareGIF() {
        guard
            let vm = self.vm,
            let path = SDImageCache.shared().defaultCachePath(forKey: vm.url.absoluteString) else { return }

        let url = URL(fileURLWithPath: path)

        guard let data = try? Data(contentsOf: url) else { return }

        let controller = UIActivityViewController(activityItems: [data], applicationActivities: nil)
        
        // Specify the anchor point for the popover.
        controller.popoverPresentationController?.barButtonItem = self.navigationItem.rightBarButtonItem
        present(controller, animated: true, completion: nil)
    }
```

- Hide away any reference to UINavigationController using `self.show(vc, sender: nil)` and create a dismiss method as well (20e3860)

```swift
extension UIViewController {
    //Based on https://stackoverflow.com/a/28158013/1152289
    @objc func dismissViewController(sender: Any?) {
        guard let presentingVC = targetViewController(forAction: #selector(dismissViewController(sender:)), sender: sender) else { return }
        presentingVC.dismissViewController(sender: sender)
    }
}

extension UINavigationController {
    @objc
    override func dismissViewController(sender: Any?) {
        self.popViewController(animated: true)
    }
}
```

# Modal presentation

- Let's create GIFWalletViewController
- Add a + Button
	- Present GIFFormVC (`9faf793`)

```swift
class GIFCreateViewController: UIViewController {

    private let tableView = UITableView(frame: .zero, style: .grouped)

    init() {
        super.init(nibName: nil, bundle: nil)
    }
    
    required init?(coder aDecoder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()
        assert(self.navigationController != nil)
        navigationItem.leftBarButtonItem = UIBarButtonItem(barButtonSystemItem: UIBarButtonSystemItem.cancel, target: self, action: #selector(dismissViewController))
        setup()
    }

    @objc func dismissViewController() {
        self.dismiss(animated: true, completion: nil)
    }

    //MARK: Private

    private func setup() {
        setupTableView()
    }

    private func setupTableView() {
        view.addSubview(tableView)
        tableView.pinToSuperviewSafeLayoutEdges()
    }
}

...

navigationItem.rightBarButtonItem = UIBarButtonItem(barButtonSystemItem: .add, target: self, action: #selector(addNewGif))

    @objc func addNewGif() {
        let createVC = GIFCreateViewController()
        let navController = UINavigationController(rootViewController: createVC)
        navController.modalPresentationStyle = .formSheet
        self.present(navController, animated: true, completion: nil)
    }

```
	
- Note that screenshot tests fail now, fix them

- Show how the initialization is problematic since there's a lot of configuration to make it modally (it has to be set on the navController)
	- Add Factory subtype and make the inits of the VC private (78e0cd0)

	```
	extension GIFCreateViewController {
    enum Factory {
        static func viewController() -> UIViewController {
            let createVC = GIFCreateViewController()
            let navController = UINavigationController(rootViewController: createVC)
            navController.modalPresentationStyle = .formSheet
            return navController
        }
    }
```

- Create a Script to automate the screenshot creation