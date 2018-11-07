**Start**

- Go to commit `68aaf45`
- Show the code around:
	- Explain the addition of GenerateScreenshots.sh
	- Added test for GIFCreateViewController

- Make the students add a SAVE button anchored to the bottom	- Commit (d4e3f5f)

**Create SaveButton.swift**

```swift
import UIKit

final class SaveButton: UIButton {
    
    private enum Constants {
        static let padding: CGFloat = 10
    }
    
    init() {
        super.init(frame: .zero)
        setup()
    }
    
    required init?(coder aDecoder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
    
    private func setup() {
        backgroundColor = UIColor.GifWallet.brand
        setTitle("Save", for: .normal)
    }
    
    override func safeAreaInsetsDidChange() {
        super.safeAreaInsetsDidChange()
        self.titleEdgeInsets = UIEdgeInsets.init(
            top: self.safeAreaInsets.bottom / 2 ,
            left: 0,
            bottom: 0,
            right: 0
        )
        self.contentEdgeInsets = UIEdgeInsets.init(
            top: self.safeAreaInsets.top + Constants.padding,
            left: self.safeAreaInsets.left + Constants.padding,
            bottom: self.safeAreaInsets.bottom + Constants.padding,
            right: self.safeAreaInsets.right + Constants.padding
        )
    }

}

```

**In GIFCreateViewController.swift**

```swift
private let saveButton = SaveButton()
```

```swift
private func layout() {
    view.addSubview(tableView)
    tableView.pinToSuperviewSafeLayoutEdges()

    saveButton.translatesAutoresizingMaskIntoConstraints = false
    view.addSubview(saveButton)
    NSLayoutConstraint.activate([
        saveButton.leftAnchor.constraint(equalTo: self.view.leftAnchor),
        saveButton.rightAnchor.constraint(equalTo: self.view.rightAnchor),
        saveButton.bottomAnchor.constraint(equalTo: self.view.bottomAnchor),
        ])
}
```

- Show the result at `dd15760`
- Add the tableview and the datasource, at `36ba294`
	- Show that the insets on the top are OK, but not so much at the bottom
	- Show that the insets at the bottom are those respecting safe area.
 
```swift
private func setupTableView() {
        view.addAutolayoutView(tableView)
        tableView.pinToSuperview()
        tableView.dataSource = self
        tableView.register(UITableViewCell.self, forCellReuseIdentifier: "ReuseID")
    }
```

- Add the insets to account for the height of the bottom button
	- override viewDidLayoutSubviews

```swift
override func viewDidLayoutSubviews() {
    super.viewDidLayoutSubviews()
    tableView.contentInset = UIEdgeInsets(
        top: 0,
        left: 0,
        bottom: saveButton.frame.size.height,
        right: 0
    )
}
```
in setup

```swift
saveButton.addTarget(self, action: #selector(onSave), for: .touchDown)
```

and

```swift
@objc func onSave() {

}
```

- Dump all the view code
	- Commit (edabd3c)
	- We've created
		- FormTableViewCell
		- (Text/GIF/Tags)InputView
		- RegisterCells with Generics
		- Added many things to GIFCreateVC


- Create GIFCreateFormValidator and show why it has to be in Kit and why it holds the `Form` in a class
	- Create all the rules and the errors
	- Unit test it
	- (95bd6a8)

We code this...

```swift
import Foundation

public final class GIFCreateFormValidator {

    public init() {

    }
    
    public let form = Form()

    public class Form {
        public var title: String?
        public var subtitle: String?
        public var gifURL: URL?
        public var tags: [String]?

        init() { }
    }

    public enum ValidationResult {
        case ok
        case error(Set<ValidationError>)
    }

    public enum ValidationError: Error {
        case titleNotProvided
        case subtitleNotProvided
        case gifNotProvided
        case tagsNotProvided
    }

    public func validateForm() -> ValidationResult {

        func isEmpty(_ string: String?) -> Bool {
            return string == nil || string!.isEmpty
        }

        var errors = Set<ValidationError>()

        if isEmpty(form.title) {
            errors.insert(.titleNotProvided)
        }

        if isEmpty(form.subtitle) {
            errors.insert(.subtitleNotProvided)
        }

        if form.gifURL == nil {
            errors.insert(.gifNotProvided)
        }

        if form.tags == nil || form.tags!.isEmpty {
            errors.insert(.tagsNotProvided)
        }

        if errors.isEmpty {
            return .ok
        } else {
            return .error(errors)
        }
    }
}
```

...and the tests

```swift
import XCTest
import GifWalletKit

class GIFFormValidatorTests: XCTestCase {

    var formValidator: GIFCreateFormValidator!

    override func setUp() {
        super.setUp()
        formValidator = GIFCreateFormValidator()
    }

    func testValidateInvalidResults() {

        let title: String? = nil
        let subtitle = ""
        let gifURL = URL(string: "invalid URL")
        let tags = [String]()

        formValidator.form.title = title
        formValidator.form.subtitle = subtitle
        formValidator.form.gifURL = gifURL
        formValidator.form.tags = tags

        let result = formValidator.validateForm()

        switch result {
        case .ok:
            XCTFail("Result should be an error")
        case .error(let errors):
            XCTAssert(errors.count == 4, "Validation must contain exactly 4 errors")
            XCTAssert(errors.contains(.titleNotProvided), "Validation result must contain error: title not provided")
            XCTAssert(errors.contains(.subtitleNotProvided), "Validation result must contain error: subtitle not provided")
            XCTAssert(errors.contains(.gifNotProvided), "Validation result must contain error: gif not provided")
            XCTAssert(errors.contains(.tagsNotProvided), "Validation result must contain error: tags not provided")
        }
    }

    func testValidateValidResults() {

        let title: String? = "A GIF Title"
        let subtitle = "A GIF Subtitle"
        let gifURL = URL(string: "https://media0.giphy.com/media/8752sSo2HbPqE7MN03/giphy.gif")
        let tags = [ "funny" ]

        formValidator.form.title = title
        formValidator.form.subtitle = subtitle
        formValidator.form.gifURL = gifURL
        formValidator.form.tags = tags

        switch formValidator.validateForm() {
        case .ok:
            break
        default:
            XCTFail()
        }
    }

}
```

- Create required sections
	- Show why it must be in the kit (in order for it to be parametrized in case of country/whatever business decision)

```swift
requiredSections = [
    .gifURL,
    .title,
    .subtitle,
    .tags
]
...
public let requiredSections: [Section]

public enum Section {
    case title
    case subtitle
    case gifURL
    case tags
}

```
- 
	- Unit test it

```swift
    func testRequiredSections() {
        XCTAssert(formValidator.requiredSections.count == 4)
        XCTAssert(formValidator.requiredSections[0] == .gifURL)
        XCTAssert(formValidator.requiredSections[1] == .title)
        XCTAssert(formValidator.requiredSections[2] == .subtitle)
        XCTAssert(formValidator.requiredSections[3] == .tags)
    }
```

- 
	- Now hook it to the VC and the tableViewDataSource
	- Show the awesome result (8699ffd)

...in cellForItemAt

```swift
let section = formValidator.requiredSections[indexPath.section]
let tableViewCell: UITableViewCell = {
    switch section {
    case .gifURL:
        let cell: FormTableViewCell<GIFInputView> = tableView.dequeueReusableCell(indexPath: indexPath)
        cell.configureFor(vm: FormTableViewCell<GIFInputView>.VM(inputVM: GIFInputView.VM(id: "hello", url: URL(string: "https://media0.giphy.com/media/8752sSo2HbPqE7MN03/giphy.gif")!), showsWarning: true))
        cell.formInputView.delegate = self
        return cell
    case .title:
        let cell: FormTableViewCell<TextInputView> = tableView.dequeueReusableCell(indexPath: indexPath)
        cell.configureFor(vm: FormTableViewCell<TextInputView>.VM(inputVM: TextInputView.VM(text: nil), showsWarning: true))
        cell.formInputView.placeholder = "Enter the Title"
        cell.formInputView.delegate = self
        return cell
    case .subtitle:
        let cell: FormTableViewCell<TextInputView> = tableView.dequeueReusableCell(indexPath: indexPath)
        cell.configureFor(vm: FormTableViewCell<TextInputView>.VM(inputVM: TextInputView.VM(text: nil), showsWarning: true))
        cell.formInputView.placeholder = "Enter the Subtitle"
        cell.formInputView.delegate = self
        return cell
    case .tags:
        let cell: FormTableViewCell<TagsInputView> = tableView.dequeueReusableCell(indexPath: indexPath)
        cell.configureFor(vm: FormTableViewCell<TagsInputView>.VM(inputVM: TagsInputView.VM(tags: ["Studio", "iOS", "Funny"]), showsWarning: true))
        cell.formInputView.delegate = self
        return cell
    }
}()
return tableViewCell
```

- Show the layout issue when turning off the warnings, and fix it (it's the stackView)
	- (606a5e0)

```swift
cell.configureFor(vm: FormTableViewCell<GIFInputView>.VM(inputVM: GIFInputView.VM(id: nil, url: nil), showsWarning: false))
...                
cell.configureFor(vm: FormTableViewCell<TextInputView>.VM(inputVM: TextInputView.VM(text: nil), showsWarning: false))
...
cell.configureFor(vm: FormTableViewCell<TagsInputView>.VM(inputVM: TagsInputView.VM(tags: []), showsWarning: false))
```

```swift
FormTableViewCell.swift
//At stackview
stackView.distribution = .fill
...
//At warningIcon
let widthConstraint = imageView.widthAnchor.constraint(equalToConstant: image.size.width)
widthConstraint.priority = .defaultLow
widthConstraint.isActive = true
```

	
- Show that form entry sucks because the keyboard is in the way
	- Add IQKeyboardHandler (61c107c)
	
- Start hooking up the cells to the FormValidator
	- Show how to use tableView.reloadSections

WARNING: I added a line here wrt repo (validationErrors)

```swift
guard let textInput = TextInputTags(rawValue: textInputView.tag) else { return }
let sectionToReload: GIFCreateFormValidator.Section
switch textInput {
case .title:
    formValidator.form.title = text
    sectionToReload = .title
case .subtitle:
    formValidator.form.subtitle = text
    sectionToReload = .subtitle
}
if case .error(let errors) = self.formValidator.validateForm() {
    lastValidationErrors = errors
}
guard let sectionToReloadIndex = formValidator.requiredSections.index(of: sectionToReload) else {
    return
}
tableView.performBatchUpdates({
    tableView.reloadSections([sectionToReloadIndex], with: .automatic)
}, completion: nil)
```

- 
	- Hook title, subtitle and tags up and error handling

```swift
...
let shouldShowWarning = lastValidationErrors.contains(.titleNotProvided)
...
cell.configureFor(vm: FormTableViewCell<TextInputView>.VM(inputVM: TextInputView.VM(text: self.formValidator.form.title), showsWarning: shouldShowWarning))
...
cell.formInputView.tag = TextInputTags.title.rawValue

```
	
- 
	- Hook save with validation

```swift
        switch self.formValidator.validateForm() {
        case .error(let errors):
            self.lastValidationErrors = errors
            self.tableView.reloadData()
        case .ok:
            self.dismissViewController()
        }
```

- 
	- (2c60644)
- Create and hook up GIFSearchVC 	
	- First, promote GIFCollectionView to it's own class 
	- Then, just drop it and walkthrough it
	- (e54fc43)
- (f3ab441)
	- Show merge bug that appears when calling twice setupTableView()
	- Add snapshot testing 