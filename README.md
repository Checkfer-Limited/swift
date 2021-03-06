# Checkfer Swift Style Guide

## Goals

Following this style guide should:

* Make it easier to read and begin understanding unfamiliar code.
* Make code easier to maintain.
* Reduce simple programmer errors.
* Reduce cognitive load while coding.
* Keep discussions on diffs focused on the code's logic rather than its style.

Note that brevity is not a primary goal. Code should be made more concise only if other good code qualities (such as readability, simplicity, and clarity) remain equal or are improved.

## Guiding Tenets

* This guide is in addition to the official [Swift API Design Guidelines](https://swift.org/documentation/api-design-guidelines/). These rules should not contradict that document.
* These rules should not fight Xcode's <kbd>^</kbd> + <kbd>I</kbd> indentation behavior.
* We strive to make every rule lintable:
  * If a rule changes the format of the code, it needs to be able to be reformatted automatically (either using [SwiftLint](https://github.com/realm/SwiftLint) autocorrect or [SwiftFormat](https://github.com/nicklockwood/SwiftFormat)).
  * For rules that don't directly change the format of the code, we should have a lint rule that throws a warning.
  * Exceptions to these rules should be rare and heavily justified.

## Table of Contents

1. [Project Guidelines](#project-guidelines)
1. [Xcode Formatting](#xcode-formatting)
1. [Naming](#naming)
1. [Style](#style)
    1. [Functions](#functions)
    1. [Closures](#closures)
    1. [Operators](#operators)
    1. [Formatting/Structure](#formatting/structure)
1. [Patterns](#patterns)
1. [File Organization](#file-organization)
1. [Objective-C Interoperability](#objective-c-interoperability)
1. [Contributors](#contributors)
1. [Amendments](#amendments)


##  Xcode Guidelines
We strive to ensure that working as a team is made much easier by following a few guidelines to back reviewing, merging and working together a lesser task.

* Ensure when working on features that Stroyboards are not used, we do not use storyboards to reduce conflicting codebases when working simultaneously.
* When using third party libraries we use the Cocoapod Dependancy Manager, and this should be setup according to the spec provided by Cocoa. 
  * Ensure that the /Pod folder is not commited to the repository, there should be an entry in the gitignore file.
  * If at any point a library needs to be locked, add a comment to the project readme, format: podname / version locked / date / reason


## Xcode Formatting

_You can enable the following settings in Xcode by running [this script](resources/xcode_settings.bash), e.g. as part of a "Run Script" build phase._

* <a id='column-width'></a>(<a href='#column-width'>link</a>) **Each line should have a maximum column width of 100 characters.**


  #### Why?
  Due to larger screen sizes, we have opted to choose a page guide greater than 80

  

* <a id='spaces-over-tabs'></a>(<a href='#spaces-over-tabs'>link</a>) **Use 2 spaces to indent lines.**

* <a id='trailing-whitespace'></a>(<a href='#trailing-whitespace'>link</a>) **Trim trailing whitespace in all lines.** [![SwiftFormat: trailingSpace](https://img.shields.io/badge/SwiftFormat-trailingSpace-7B0051.svg)](https://github.com/nicklockwood/SwiftFormat/blob/master/Rules.md#trailingSpace)

**[⬆ back to top](#table-of-contents)**

# Naming

<a id='prefer-pure-swift-classes'></a>(<a href='#prefer-pure-swift-classes'>quick-link</a>) 

* **Use PascalCase for type and protocol names, and lowerCamelCase for everything else.** [![SwiftLint: type_name](https://img.shields.io/badge/SwiftLint-type__name-007A87.svg)](https://github.com/realm/SwiftLint/blob/master/Rules.md#type-name)

  	```swift
	protocol SpaceThing {
  		// ...
  	}
  
	class SpaceFleet: SpaceThing {
	  
		enum Formation {
			// ...
		}
		  
		class Spaceship {
			// ...
		}
		  
		var ships: [Spaceship] = []
		static let worldName: String = "Earth"
		  
		func addShip(_ ship: Spaceship) {
			// ...
		}
  
	}
  
	let myFleet = SpaceFleet()
	```

_Exception: You may prefix a private property with an underscore if it is backing an identically-named property or method with a higher access level_

#### Why?
There are specific scenarios where a backing a property or method could be easier to read than using a more descriptive name.

  - Type erasure

	```swift
	public final class AnyRequester<ModelType>: Requester {

	public init<T: Requester>(_ requester: T) where T.ModelType == ModelType {
		_executeRequest = requester.executeRequest
	}

	@discardableResult
	public func executeRequest(
		_ request: URLRequest,
		onSuccess: @escaping (ModelType, Bool) -> Void,
		onFailure: @escaping (Error) -> Void) -> URLSessionCancellable {
			return _executeRequest(request, session, parser, onSuccess, onFailure)
	}

	private let _executeRequest: (
		URLRequest,
		@escaping (ModelType, Bool) -> Void,
		@escaping (NSError) -> Void) -> URLSessionCancellable {
		
	}
		
	```

  - Backing a less specific type with a more specific type

	```swift
	final class ExperiencesViewController: UIViewController {
		// We can't name this view since UIViewController has a view: UIView property.
		private lazy var _view = CustomView()
		
		override func loadView() {
			self.view = _view
		}
		
	}
	```

* **Name booleans like `isSpaceship`, `hasSpacesuit`, etc.** This makes it clear that they are booleans and not other types.

* **Name outlets like `lblTitle`, `btnAction`, `txtEmail`, etc.** This helps to keep consistency throughout the codebase, the beginning should be a 2/3 character identifier and the followed by something sensible and descriptive. Here are some examples:
  
  ```swift
  @IBOutlet weak var lblTitle: UILabel
  @IBOutlet weak var btnAction: UIButton
  @IBOutlet weak var txtEmailInput: UITextField
  @IBOutlet weak var ctlOption: UIControl
  ```
  Views, `UIView`, `UITableView`... should follow `<ViewName>View`:
  
  ```swift
  @IBOutlet weak var tableView: UITableView
  @IBOutlet weak var collectionView: UICollectionView
  ```
  
  Functions should follow Swift standards and should not contain `with` before the parameter, `func reload(withSection section: Int? = nil)` this is an objc trait. Functions should follow this standard:

  `func reload(section: Int? = nil)`

* **Acronyms in names (e.g. `URL`) should be all-caps except when it’s the start of a name that would otherwise be lowerCamelCase, in which case it should be uniformly lower-cased.**

	*Not preferred:*

	```swift
	class UrlValidator {
	  
		func isValidUrl(_ URL: URL) -> Bool {
			// ...
		}
		    
		func isProfileUrl(_ URL: URL, for userId: String) -> Bool {
			// ...
		}
	  
	}
	  
	let URLValidator = UrlValidator()
	let isProfile = URLValidator.isProfileUrl(URLToTest, userId: IDOfUser)
	```

	*Preferred:*

	```swift
	class URLValidator {
	  
		func isValidURL(_ url: URL) -> Bool {
			// ...
		}
		  
		func isProfileURL(_ url: URL, for userID: String) -> Bool {
			// ...
		}
	  
	}
	  
	let urlValidator = URLValidator()
	let isProfile = urlValidator.isProfileUrl(urlToTest, userID: idOfUser)
	```

* **Names should be written with their most general part first and their most specific part last.** The meaning of "most general" depends on context, but should roughly mean "that which most helps you narrow down your search for the item you're looking for." Most importantly, be consistent with how you order the parts of your name.

	*Not preferred:*

	```swift
	let rightTitleMargin: CGFloat
	let leftTitleMargin: CGFloat
	let bodyRightMargin: CGFloat
	let bodyLeftMargin: CGFloat
	```

	*Preferred:*

	```swift
	let titleMarginRight: CGFloat
	let titleMarginLeft: CGFloat
	let bodyMarginRight: CGFloat
	let bodyMarginLeft: CGFloat
	```

* **Include a hint about type in a name if it would otherwise be ambiguous.**

	*Not preferred:*

	```swift
	let title: String
	let cancel: UIButton
	```

	*Preferred:*

	```swift
	let titleText: String
	let cancelButton: UIButton
	```

* **Event-handling functions should be named like past-tense sentences.** The subject can be omitted if it's not needed for clarity.

	*Not preferred:*

	```swift
	class ExperiencesViewController {
	  
		private func handleBookButtonTap() {
  			// ...
		}
		  
		private func modelChanged() {
  			// ...
		}
	    
	}
	```

	*Preferred:*
	
	```swift
	class ExperiencesViewController {
	  
		private func didTapBookButton() {
  			// ...
		}
		  
		private func modelDidChange() {
  			// ...
		}
	    
	}
	```

* **Avoid Objective-C-style acronym prefixes.** This is no longer needed to avoid naming conflicts in Swift.

	*Not preferred:*

	```swift
	class AIRAccount {
		// ...
	}
	```

	*Preferred:*
	
	```swift
	class Account {
		// ...
	}
	```

* **Avoid `*Controller` in names of classes that aren't view controllers.**

  Controller is an overloaded suffix that doesn't provide information about the responsibilities of the class.

**[⬆ back to top](#table-of-contents)**

# Style

<a id='prefer-pure-swift-classes'></a>(<a href='#prefer-pure-swift-classes'>quick-link</a>) 

* **Don't include types where they can be easily inferred.**

	*Not preferred:*
	
	```swift
	let host: Host = Host()
	```
	
	*Preferred:*
	
	```swift
	let host = Host()
	```

  
	*Not preferred:*
	
	```swift
	enum Direction {
		case left
		case right
	}
	  
	func someDirection() -> Direction {
		return Direction.left
	}
	```

	*Preferred:*
	
	```swift
	enum Direction {
		case left
		case right
	}
	  
	func someDirection() -> Direction {
		return .left
	}
	```

* **Don't use `self` unless it's necessary for disambiguation or required by the language.** [![SwiftFormat: redundantSelf](https://img.shields.io/badge/SwiftFormat-redundantSelf-7B0051.svg)](https://github.com/nicklockwood/SwiftFormat/blob/master/Rules.md#redundantSelf)

	*Not preferred:*
	
	```swift
	final class Listing {
	  
		init(capacity: Int, allowsPets: Bool) {
			self.capacity = capacity
			self.isFamilyFriendly = !allowsPets // `self.` not required here
		}
		  
		private let isFamilyFriendly: Bool
		private var capacity: Int
		  
		private func increaseCapacity(by amount: Int) {
			self.capacity += amount
			self.save()
		}
    
	}
	```

	*Preferred:*
	
	```swift
	final class Listing {
	  
		init(capacity: Int, allowsPets: Bool) {
			self.capacity = capacity
			isFamilyFriendly = !allowsPets
		}
		  
		private let isFamilyFriendly: Bool
		private var capacity: Int
		  
		private func increaseCapacity(by amount: Int) {
			capacity += amount
			save()
		}
    
	}
	```

* **Bind to `self` when upgrading from a weak reference.** [![SwiftFormat: strongifiedSelf](https://img.shields.io/badge/SwiftFormat-strongifiedSelf-7B0051.svg)](https://github.com/nicklockwood/SwiftFormat/blob/master/Rules.md#strongifiedSelf)

	*Not preferred:*
	
	```swift
	class MyClass {
	  
		func request(completion: () -> Void) {
			API.request() { [weak self] response in
				guard let strongSelf = self else { return }
				// Do work
				completion()
			}
		}
	    
	}
	```

	*Preferred:*
	
	```swift
	class MyClass {
	  
		func request(completion: () -> Void) {
			API.request() { [weak self] response in
				guard let self = self else { return }
				// Do work
				completion()
			}
		}
	    
	}
	```

* **Add a trailing comma on the last element of a multi-line array.** [![SwiftFormat: trailingCommas](https://img.shields.io/badge/SwiftFormat-trailingCommas-7B0051.svg)](https://github.com/nicklockwood/SwiftFormat/blob/master/Rules.md#trailingCommas)

	*Not preferred:*
	
	```swift
	let rowContent = [
		listingUrgencyDatesRowContent(),
		listingUrgencyBookedRowContent(),
		listingUrgencyBookedShortRowContent()
	]
	```

	*Preferred:*
	
	```swift
	let rowContent = [
		listingUrgencyDatesRowContent(),
		listingUrgencyBookedRowContent(),
		listingUrgencyBookedShortRowContent(),
	]
	```

* **Name members of tuples for extra clarity.** Rule of thumb: if you've got more than 3 fields, you should probably be using a struct.

	*Not preferred:*
	
	```swift
	func whatever() -> (Int, Int) {
		return (4, 4)
	}
	
	let thing = whatever()
	print(thing.0)
	```

	*Preferred:*
	
	```swift
	func whatever() -> (x: Int, y: Int) {
		return (x: 4, y: 4)
	}
	  
	// THIS IS ALSO OKAY
	func whatever2() -> (x: Int, y: Int) {
		let x = 4
		let y = 4
		return (x, y)
	}
	  
	let coord = whatever()
	coord.x
	coord.y
	```

  

* **Place the colon immediately after an identifier, followed by a space.** [![SwiftLint: colon](https://img.shields.io/badge/SwiftLint-colon-007A87.svg)](https://github.com/realm/SwiftLint/blob/master/Rules.md#colon)

	*Not preferred:*
	
	```swift
	var something : Double = 0
	```
	
	*Preferred:*
	
	```swift
	var something: Double = 0
	```
	
	  
	
	*Not preferred:*
	
	```swift
	class MyClass : SuperClass {
		// ...
	}
	```
	
	*Preferred:*
	
	```swift
	class MyClass: SuperClass {
		// ...
	}
	```


	*Not preferred:*
	
	```swift
	var dict = [KeyType:ValueType]()
	var dict = [KeyType : ValueType]()
	```
	
	*Preferred:*
	
	```swift
	var dict = [KeyType: ValueType]()
	```

* **Place a space on either side of a return arrow for readability.** [![SwiftLint: return_arrow_whitespace](https://img.shields.io/badge/SwiftLint-return__arrow__whitespace-007A87.svg)](https://github.com/realm/SwiftLint/blob/master/Rules.md#returning-whitespace)

	*Not preferred:*
		
	```swift
	func doSomething()->String {
		// ...
	}
	```
		
	*Preferred:*
		
	```swift
	func doSomething() -> String {
		// ...
	}
	```
		
	  
		
	*Not preferred:*
		
	```swift
	func doSomething(completion: ()->Void) {
		// ...
	}
	```
		
	*Preferred:*
		
	```swift
	func doSomething(completion: () -> Void) {
		// ...
	}
	``` 

* **Omit unnecessary parentheses.** [![SwiftFormat: redundantParens](https://img.shields.io/badge/SwiftFormat-redundantParens-7B0051.svg)](https://github.com/nicklockwood/SwiftFormat/blob/master/Rules.md#redundantParens)

	*Not preferred:*
	
	```swift
	if (userCount > 0) { ... }
	switch (someValue) { ... }
	let evens = userCounts.filter { (number) in number % 2 == 0 }
	let squares = userCounts.map() { $0 * $0 }
	```
	
	*Preferred:*
	
	```swift
	if userCount > 0 { ... }
	switch someValue { ... }
	let evens = userCounts.filter { number in number % 2 == 0 }
	let squares = userCounts.map { $0 * $0 }
	```

* **Omit enum associated values from case statements when all arguments are unlabeled.** [![SwiftLint: empty_enum_arguments](https://img.shields.io/badge/SwiftLint-empty__enum__arguments-007A87.svg)](https://github.com/realm/SwiftLint/blob/master/Rules.md#empty-enum-arguments)

	*Not preferred:*
	
	```swift
	if case .done(_) = result { ... }
	  
	switch animal {
	case .dog(_, _, _):
	...
	}
	```
	
	*Preferred:*
	
	```swift
	if case .done = result { ... }
	  
	switch animal {
	case .dog:
	...
	}
	```

* **Use constructors instead of Make() functions for NSRange and others.** [![SwiftLint: legacy_constructor](https://img.shields.io/badge/SwiftLint-legacy__constructor-007A87.svg)](https://github.com/realm/SwiftLint/blob/master/Rules.md#legacy-constructor)

	*Not preferred:*
	  
	```swift
	let range = NSMakeRange(10, 5)
	```
		
	*Preferred:*
		
	```swift
	let range = NSRange(location: 10, length: 5)
	```

## Functions

<a id='prefer-pure-swift-classes'></a>(<a href='#prefer-pure-swift-classes'>quick-link</a>) 

* **Omit `Void` return types from function definitions.** [![SwiftLint: redundant_void_return](https://img.shields.io/badge/SwiftLint-redundant__void__return-007A87.svg)](https://github.com/realm/SwiftLint/blob/master/Rules.md#redundant-void-return)

	*Not preferred:*
	  
	```swift
	func doSomething() -> Void {
		...
	}
	```
		
	*Preferred:*
		
	```swift
	func doSomething() {
		...
	}
	```

## Closures

<a id='prefer-pure-swift-classes'></a>(<a href='#prefer-pure-swift-classes'>quick-link</a>) 

* **Favor `Void` return types over `()` in closure declarations.** If you must specify a `Void` return type in a function declaration, use `Void` rather than `()` to improve readability. [![SwiftLint: void_return](https://img.shields.io/badge/SwiftLint-void__return-007A87.svg)](https://github.com/realm/SwiftLint/blob/master/Rules.md#void-return)

	*Not preferred:*
	
	```swift
	func method(completion: () -> ()) {
		...
	}
	```
	
	*Preferred:*
	
	```swift
	func method(completion: () -> Void) {
		...
	}
	```

* **Name unused closure parameters as underscores (`_`).** [![SwiftLint: unused_closure_parameter](https://img.shields.io/badge/SwiftLint-unused__closure__parameter-007A87.svg)](https://github.com/realm/SwiftLint/blob/master/Rules.md#unused-closure-parameter)

    Naming unused closure parameters as underscores reduces the cognitive overhead required to read
    closures by making it obvious which parameters are used and which are unused.

	*Not preferred:*
	
	```swift
	someAsyncThing() { argument1, argument2, argument3 in
		print(argument3)
	}
	```
	
	*Preferred:*
	
	```swift
	someAsyncThing() { _, _, argument3 in
		print(argument3)
	}
	```

* <a id='closure-brace-spacing'></a>(<a href='#closure-brace-spacing'>link</a>) **Single-line closures should have a space inside each brace.** [![SwiftLint: closure_spacing](https://img.shields.io/badge/SwiftLint-closure__spacing-007A87.svg)](https://github.com/realm/SwiftLint/blob/master/Rules.md#closure-spacing)
	
	*Not preferred:*
		
	```swift
	let evenSquares = numbers.filter {$0 % 2 == 0}.map {  $0 * $0  }
	```
		
	*Preferred:*
		
	```swift
	let evenSquares = numbers.filter { $0 % 2 == 0 }.map { $0 * $0 }
	```

## Operators

<a id='prefer-pure-swift-classes'></a>(<a href='#prefer-pure-swift-classes'>quick-link</a>) 

* **Infix operators should have a single space on either side.** Prefer parenthesis to visually group statements with many operators rather than varying widths of whitespace. This rule does not apply to range operators (e.g. `1...3`) and postfix or prefix operators (e.g. `guest?` or `-1`). [![SwiftLint: operator_usage_whitespace](https://img.shields.io/badge/SwiftLint-operator__usage__whitespace-007A87.svg)](https://github.com/realm/SwiftLint/blob/master/Rules.md#operator-usage-whitespace)

	*Not preferred:*
		
	```swift
	let capacity = 1+2
	let capacity = currentCapacity   ?? 0
	let mask = (UIAccessibilityTraitButton|UIAccessibilityTraitSelected)
	let capacity=newCapacity
	let latitude = region.center.latitude - region.span.latitudeDelta/2.0
	```
		
	*Preferred:*
		
	```swift
	let capacity = 1 + 2
	let capacity = currentCapacity ?? 0
	let mask = (UIAccessibilityTraitButton | UIAccessibilityTraitSelected)
	let capacity = newCapacity
	let latitude = region.center.latitude - (region.span.latitudeDelta / 2.0)
	```
  
**[⬆ back to top](#table-of-contents)**

## Formatting/Structure

<a id='prefer-pure-swift-classes'></a>(<a href='#prefer-pure-swift-classes'>quick-link</a>) 

* **Marking indicators should be used on all swift class files.** There should always be a distiguishable sections to the code and Markings to clearly identify each section, this should not just be functional marking but visual. 

	*Not preferred:*
	  
	```swift
	class MyClass: UIViewController {
	  
		// MARK: Properties
		var someValue: Int = 0
	  
	}
	```
	
	*Preferred:*
	  
	```swift
	class MyClass: UIViewController {
	  
		//==================================================
		// MARK: Properties
		//==================================================
		
		var someValue: Int = 0
	  
	}
	```

- **Code structure should be in a standardised order** There should always be a distiguishable flow to the code, this will help create readable code. 
	
	*Not preferred:*
	  
	```swift
	class MyClass: NSObject {
	  
		// MARK: Helpers
		func someTask() {
		
		}
		  
		// MARK: Initialization
		init() {
			super.init()
		}
		  
		// MARK: Properties
		var someValue: Int = 0
	  
	}
	```
	
	*Preferred:*
	  
	```swift
	class MyClass: NSObject {
	  
		//==================================================
		// MARK: Properties
		//==================================================
		
		var someValue: Int = 0
		  
		//==================================================
		// MARK: Initialization
		//==================================================
		  
		init() {
			super.init()
		}
		  
		//==================================================
		// MARK: Helpers
		//==================================================
		
		func someTask() {
		
		}
	  
	}
	```

- **Enforced trailing new lines (code white-space)** There should be a necessary amount of new lines that creates better readability and structure to the code, spacing between markings, functions and helper code. White spacing should be used in single lines only. [![SwiftLint: implicitly_unwrapped_optional](https://img.shields.io/badge/SwiftLint-trailing__newline-007A87.svg)](https://realm.github.io/SwiftLint/trailing_newline.html)

	*Not preferred:*
	  
	```swift
	class MyClass: NSObject {
		//==================================================
		// MARK: Properties
		//==================================================
		var someValue: Int = 0
		//==================================================
		// MARK: Helpers
		//==================================================
		func someTask() {
		  
		}
	}
	```
		
	*Preferred:*
	
	```swift
	class MyClass: NSObject {
	  
		//==================================================
		// MARK: Properties
		//==================================================
		  
		var someValue: Int = 0
		  
		//==================================================
		// MARK: Helpers
		//==================================================
		  
		func someTask() {
	  
		}
	  
	}
	```

**[⬆ back to top](#table-of-contents)**

# Patterns

<a id='prefer-pure-swift-classes'></a>(<a href='#prefer-pure-swift-classes'>quick-link</a>) 

* **Prefer initializing properties at `init` time whenever possible, rather than using implicitly unwrapped optionals.**  A notable exception is UIViewController's `view` property. [![SwiftLint: implicitly_unwrapped_optional](https://img.shields.io/badge/SwiftLint-implicitly__unwrapped__optional-007A87.svg)](https://github.com/realm/SwiftLint/blob/master/Rules.md#implicitly-unwrapped-optional)

	*Not preferred:*
	
	```swift
	class MyClass: NSObject {
	  
		init() {
			super.init()
			someValue = 5
		}
		  
		var someValue: Int!
		    
	}
	```
	
	*Preferred:*
	
	```swift
	class MyClass: NSObject {
	  
		init() {
			someValue = 0
			super.init()
		}
		  
		var someValue: Int
	    
	}
	```

* **Avoid force casting objects.** Avoid force casting objects into an expected object, make use of `guard` to unwrap the object correctly. SwiftLint will capture these. [![SwiftLint: fatal_error_message](https://img.shields.io/badge/SwiftLint-fatal__error__message-007A87.svg)](https://github.com/realm/SwiftLint/blob/master/Rules.md#fatal-error-message) [![SwiftLint: force_cast](https://img.shields.io/badge/SwiftLint-force__cast-007A87.svg)](https://github.com/realm/SwiftLint/blob/master/Rules.md#force-cast)

    *Not preferred:*
	
     ```swift
      class TextField {
     	
          func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
              let cell = tableView.dequeueReusableCell(withIdentifier: Constant.Cells.SearchResultTableCell, for: indexPath) as! SearchResultTableCell
              cell.selectionStyle = .none
              cell.layoutIfNeeded()
              return cell
          }
          
      }
   	```
   	
   	*Preferred:*
	
     ```swift
      class TextField {
      
          func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
              guard let cell = tableView.dequeueReusableCell(
                  withIdentifier: Constant.Cells.SearchResultTableCell,
                  for: indexPath) as? SearchResultTableCell else { return UITableViewCell() }
              cell.selectionStyle = .none
              cell.layoutIfNeeded()
              return cell
          }
          
      }
   	```
  
* **Avoid performing any meaningful or time-intensive work in `init()`.** Avoid doing things like opening database connections, making network requests, reading large amounts of data from disk, etc. Create something like a `start()` method if these things need to be done before an object is ready for use.

* **Extract complex property observers into methods.** This reduces nestedness, separates side-effects from property declarations, and makes the usage of implicitly-passed parameters like `oldValue` explicit.

	*Not preferred:*
	
	```swift
	class TextField {
	    
		var text: String? {
			didSet {
				guard oldValue != text else {
					return
				}
				  
				// Do a bunch of text-related side-effects.
			}
		}
	    
	}
	```
	
	*Preferred:*
		
	```swift
	class TextField {
	    
		var text: String? {
			didSet { textDidUpdate(from: oldValue) }
		}
			  
		private func textDidUpdate(from oldValue: String?) {
			guard oldValue != text else {
				return
			}
		  
			// Do a bunch of text-related side-effects.
		}
	    
	}
	```

* **Extract complex callback blocks into methods**. This limits the complexity introduced by weak-self in blocks and reduces nestedness. If you need to reference self in the method call, make use of `guard` to unwrap self for the duration of the callback.

	*Not preferred:*
	
	```swift
	class MyClass {
	  
		func request(completion: () -> Void) {
			API.request() { [weak self] response in
				if let self = self {
					// Processing and side effects
				}
				completion()
			}
		}
	    
	}
	```

	*Preferred:*
	
	```swift
	class MyClass {
	  
		func request(completion: () -> Void) {
			API.request() { [weak self] response in
				guard let self = self else { return }
				self.doSomething(self.property)
				completion()
			}
		}
		  
		func doSomething(nonOptionalParameter: SomeClass) {
			// Processing and side effects
		}
	    
	}
	```

* **Prefer using `guard` at the beginning of a scope.**

  It's easier to reason about a block of code when all `guard` statements are grouped together at the top rather than intermixed with business logic.

* **Access control should be at the strictest level possible.** Prefer `public` to `open` and `private` to `fileprivate` unless you need that behavior.

* **Avoid global functions whenever possible.** Prefer methods within type definitions.

	*Not preferred:*

	```swift
	func age(of person, bornAt timeInterval) -> Int {
		// ...
	}
  
	func jump(person: Person) {
		// ...
	}
	```

	*Preferred:*

	```swift
	class Person {
		var bornAt: TimeInterval
  
		var age: Int {
			// ...
		}
  
		func jump() {
			// ...
		}
    
	}
	```

* **Prefer putting constants in the top level of a file if they are `private`.** If they are `public` or `internal`, define them as static properties, for namespacing purposes.

	*Preferred:*
	
	```swift
	private let privateValue = "secret"
	  
	public class MyClass {
	  
		public static let publicValue = "something"
		  
		func doSomething() {
			print(privateValue)
			print(MyClass.publicValue)
		}
	    
	}
	```

* **Use caseless `enum`s for organizing `public` or `internal` constants and functions into namespaces.** Avoid creating non-namespaced global constants and functions. Feel free to nest namespaces where it adds clarity. Caseless `enum`s work well as namespaces because they cannot be instantiated, which matches their intent.
  
	*Preferred:*
	
	```swift
	enum Environment {
	  
		enum Earth {
			static let gravity = 9.8
		}
		  
		enum Moon {
			static let gravity = 1.6
		}
    
	}
	```

* **Use Swift's automatic enum values unless they map to an external source.** Add a comment explaining why explicit values are defined. [![SwiftLint: redundant_string_enum_value](https://img.shields.io/badge/SwiftLint-redundant__string__enum__value-007A87.svg)](https://github.com/realm/SwiftLint/blob/master/Rules.md#redundant-string-enum-value)

	To minimize user error, improve readability, and write code faster, rely on Swift's automatic enum values. If the value maps to an external source (e.g. it's coming from a network request) or is persisted across binaries, however, define the values explicity, and document what these values are mapping to.
	
	This ensures that if someone adds a new value in the middle, they won't accidentally break things.

  

	*Not preferred:*
	
	```swift
	enum ErrorType: String {
		case error = "error"
		case warning = "warning"
	}
	  
	enum UserType: String {
		case owner
		case manager
		case member
	}
	  
	enum Planet: Int {
		case mercury = 0
		case venus = 1
		case earth = 2
		case mars = 3
		case jupiter = 4
		case saturn = 5
		case uranus = 6
		case neptune = 7
	}
	  
	enum ErrorCode: Int {
		case notEnoughMemory
		case invalidResource
		case timeOut
	}
	```

	*Preferred:*

	```swift
	enum ErrorType: String {
		case error
		case warning
	}
	  
	/// These are written to a logging service. Explicit values ensure they're consistent across binaries.
	
	// swiftlint:disable redundant_string_enum_value
	enum UserType: String {
		case owner = "owner"
		case manager = "manager"
		case member = "member"
	}
	
	// swiftlint:enable redundant_string_enum_value  
	enum Planet: Int {
		case mercury
		case venus
		case earth
		case mars
		case jupiter
		case saturn
		case uranus
		case neptune
	}
	  
	/// These values come from the server, so we set them here explicitly to match those values.
	enum ErrorCode: Int {
		case notEnoughMemory = 0
		case invalidResource = 1
		case timeOut = 2
	}
	```

* **Use optionals only when they have semantic meaning.**

* **Prefer immutable values whenever possible.** Use `map` and `compactMap` instead of appending to a new collection. Use `filter` instead of removing elements from a mutable collection.

	Mutable variables increase complexity, so try to keep them in as narrow a scope as possible.

  

	*Not preferred:*

	```swift
	var results = [SomeType]()
	for element in input {
		let result = transform(element)
		results.append(result)
	}
	```

	*Preferred:*

	```swift
	let results = input.map { transform($0) }
	```

  

	*Not preferred:*

	```swift
	var results = [SomeType]()
	for element in input {
		if let result = transformThatReturnsAnOptional(element) {
			results.append(result)
		}
	}
	```

	*Preferred:*

	```swift
	let results = input.compactMap { transformThatReturnsAnOptional($0) }
	```

* **Handle an unexpected but recoverable condition with an `assert` method combined with the appropriate logging in production. If the unexpected condition is not recoverable, prefer a `precondition` method or `fatalError()`.** This strikes a balance between crashing and providing insight into unexpected conditions in the wild. Only prefer `fatalError` over a `precondition` method when the failure message is dynamic, since a `precondition` method won't report the message in the crash report. [![SwiftLint: fatal_error_message](https://img.shields.io/badge/SwiftLint-fatal__error__message-007A87.svg)](https://github.com/realm/SwiftLint/blob/master/Rules.md#fatal-error-message) [![SwiftLint: force_cast](https://img.shields.io/badge/SwiftLint-force__cast-007A87.svg)](https://github.com/realm/SwiftLint/blob/master/Rules.md#force-cast) [![SwiftLint: force_try](https://img.shields.io/badge/SwiftLint-force__try-007A87.svg)](https://github.com/realm/SwiftLint/blob/master/Rules.md#force-try) [![SwiftLint: force_unwrapping](https://img.shields.io/badge/SwiftLint-force__unwrapping-007A87.svg)](https://github.com/realm/SwiftLint/blob/master/Rules.md#force-unwrapping)

  *Preferred:*

  ```swift
  func didSubmitText(_ text: String) {
    // It's unclear how this was called with an empty string; our custom text field shouldn't allow this.
    // This assert is useful for debugging but it's OK if we simply ignore this scenario in production.
    guard !text.isEmpty else {
      assertionFailure("Unexpected empty string")
      return
    }
    // ...
  }
  
  func transformedItem(atIndex index: Int, from items: [Item]) -> Item {
    precondition(index >= 0 && index < items.count)
    // It's impossible to continue executing if the precondition has failed.
    // ...
  }
  
  func makeImage(name: String) -> UIImage {
    guard let image = UIImage(named: name, in: nil, compatibleWith: nil) else {
      fatalError("Image named \(name) couldn't be loaded.")
      // We want the error message so we know the name of the missing image.
    }
    return image
  }
  ```

* **Default type methods to `static`.**

	If a method needs to be overridden, the author should opt into that functionality by using the `class` keyword instead.

  
	*Not preferred:*

	```swift
	class Fruit {
		class func eatFruits(_ fruits: [Fruit]) { ... }
	}
	```

	*Preferred:*

	```swift
	class Fruit {
		static func eatFruits(_ fruits: [Fruit]) { ... }
	}
	```

* **Default classes to `final`.**

	If a class needs to be overridden, the author should opt into that functionality by omitting the `final` keyword.

  

	*Not preferred:*

	```swift
	class SettingsRepository {
		// ...
	}
	```

	*Preferred:*
	
	```swift
	final class SettingsRepository {
		// ...
	}
	```

* **Never use the `default` case when `switch`ing over an enum.**

	Enumerating every case requires developers and reviewers have to consider the correctness of every switch statement when new cases are added.

	*Not preferred:*
	
	```swift
	switch anEnum {
	case .a:
		// Do something
	default:
		// Do something else.
	}
	```

	*Preferred:*
	
	```swift
	switch anEnum {
	case .a:
		// Do something
	case .b, .c:
		// Do something else.
	}
	```

* **Check for nil rather than using optional binding if you don't need to use the value.** [![SwiftLint: unused_optional_binding](https://img.shields.io/badge/SwiftLint-unused_optional_binding-007A87.svg)](https://github.com/realm/SwiftLint/blob/master/Rules.md#unused-optional-binding)

	#### Why?
	Checking for nil makes it immediately clear what the intent of the statement is. Optional binding is less explicit.
	
	*Not preferred:*
  
	```swift
	var thing: Thing?
		
	if let _ = thing {
		doThing()
	}
	```
  
	*Preferred:*
	
	```swift
	var thing: Thing?
  
	if thing != nil {
		doThing()
	}
	```
 
**[⬆ back to top](#table-of-contents)**

# File Organization

<a id='prefer-pure-swift-classes'></a>(<a href='#prefer-pure-swift-classes'>quick-link</a>) 

* **Alphabetize module imports at the top of the file a single line below the last line of the header comments. Do not add additional line breaks between import statements.** [![SwiftFormat: sortedImports](https://img.shields.io/badge/SwiftFormat-sortedImports-7B0051.svg)](https://github.com/nicklockwood/SwiftFormat/blob/master/Rules.md#sortedImports)


	#### Why?
	A standard organization method helps engineers more quickly determine which modules a file depends on.

	*Not preferred:*

	```swift
	//  Copyright © 2020 Checkfer Ltd. All rights reserved.
	//
	import DLSPrimitives
	import Constellation
	import Epoxy
	
	import Foundation
	```

	*Preferred:*

	```swift
	//  Copyright © 2020 Checkfer Ltd. All rights reserved.
	//

	import Constellation
	import DLSPrimitives
	import Epoxy
	import Foundation
 	```

	_Exception: `@testable import` should be grouped after the regular import and separated by an empty line._

	*Not preferred:*

	```swift
	//  Copyright © 2020 Checkfer Ltd. All rights reserved.
	//
	
	import DLSPrimitives
	@testable import Epoxy
	import Foundation
	import Nimble
	import Quick
	```

	*Preferred:*

	```swift
	//  Copyright © 2020 Checkfer Ltd. All rights reserved.
	//
	
	import DLSPrimitives
	import Foundation
	import Nimble
	import Quick
	
	@testable import Epoxy
	```

* **Limit empty vertical whitespace to one line.** Favor the following formatting guidelines over whitespace of varying heights to divide files into logical groupings. [![SwiftLint: vertical_whitespace](https://img.shields.io/badge/SwiftLint-vertical__whitespace-007A87.svg)](https://github.com/realm/SwiftLint/blob/master/Rules.md#vertical-whitespace)

* **Files should end in a newline.** [![SwiftLint: trailing_newline](https://img.shields.io/badge/SwiftLint-trailing__newline-007A87.svg)](https://github.com/realm/SwiftLint/blob/master/Rules.md#trailing-newline)

**[⬆ back to top](#table-of-contents)**

## Objective-C Interoperability

<a id='prefer-pure-swift-classes'></a>(<a href='#prefer-pure-swift-classes'>quick-link</a>) 

* **Prefer pure Swift classes over subclasses of NSObject.** If your code needs to be used by some Objective-C code, wrap it to expose the desired functionality. Use `@objc` on individual methods and variables as necessary rather than exposing all API on a class to Objective-C via `@objcMembers`.
	
	*Preferred:*
  
	```swift
	class PriceBreakdownViewController {
	  
		private let acceptButton = UIButton()
		  
		private func setUpAcceptButton() {
			acceptButton.addTarget(
				self,
				action: #selector(didTapAcceptButton),
				forControlEvents: .TouchUpInside)
		}
		  
		@objc
		private func didTapAcceptButton() {
			// ...
		}
	}
	```

**[⬆ back to top](#table-of-contents)**

## Contributors

  - [View Contributors](https://github.com/Checkfer-Limited/swift/graphs/contributors)

**[⬆ back to top](#table-of-contents)**

## Amendments

We encourage you to fork this guide and change the rules to fit your team’s style guide. Below, you may list some amendments to the style guide. This allows you to periodically update your style guide without having to deal with merge conflicts.

**[⬆ back to top](#table-of-contents)**
