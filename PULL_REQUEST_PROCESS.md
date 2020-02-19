# iOS PR Process

This document will run through our standard iOS PR process in Checkfer. It will provide engineers with an insight into what is expected as a standard and also help us all to build a future.

## Architecture
The codebase should be following to an MVVM architecture and a modularized approach where possible.

Currently the code base has 2 cores:

### Core

The core provides the app with generic services that are required for the app to function in a general environment. It should provide the app with external third party provides, such as `Firebase` and present in a standard pattern to the application.

Contains:
  - Third party services.
  - General extension to `Foundation` and `UIKit` - anything custom should be within the main application system.
  - Test coverage across the core.

__**If other cores need to be spun up then this can be discussed at any time.**__

### Network Core

The network core provides the app with all the necessary protocols to retrieve data from Checkfer backend systems. It should also provide a route of retrieving demo data.

Contains:
  - Models.
  - Service Calls.
  - Demo data.
  - Test coverage for both calls and demo. (*No actual network operations should be tested, each call should follow the `isDemo` flow.*)


## Folder Structure

There should be a distinct project structure within the app, and this should represent what is in the folder structure. Modules (or features) should have the following:

### Modules

There should be a visible structure the the file structure, so it is easy to maintain and find files for the module.

- **<ModuleName>**
  - **Constants**: Strings, Sizes for the module (anything away from generic) there should not be any hardcoded values in the codebase.
  - **Controllers**: `UIViewControllers`
  - **Extensions**: Any custom extensions that are not generic.
  - **Models**: Any models that are specific for the module and are not obtained via the `NetworkCore`.
  - **Views**: `UIView`, `UITableViewCell` etc.
  - **ViewModels**: ViewModels that are required for `Controllers` and `Views`

These are not exhaustive, but if not required don't worry about creating an empty folder.

## Code base

### Coding style

There is a [Swift Style Guide](https://github.com/Checkfer-Limited/swift) that has been created to help standardize the code base across all engineers. Helping with readability, review processes and keeping to a modern codebase.

This style guide is a life document and any suggestions or changes that are required can be submitted via a pull request to review as a team.

The app should comply to [SwiftLint](https://github.com/realm/SwiftLint) and to help with this engineers should have this installed on their machines, allowing for scripts to run during build times.

### Naming Convention

As mentioned in the [Swift Style Guide](https://github.com/Checkfer-Limited/swift#prefer-pure-swift-classes) we need to ensure that a level of naming is adhered to, for example:

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

### Constants

There should not be any hardcoded `const` in the Views/Controllers/ViewModels they should be reference from a Const file for example:

- Strings, should be in `<ModuleName>+Strings.swift` file extension of `TextString`.
- Generic `CGFloat` values should be contained in a Global `Constant.swift` file.


## Hanging fruit

If a feature or module has not been completed for any particular reason, this code should be marked with the ```// TODO:``` marking. This marking should have a comment to present to the reader why this marking has been added. 

No TODO should be pushed into `master` unless it is not impacting the product.

  
## What to look for

When reviewing always look out for the following points:

1. Structure, both code and file system
2. Monitor deprecated files in the codebase
3. Architecture, ensure the MVVM is as pure as possible - there should be no coupling of models with controllers and views.
4. Standardized Code Style and meets the Style Guide
5. Linting
6. Constants

### *MVVM Reminders*

- File/Folder Structure
- Views have `ViewModels` not data models
- Controllers pass `ViewModel` from `<Module>ViewModel` to `View`