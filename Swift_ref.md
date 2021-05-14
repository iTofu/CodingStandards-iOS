# Swift Coding Standards

> ref: https://engineering.vokal.io/iOS/CodingStandards/Swift.md.html

## Using Swift

- These standards apply to Swift 5.0 and later.
- When using Swift in a Vokal project, use Swift 5.0 or higher.
- Favor readability and clarity above fewer keystrokes.

## When In Doubt

If questions aren't addressed here refer to the style guides of [raywenderlich.com](https://github.com/raywenderlich/swift-style-guide) and [Apple](https://docs.swift.org/swift-book/) and [Swift API Design Guidelines](https://swift.org/documentation/api-design-guidelines/). If there are inconsistencies, our own standards take precedence. We're at the mercy of a cruel and capricious language unless you, the Vokal iOS Engineer, open a pull request.

---

- [Musts](#musts)
    - [Native Swift Types](#native-swift-types)
    - [Class Prefixes](#class-prefixes)
    - [Optionals](#optionals)
    - [Error Handling](#error-handling)
    - [Let vs. Var](#let-vs-var)
    - [Access Control](#access-control)
    - [Spacing](#spacing)
    - [Memory Management](#memory-management)
    - [Closures](#closures)
    - [Protocols](#protocols)
    - [Arrays and Dictionaries](#arrays-and-dictionaries)
    - [Constants](#constants)
    - [Function Parameters](#function-parameters)
    - [Semicolons](#semicolons)
    - [Typealiases](#typealiases)
    - [Flow Control](#flow-control)
    - [Switch Statements](#switch-statements)
    - [Use Implicit Getters](#use-implicit-getters)
    - [Implicit `return`](#implicit-return)
    - [Loops](#loops)

- [Shoulds](#shoulds)
    - [Declaring Variables](#declaring-variables)
    - [Optionals](#optionals-1)
    - [Spacing](#spacing-1)
    - [Usage of self](#usage-of-self)
    - [Loops](#loops-1)
    - [Closures](#closures-1)
    - [Implicit `return`](#implicit-return-1)
    - [Operator Overloading + Custom Operators](#operator-overloading--custom-operators)
    - [Tuples](#tuples)
    - [Constants](#constants-1)
    - [Default Initializers](#default-initializers)
    - [Classes vs Structs](#classes-vs-structs)

- [Tips & Tricks](#tips--tricks)

    - [Simplify Xcode's Autocompletion Suggestions](#simplify-xcodes-autocompletion-suggestions)

## Musts

### Native Swift Types

- **Use Swift types whenever possible** (`Array`, `Dictionary`, `Set`, `String`, etc.) as opposed to the `NS*` types from Objective-C.  Many Objective-C types can be automatically converted to Swift types and vice versa.  See [Working With Cocoa Data Types](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/BuildingCocoaApps/WorkingWithCocoaDataTypes.html#//apple_ref/doc/uid/TP40014216-CH6-ID61) for more details.

    **Incorrect**

    ```swift
    let pageLabelText = NSString(format: "%@/%@", currentPage, pageCount)
    ```

    **Correct**

    ```swift
    let pageLabelText = "\(currentPage)/\(pageCount)"
    let alsoPageLabelText = currentPage + "/" + pageCount
    ```

#### Swift Collection Types

- Do not make `NSArray`, `NSDictionary`, and `NSSet` properties or variables.  If you need to use a specific method only found on a Foundation collection, cast your Swift type in order to use that method.

    **Incorrect**

    ```swift
    var arrayOfJSONObjects: NSArray = NSArray()
    ...
    let names: AnyObject? = arrayOfJSONObjects.value(forKeyPath: "name")
    ```

    **Correct**

    ```swift
    var arrayOfJSONObjects = [[String: AnyObject]]()
    ...
    let names: AnyObject? = (arrayOfJSONObjects as NSArray).value(forKeyPath: "name")
    ```

- Consider if there is a Swiftier way to do what you're trying to do:

    **Swiftier Correct**

    ```swift
    var arrayOfJSONObjects = [[String: AnyObject]]()
    ...
    let names: [String] = arrayOfJSONObjects.compactMap { object in
        return object["name"] as? String
    }
    ```

### Class Prefixes

- Do not add a class prefix.  Swift types are automatically namespaced by the module that contains them.  If two names from different modules collide you can disambiguate by prefixing the type name with the module name.

    ```swift
    // SomeModule.swift
    public class UsefulClass {
        public class func helloWorld() {
            print("helloWorld from SomeModule")
        }
    }

    // MyApp.Swift
    class UsefulClass {
        class func helloWorld() {
            print("helloWorld from MyApp")
        }   
    }

    import SomeModule

    let appClass = UsefulClass.helloWorld()
    let moduleClass = SomeModule.UsefulClass.helloWorld()
    ```

  Source: [RW - Swift Style Guide](https://github.com/raywenderlich/swift-style-guide/blob/master/README.markdown#class-prefixes)

### Optionals

#### Force Unwrapping

- **Avoid force unwrapping optionals** by using `!` or `as!` as this will cause your app to crash if the value you are trying to use is `nil`. Safely unwrap the optional first by using things like `guard let`, `if let`, `guard let as?`, `if let as?`, and optional chaining. A rare reason to force-unwrap would be if you have a value you expect to never be `nil` and you want your app to crash if the value actually is `nil` due to some implementation mistake. An example of this would be an `@IBOutlet` that accidentally gets disconnected. However, consider this an edge-case and rethink whether your own code could be refactored to not use force-unwrapping.

    **Incorrect unwrap**

    ```swift
    // URL init(string:) is a failable initializer and will crash at runtime with a force unwrap if initialization fails!
    let url = URL(string: "http://www.example.com/")!

    UIApplication.shared.open(url)
    ```

    **Correct unwrap**

    ```swift
    guard let url = URL(string: "http://www.example.com/") else {
        return
    }

    UIApplication.shared.open(url)
    ```

    **Incorrect downcast**

    ```swift
    // segue.destination is declared to be of type UIViewController, so forcing a downcast to type
    // DetailViewController here will crash if the type is not DetailViewController at runtime!
    let detailViewController = segue.destination as! DetailViewController
    detailViewController.person = person
    ```

    **Correct downcast**

    ```swift
    guard let detailViewController = segue.destination as? DetailViewController else {
        return
    }

    detailViewController.person = person
    ```

    **Incorrect optional chaining**

    ```swift
    // delegate is an optional so force unwrapping here will crash if delegate is actually nil at runtime!
    delegate!.didSelectItem(item)
    ```

    **Correct optional chaining**

    ```swift
    delegate?.didSelectItem(item)
    ```

#### `if let` Pyramid of Doom

- Use multiple optional binding in an `if let` statement where possible to avoid the [pyramid of doom](https://en.wikipedia.org/wiki/Pyramid_of_doom_(programming)):

    **Incorrect**

    ```swift
    if let id = jsonObject[Constants.id] as? Int {
        if let firstName = jsonObject[Constants.firstName] as? String {
            if let lastName = jsonObject[Constants.lastName] as? String {
                if let initials = jsonObject[Constants.initials] as? String {
                    // Deep nesting
                    let user = User(id: id, firstName: name, lastName: lastName, initials: initials)
                    // ...
                }
            }
        }
    }
    ```

    **Correct**

    ```swift
    if
        let id = jsonObject[Constants.Id] as? Int,
        let firstName = jsonObject[Constants.firstName] as? String,
        let lastName = jsonObject[Constants.lastName] as? String,
        let initials = jsonObject[Constants.initials] as? String {
            // Flat
            let user = User(id: id, name: name, initials: initials)
            // ...
    }
    ```

- If there are multiple unwrapped variables created, put each on its own line for readability (as in the example above).

#### Unwrapping Multiple Optionals

- When using `guard`, `if`, or `while` to unwrap multiple optionals, put each constant and/or variable onto its own line, followed by a `,` except for the last line, which should be followed by `else {` for `guard` (though this may be on its own line), or `{` for `if` and `while`.

    **Incorrect**

    ```swift
    guard let constantOne = valueOne,
        let constantTwo = valueTwo,
        let constantThree = valueThree
        else {
            return
    }

    if let constantOne = valueOne,
        let constantTwo = valueTwo,
        let constantThree = valueThree
        {
            // Code
    }

    guard let constantOne = valueOne, constantTwo = valueTwo, constantThree = valueThree else {
        return
    }

    if let constantOne = valueOne, let constantTwo = valueTwo, let constantThree = valueThree {
        // Code
    }
    ```

    **Correct**

    ```swift
    guard
        let constantOne = valueOne,
        let constantTwo = valueTwo,
        let constantThree = valueThree else {
            return
    }

    if
        let constantOne = valueOne,
        let constantTwo = valueTwo,
        let constantThree = valueThree {
            // Code
    }
    ```

- Put a line-break after `guard`, `if`, or `while` and list each constant or variable its own line.

    **Incorrect**

    ```swift
    guard let
        constantOne = valueOne,
        var variableOne = valueTwo,
        let constantTwo = valueThree else {
            return
    }

    if let constantOne = valueOne,
        var variableOne = valueTwo,
        var variableTwo = valueThree,
        var variableThree = valueFour,
        let constantTwo = valueFive {
            // Code
    }
    ```

    **Correct**

    ```swift
    guard
        let constantOne = valueOne,
        let constantTwo = valueTwo,
        let constantThree = valueThree,
        var variableOne = valueFour,
        var variableTwo = valueFive,
        var variableThree = valueSix else {
            return
    }

    if
        let constantOne = valueOne,
        let constantTwo = valueTwo,
        let constantThree = valueThree,
        var variableOne = valueFour,
        var variableTwo = valueFive,
        var variableThree = valueSix {
            // Code
    }
    ```

### Error Handling

#### Forced-try Expression

- **Avoid using the forced-try expression** `try!` as a way to ignore errors from throwing methods as this will crash your app if the error actually gets thrown. Safely handle errors using a `do` statement along with `try` and `catch`. A rare reason to use the forced-try expression is similar to force unwrapping optionals; you actually want the app to crash (ideally during debugging before the app ships) to indicate an implementation error. An example of this would be loading a bundle resource that should always be there unless you forgot to include it or rename it.

    **Incorrect**

    ```swift
    // This will crash at runtime if there is an error parsing the JSON data!
    let json = try! JSONSerialization.jsonObject(with: data, options: .allowFragments)
    print(json)
    ```

    **Correct**

    ```swift
    do {
        let json = try JSONSerialization.jsonObject(with: data, options: .allowFragments)
        print(json)
    } catch {
        print(error)
    }
    ```

### Let vs. Var

- Whenever possible use `let` instead of `var`.
- Declare properties of an object or struct that shouldn't change over its lifetime with `let`.
- If the value of the property isn't known until creation, it can still be declared `let`: [assigning constant properties during initialization](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/Initialization.html#//apple_ref/doc/uid/TP40014097-CH18-ID212).

### (#access-control)Access Control

- Prefer `private` properties and methods whenever possible to encapsulate and limit access to internal object state.

- For private declarations at the top level of a file that are outside of a type, explicitly specify the declaration as `fileprivate`. This is functionally the same as marking these declarations `private`, but clarifies the scope:

    **Incorrect**

    ```swift
    import Foundation

    // Top level declaration
    private let foo = "bar"

    struct Baz {
    ...
    ```

    **Correct**

    ```swift
    import Foundation

    // Top level declaration
    fileprivate let foo = "bar"

    struct Baz {
    ...
    ```

- If you need to expose functionality to other modules, prefer `public` classes and class members whenever possible to ensure functionality is not accidentally overridden. Better to expose the class to `open` for subclassing when needed.

### Spacing

- Open curly braces on the same line as the statement and close on a new line.

- Don't add empty lines after opening braces or before closing braces.

- Put `else` statements on the same line as the closing brace of the previous `if` block.

- Make all colons left-hugging (no space before but a space after) except when used with the ternary operator (a space both before and after).

    **Incorrect**

    ```swift
    class SomeClass : SomeSuperClass
    {

        private let someString:String

        func someFunction(someParam :Int)
        {
            
            let dictionaryLiteral : [String : AnyObject] = ["foo" : "bar"]

            let ternary = (someParam > 10) ? "foo": "bar"

            if someParam > 10 { ... }

            else {
                ...
            } } }
    ```

    **Correct**

    ```swift
    class SomeClass: SomeSuperClass {
        private let someString: String

        func someFunction(someParam: Int) {
            let dictionaryLiteral: [String: AnyObject] = ["foo": "bar"]

            let ternary = (someParam > 10) ? "foo" : "bar"

            if someParam > 10 {
                ...
            } else {
                ...
            }
        }
    }
    ```

### Memory Management

- A “normal” reference is a `strong` one, which increments the retain count of the underlying object and ensures that the object continues to live.

- A `weak` reference does not increment the retain count of the underlying object, so the object may go away, but if the object goes away, the `weak` reference will become nil.

- An `unowned` reference also does not increment the retain count of the underlying object, so the object may go away, but if the object goes away, the `unowned` reference will become a dangling pointer, which is dangerous.  [The docs](https://docs.swift.org/swift-book/LanguageGuide/AutomaticReferenceCounting.html#ID54) say “If you try to access the value of an unowned reference after that instance has been deallocated, you’ll get a runtime error.“

- **Do not use unowned unless you have a very specific reason why it’s the right thing to do.**

### Closures

#### Shorthand Argument Syntax

- Only use shorthand argument syntax for simple one-line closure implementations:

    ```swift
    let doubled = [2, 3, 4].map { $0 * 2 } // [4, 6, 8]
    ```

- For all other cases, explicitly define the argument(s):

    ```swift
    let names = ["George Washington", "Martha Washington", "Abe Lincoln"]
    let emails: [String] = names.map { fullname in
        let dottedName = fullname.replacingOccurrences(of: " ", with: ".")
        return dottedName.lowercased() + "@whitehouse.gov"
    }
    ```

#### Capture lists

- Use capture lists to [resolve strong reference cycles in closures](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/AutomaticReferenceCounting.html#//apple_ref/doc/uid/TP40014097-CH20-ID57). We typically want `[weak self]` here to avoid the strong capturing of self inside the block, as it is less likely to lead to retain cycles that cause memory leaks:

    **Correct**

    ```swift
    UserAPI.registerUser(user) { [weak self] result in
        if result.success {
            self?.doSomethingWithResult(result)
        }
    }
    ```

    **Incorrect**

    ```swift
    UserAPI.registerUser(user) { result in
        if result.success {
            self.doSomethingWithResult(result)
        }
    }
    ```

    **Incorrect**

    ```swift
    UserAPI.registerUser(user) { [unowned self] result in
        if result.success {
            self.doSomethingWithResult(result)
        }
    }
    ```

### Protocols

#### Protocol Conformance

- When adding protocol conformance to a type, use a separate extension for the protocol methods. This keeps the related methods grouped together with the protocol and can simplify instructions to add a protocol to a type with its associated methods.

- Use a `// MARK: - SomeDelegate` comment to keep things well organized.

    **Incorrect**

    ```swift
    class MyViewcontroller: UIViewController, UITableViewDataSource, UIScrollViewDelegate {
      // All methods
    }
    ```

    **Correct**

    ```swift
    class MyViewcontroller: UIViewController {
      ...
    }

    // MARK: - UITableViewDataSource

    extension MyViewcontroller: UITableViewDataSource {
      // Table view data source methods
    }

    // MARK: - UIScrollViewDelegate

    extension MyViewcontroller: UIScrollViewDelegate {
      // Scroll view delegate methods
    }
    ```

#### Delegate Protocols

- Limit delegate protocols to classes only by adding `class` to the protocol's inheritance list (as discussed in [Class-Only Protocols](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/Protocols.html#//apple_ref/doc/uid/TP40014097-CH25-ID281)).

- If your protocol should have [optional methods](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/Protocols.html#//apple_ref/doc/uid/TP40014097-CH25-ID284), it must be declared with the `@objc` attribute.

- Declare protocol definitions near the class that uses the delegate, not the class that implements the delegate methods.

- If more than one class uses the same protocol, declare it in its own file.

- Use `weak` optional `var`s for delegate variables to avoid retain cycles.

    ```swift
    //SomeTableCell.swift

    protocol SomeTableCellDelegate: class {
        func cellButtonWasTapped(cell: SomeTableCell)
    }

    class SomeTableCell: UITableViewCell {
        weak var delegate: SomeTableCellDelegate?
        // ...
    }
    ```

    ```swift
    //SomeTableViewController.swift

    class SomeTableViewController: UITableViewController {
        // ...
    }

    // MARK: - SomeTableCellDelegate

    extension SomeTableViewController: SomeTableCellDelegate {
        func cellButtonWasTapped(cell: SomeTableCell) {
            // Implementation of cellbuttonwasTapped method
        }
    }
    ```

### Arrays and Dictionaries

#### Type Shorthand Syntax

- Use square bracket shorthand type syntax for Array and Dictionary as recommended by Apple in [Array Type Shorthand Syntax](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/CollectionTypes.html#//apple_ref/doc/uid/TP40014097-CH8-ID107):

    **Incorrect**

    ```swift
    let users: Array<String>
    let usersByName: Dictionary<String, User>
    ```

    **Correct**

    ```swift
    let users: [String]
    let usersByName: [String: User]
    ```

#### Trailing Comma

- For array and dictionary literals, unless the literal is very short, split it into multiple lines, with the opening symbols on their own line, each item or key-value pair on its own line, and the closing symbol on its own line. Put a trailing comma after the last item or key-value pair to facilitate future insertion/editing. Xcode will handle alignment sanely.

    **Correct**

    ```swift
    let anArray = [
        object1,
        object2,
        object3,
    ]

    let aDictionary = [
        "key1": value1,
        "key2": value2,
    ]
    ```

    **Incorrect**

    ```swift
    let anArray = [
        object1,
        object2,
        object3 //no trailing comma
    ]

    let aDictionary = ["key1": value1, "key2": value2] //how can you even read that?!
    ```

### Constants

- Define constants for unchanging pieces of data in the code. Some examples are `CGFloat` constants for cell heights, string constants for cell identifiers, key names (for KVC and dictionaries), or segue identifiers.

- Where possible, keep constants private to the file they are related to.

- File-level constants must be declared with `fileprivate let`.

- File-level constants must be capital camel-cased to indicate that they are named constants instead of properties.

- If the constant will be used outside of one file, `fileprivate` must be omitted.

- If the constant will be used outside of the module, it must be declared `public` (mostly useful for Pods or shared libraries).

- If the constant is declared within a class or struct, it must be declared `static` to avoid declaring one constant per instance.

    ```swift
    //SomeTableCell.swift

    //not declared private since it is used in another file
    let SomeTableCellIdentifier = "SomeTableCell"

    class SomeTableCell: UITableViewCell {
        ...
    }
    ```

    ```swift
    //ATableViewController.swift

    //declared fileprivate since it isn't used outside this file
    fileprivate let RowHeight: CGFloat = 150.0

    class ATableViewController: UITableViewController {

        ...

        private func configureTableView() {
            tableView.rowHeight = RowHeight
        }

        func tableView(tableView: UITableView, cellForRowAtIndexPath indexPath: NSIndexPath) -> UITableViewCell {
            return tableView.dequeueReusableCellWithIdentifier(SomeTableCellIdentifier, forIndexPath: indexPath)
        }
    }
    ```

### Function Parameters

- Follow the Swift API Design Guidelines for proper handling of [function parameters](https://swift.org/documentation/api-design-guidelines/#parameter-names) and [argument labels](https://swift.org/documentation/api-design-guidelines/#argument-labels).

### Semicolons

- Forget them. They're dead to us.
- Don't you dare put multiple statements on one line.

### Typealiases

- Create `typealias`es to give semantic meaning to commonly used datatypes and closures.

- `typealias` is equivalent to `typedef` in C and should be used for making names for types.

- Where appropriate, nest `typealias`es inside parent types (classes, structs, etc.) to which they relate.

    ```swift
    typealias IndexRange = Range<Int>

    typealias JSONObject = [String: AnyObject]

    typealias APICompletion = (jsonResult: [JSONObject]?, error: NSError?) -> Void

    typealias BasicBlock = () -> Void
    ```

### Flow Control

- For single conditional statements, do not use parentheses.

- Use parentheses around compound conditional statements for clarity or to make the order of operations explicit.

- When using optional booleans and optional `NSNumber`s that represent booleans, check for `true` or `false` rather than using the nil-coalescing operator:

    **Incorrect**

    ```swift
    if user.isCurrent?.boolValue ?? false {
        // isCurrent is true
    } else {
        // isCurrent is nil or false
    }
    ```

    **Correct**

    ```swift
    if user.isCurrent?.boolValue == true {
        // isCurrent is true
    } else {
        // isCurrent is nil or false
    }
    ```

### Switch Statements

- `break` is not needed between `case` statements (they don't fall through by default)

- Use multiple values on a single `case` where it is appropriate:

    ```swift
    var someCharacter: Character

    ...

    switch someCharacter {
    case "a", "e", "i", "o", "u":
        print("\(someCharacter) is a vowel")
    ...
    }
    ```

- When pattern matching over an enum case with an associated value, use `case .CASENAME(let ...)` rather than `case let ...` syntax for value binding.

    **Incorrect**

    ```swift
    enum AnEnum {
        case foo
        case bar(String)
        case baz
    }

    let anEnumInstanceWithAssociatedValue = AnEnum.Bar("hello")

    switch anEnumInstanceWithAssociatedValue {
        case .foo: print("Foo")
        // Incorrect
        case let .bar(barValue): print(barValue) // "hello"
        case .baz: print("Baz")
    }
    ```

    **Correct**

    ```swift
    enum AnEnum {
        case foo
        case bar(String)
        case baz
    }

    let anEnumInstanceWithAssociatedValue = AnEnum.Bar("hello")

    switch anEnumInstanceWithAssociatedValue {
        case .foo: print("Foo")
        // Correct
        case .bar(let barValue): print(barValue) // "hello"
        case .baz: print("Baz")
    }
    ```

### Use Implicit Getters

- When possible, omit the `get` keyword on read-only, computed properties and read-only subscripts as recommended by Apple under [Read-Only Computed Properties](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/Properties.html#//apple_ref/doc/uid/TP40014097-CH14-ID259):

    **Incorrect**

    ```swift
    var someProperty: Int {
        get {
            4 * someOtherProperty
        }
    }

    subscript(index: Int) -> T {
        get {
            return object[index]
        }
    }
    ```

    **Correct**

    ```swift
    var someProperty: Int { 4 * someOtherProperty }

    subscript(index: Int) -> T {
        object[index]
    }
    ```

### Implicit `return`

- In single-line closures, implicit getters, and other code blocks where the opening `{`, inner statement, and closing `}` are all on one line, omit `return`.

    **Incorrect**

    ```swift
    let doubled = [2, 3, 4].map { return $0 * 2 }

    var someProperty: Int { return 4 * someOtherProperty }

    func helloWorld() -> String { return "Hello, world!" }
    ```

    **Correct**

    ```swift
    let doubled = [2, 3, 4].map { $0 * 2 }

    var someProperty: Int { 4 * someOtherProperty }

    func helloWorld() -> String { "Hello, world!" }
    ```

### Loops

- Use the `enumerated()` function if you need to loop over a Sequence and use the index:

    ```swift
    for (index, element) in someArray.enumerated() {
        ...
    }
    ```

- Use `map` when transforming Arrays (`compactMap` for Arrays of Optionals or `flatMap` for Arrays of Arrays):

    ```swift
    let array = [1, 2, 3, 4, 5]
    let stringArray = array.map { item in
        return "item \(item)"
    }

    let optionalArray: [Int?] = [1, nil, 3, 4, nil]
    let nonOptionalArray = optionalArray.compactMap { item -> Int? in
        guard let item = item else {
            return nil
        }

        return item * 2
    }

    let arrayOfArrays = [array, nonOptionalArray]
    let anotherStringArray = arrayOfArrays.flatmap { item in
        return "thing \(item)"
    }
    ```

- If you are not performing a transform, or if there are side effects _do not_ use `map`/`flatmap`; use a `for in` loop instead ([tips](http://www.mokacoding.com/blog/when-to-use-map-flatmap-for/)).

- Avoid the use of `forEach` except for simple one line closures, similar to `makeObjectsPerformSelector:` in Objective-C.

## Shoulds

### Declaring Variables

- When declaring variables you should be verbose, but not too verbose.

- Avoid repeating type information. Once should be enough.

- Make the name descriptive.

    **Incorrect**

    ```swift
    var someDictionary: Dictionary = [String: String]() //Dictionary is redundant
    var somePoint: CGPoint = CGPoint(x:100, y: 200) //CGPoint is repeated
    var b = Bool(false) //b is not a descriptive name
    ```

    **Correct**

    ```swift
    var someArray = [String]()
    var someArray: [String] = []
    var someDictionary = [String: Int]()
    var someDictionary: [String : Int] = [:]
    var countOfCats: UInt32 = 12
    var isMadeOfCheese = false
    var somePoint = CGPoint(x:100, y: 200)
    ```

### Optionals

#### `guard let` vs. `if let`

- Use `guard let` over `if let` where possible. This improves readability by [focusing on the happy-path](http://natashatherobot.com/swift-guard-better-than-if/) and can also reduce nesting by keeping your code flatter:

    **Incorrect**

    ```swift
    func openURL(string: String) {
        if let url = URL(string: string) {
            // Nested
            UIApplication.shared.open(url)
        }
    }
    ```

    **Correct**

    ```swift
    func openURL(string: String) {
        guard let url = URL(string: string) else {
            return
        }
        // Flat
        UIApplication.shared.open(url)
    }
    ```

- Since `guard let` needs to exit the current scope upon failure, `if let` is better suited for situations where you still need to move forward after failing to unwrap an optional:

    ```swift
    if let anImage = UIImage(named: ImageNames.background) {
        imageView.image = anImage
    }

    // Do more configuration
    // ...
    ```

### Spacing

- Use a newline for each logical step when chaining 3 or more methods together in a fluent style:

    **Incorrect**

    ```swift
    func foo() -> Int {
        let nums: [Int] = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

        return nums.map { $0 * 2 }.filter { $0 % 2 == 0 }.reduce(0, +)
    }
    ```

    **Correct**

    ```swift
    func foo() -> Int {
        let nums: [Int] = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

        return nums
            .map { $0 * 2 }
            .filter { $0 % 2 == 0 }
            .reduce(0, +)
    }
    ```

- For `guard` statements, the `else {` may be placed on its own line after the last condition (rather than on the same line as the last condition).  For a single-condition `guard`, if the `else {` is on its own line, the condition should be on its own line, too.  If the `else` clause of a `guard` statement is a simple `return` statement, it may be all on one line.

    **Incorrect**

    ```swift
    func openURL(string: String) {
        guard let url = URL(string: string)
            else {
                return
        }
        // Flat
        UIApplication.shared.open(url)
    }
    ```

    ```swift
    guard let constantOne = valueOne,
        let constantTwo = valueTwo,
        let constantThree = valueThree
        else { return }
    ```

    **Correct** (lots of choices here)

    ```swift
    func openURL(string: String) {
        guard let url = URL(string: string) else {
            return
        }
        // Flat
        UIApplication.shared.open(url)
    }
    ```

    ```swift
    func openURL(string: String) {
        guard let url = URL(string: string) else { return }
        // Flat
        UIApplication.shared.open(url)
    }
    ```

    ```swift
    func openURL(string: String) {
        guard
            let url = URL(string: string)
            else {
                return
        }
        // Flat
        UIApplication.shared.open(url)
    }
    ```

    ```swift
    func openURL(string: String) {
        guard
            let url = URL(string: string)
            else { return }
        // Flat
        UIApplication.shared.open(url)
    }
    ```

    ```swift
    guard
        let constantOne = valueOne,
        let constantTwo = valueTwo,
        let constantThree = valueThree else { return }
    ```

    ```swift
    guard
        let constantOne = valueOne,
        let constantTwo = valueTwo,
        let constantThree = valueThree
        else { return }
    ```

- When grouping multiple `case`s in a `switch` statement, prefer putting each case on its own line, unless there are only a few cases and they are short.

    **Incorrect**

    ```swift
    switch something {
    case .oneLongCase, .anotherLongCase, .thereAreMoreCases, .thisIsWayTooFarToTheRight:
        return true
    case .sanity:
        return false
    }
    ```

    **Correct**

    ```swift
    switch something {
    case .oneLongCase,
        .anotherLongCase,
        .thereAreMoreCases,
        .thisIsInASanerPlace:
        return false
    case .sanity:
        return true
    }
    ```

### Usage of `self`

- Except where necessary, avoid using `self.`.  If you have a local variable that conflicts with a property name or are in a context where `self` is captured, you may need to use `self.`.

### Loops

- If you have an Array of Arrays and want to loop over all contents, consider a `for in` loop using `joined(separator:)` instead of nested loops:

    ```swift
    let arraysOfNames = [["Moe", "Larry", "Curly"], ["Groucho", "Chico", "Harpo", "Zeppo"]]
    ```

    **Recommended**

    ```swift
    for name in arraysOfNames.joined() {
        print("\(name) is an old-timey comedian")
    }
    ```

    **Discouraged**

    ```swift
    for names in arraysOfNames {
        for name in names {
            print("\(name) is an old-timey comedian")
        }
    }
    ```

### Closures

- Avoid unnecessary parentheses around closure parameters.

    **Incorrect**

    ```swift
    functionWithAClosure { (result) in
        ...
    }
    ```

    ```swift
    functionWithAClosure { (result) -> Int in
        ...
    }
    ```

    **Correct**

    ```swift
    functionWithAClosure { result in
        ...
    }
    ```

    ```swift
    functionWithAClosure { result -> Int in
        ...
    }
    ```

    ```swift
    functionWithAClosure { (result: String) in
        ...
    }
    ```

#### Trailing Closure Syntax

- Use trailing closure syntax when the only or last argument to a function or method is a closure and there is only one closure parameter.

    ```swift
    //a function that has a completion closure/block
    func registerUser(user: User, completion: (Result) -> Void)
    ```

    **Correct**

    ```swift
    UserAPI.registerUser(user) { result in
        if result.success {
            ...
        }
    }
    ```

    **Incorrect**

    ```swift
    UserAPI.registerUser(user, completion: { result in
        if result.success {
            ...
        }
    })
    ```

- Omit the empty parens `()` when the only argument is a closure.

    **Correct**

    ```swift
    let doubled = [2, 3, 4].map { $0 * 2 }
    ```

    **Incorrect**

    ```swift
    let doubled = [2, 3, 4].map() { $0 * 2 }
    ```

- NOTE: Whether the argument list to a closure is directly after the opening `{` or on the next line is up to individual preference. However, it should be consistent throughout a project. In the code standards, we're leaving it on the same line to conserve space.

### Implicit `return`

- Avoid unnecessary explicit use of `return`.

    **Incorrect**

    ```swift
    var foo: String {
        return someCollectionVariable
            .filter { $0.shouldBeIncluded }
            .compactMap { String($0) }
            .joined()
    }

    func product(_ numbers: [Int]) -> Int {
        return numbers.reduce(1, { resultSoFar, element in
            return resultSoFar * element
        })
    }
    ```

    **Correct**

    ```swift
    var foo: String {
        someCollectionVariable
            .filter { $0.shouldBeIncluded }
            .compactMap { String($0) }
            .joined()
    }

    func product(_ numbers: [Int]) -> Int {
        numbers.reduce(1, { resultSoFar, element in
            resultSoFar * element
        })
    }
    ```

### Operator Overloading + Custom Operators

- The use of operator overloading and custom operators is **strongly discouraged** as this can hurt readability and potentially create a significant amount of confusion for other developers on a shared project. There are cases that it would be necessary (ex. overloading `==` to conform to `Equatable`). When writing a custom operator or overloading an existing one, the operator function should call another **explicitly named** function that performs that actual work. For more guidance on best practices on this matter, view the guidelines at the bottom of this [NSHipster article](http://nshipster.com/swift-operators/#guidelines-for-swift-operators).

### Tuples

- The word is pronounced like "tuh-ple"

- Rhymes with "couple" and "supple"

- Name the members of your tuples when creating or decomposing tuples:

    ```swift
    let foo = (something: "cats", somethingElse: 909_099)
    let (something, somethingElse) = foo
    ```

### Constants

- Declare constants with `static let` to ensure static storage.

- Prefer declaring constants in the scope in which they will be used rather than in a central shared constants file like _Constants.swift_.

- Be wary of large constants files as they can become unmanageable over time. Refactor related parts of the main constants file into separate files for that situation.

- Use `enum`s to group related constants together in a namespace. The name of the `enum` should be singular, and each constant should be written using camelCase.  (Using a `case`-less `enum` prevents useless instances from being created.)

    ```swift
    enum SegueIdentifier {
        static let onboarding = "OnboardingSegue"
        static let login = "LoginSegue"
        static let logout = "LogoutSegue"
    }

    enum StoryboardIdentifier {
        static let main = "Main"
        static let onboarding = "Onboarding"
        static let settings = "Settings"
    }

    print(SegueIdentifier.login) // "LoginSegue"
    ```

- Where appropriate, constants can also be grouped using an `enum` with a `rawValue` type that is relevant to the type you need to work with. An `enum` with a `rawValue` of type `String` will [implicitly assign](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/Enumerations.html#//apple_ref/doc/uid/TP40014097-CH12-ID149) its `rawValue` from the name of the case if nothing is already explicitly defined for the `rawValue`. This can be useful when all the names of the cases match with the value of the constant. Be aware that if you use `enum` `case`s for constants in this way, you need to explicitly use `rawValue` every time you need to access the value of the constant:

    ```swift
    enum UserJSONKeys: String {
        case username
        case email
        case role
        // Explicitly defined rawValue
        case identifier = "id"
        ...
    }

    print(UserJSONKeys.username.rawValue) // "username"
    print(UserJSONKeys.identifier.rawValue) // "id"

    guard let url = URL(string: "http://www.example.com") else {
        return
    }

    let mutableURLRequest = NSMutableURLRequest(url: url)
    mutableURLRequest.HTTPMethod = HTTPMethods.POST.rawValue
    print(mutableURLRequest.httpMethod) // "POST"
    ```

### Default Initializers

- Use [default initializers](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/Initialization.html#//apple_ref/doc/uid/TP40014097-CH18-ID213) where possible.

### Classes vs Structs

- Most of your custom data types should be structs and enums.
- Some situations where you may want to use classes:

    - When you need the data sharing capabilities of reference types.
    - When you need deinitializers to help free up any resources.
    - When you need runtime class type checks.
    - When you need Objective-C interoperability.
    - When you need inheritance after considering all other possibilities like: protocols, protocol inheritance, protocol extensions with default implementations, generics.

- Refer to the [Swift Programming Language Guidelines](https://docs.swift.org/swift-book/LanguageGuide/ClassesAndStructures.html) and [Choosing Between Structures and Classes](https://developer.apple.com/documentation/swift/choosing_between_structures_and_classes) for detailed info on this topic.

## Tips & Tricks

### Simplify Xcode's Autocompletion Suggestions

- Xcode will try to be helpful when autocompleting closures for you by giving you the _full_ type signature of the closure (input type(s) and return type). Simplify that information so that it's easier to read.

- Remove return types of `Void` and parentheses around single input parameters.  This is especialy relevant if the closure takes no input and returns no output.

    **What Xcode Autocompletion Suggests**

    ```swift
    UIView.animate(withDuration: 0.5) { () -> Void in
        ...
    }

    UIView.animate(withDuration: 0.5, animations: { () -> Void in
            ...   
        }) { (complete) -> Void in
            ...
    }
    ```

    **Simplified With Type Inference**

    ```swift
    UIView.animate(withDuration: 0.5) {
        // no need to specify type information for a no input, no output closure
    }

    //note the formatting of this example is further changed from the suggestion for better readability
    UIView.animate(withDuration: 0.5,
        animations: {
            ...    
        },
        completion: { complete in
            //the return type is inferred to be `Void` and `complete` does not need parens
        }
    )
    ```
