# Swift 编码标准

## 使用 Swift

- 这些标准适用于 Swift 5.0 及更高版本。
- 在项目中使用 Swift 时，请使用 Swift 5.0 或更高版本。
- 在减少复杂输入的基础上，提高了可读性和清晰度。

## 有疑问时

如遇此处未包含的问题，请参考 [Swift Book（Swift）](https://docs.swift.org/swift-book/)、[Swift API Design Guidelines（Swift）](https://swift.org/documentation/api-design-guidelines/) 和 [Swift Style Guide（Raywenderlich）](https://github.com/raywenderlich/swift-style-guide)。如有不一致之处，**以我们自己的标准为准**。

如果你有更好的建议，请直接 Pull Request。

---

- [Musts](#musts)
    - [Swift 原生类型](#swift-原生类型)
    - [类前缀](#类前缀)
    - [可选型](#可选型)
    - [错误/异常处理](#错误异常处理)
    - [Let vs. Var](#let-vs-var)
    - [访问控制](#访问控制)
    - [间距](#间距)
    - [内存管理](#内存管理)
    - [闭包](#闭包)
    - [协议](#协议)
    - [Array 和 Dictionary](#array-和-dictionary)
    - [常量](#常量)
    - [函数参数](#函数参数)
    - [分号](#分号)
    - [类型别名（Typealiases）](#类型别名typealiases)
    - [控制流](#控制流)
    - [Switch 语句](#switch-语句)
    - [隐式 Getter](#隐式-getter)
    - [隐式 `return`](#隐式-return)
    - [循环](#循环)

- [Shoulds](#shoulds)
    - [变量声明](#变量声明)
    - [可选型](#可选型-1)
    - [间距](#间距-1)
    - [self](#self)
    - [循环](#循环-1)
    - [闭包](#闭包-1)
    - [运算符重载 + 自定义运算符](#运算符重载-自定义运算符)
    - [元组](#元组)
    - [常量](#常量-1)
    - [默认初始化器（Default Initializers）](#默认初始化器default-initializers)
    - [类与结构体](#类与结构体)

- [提示和技巧](#提示和技巧)
    - [参数对齐](#参数对齐)

## Musts

### Swift 原生类型

- **尽可能使用 Swift 类型**（`Array`、`Dictionary`、`Set`、`String` 等），而不是 Objective-C 中的 `NS*` 类型。许多 Objective-C 类型可以轻松转换为 Swift 类型，反之亦然。参考：[Working With Cocoa Data Types（Apple）](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/BuildingCocoaApps/WorkingWithCocoaDataTypes.html#//apple_ref/doc/uid/TP40014216-CH6-ID61)。

    **Incorrect**

    ```swift
    let pageLabelText = NSString(format: "%@/%@", currentPage, pageCount)
    ```

    **Correct**

    ```swift
    let pageLabelText = "\(currentPage)/\(pageCount)"
    let alsoPageLabelText = currentPage + "/" + pageCount
    ```

#### Swift 集合类型

- 不要创建 `NSArray`、`NSDictionary` 和 `NSSet` 属性或变量。如果你需要使用只在 Foundation 中的特定方法，转换 Swift 类型后使用该方法。

    **Incorrect**

    ```swift
    var arrayOfJSONObjects: NSArray = NSArray()
    // ...
    let names: AnyObject? = arrayOfJSONObjects.value(forKeyPath: "name")
    ```

    **Correct**

    ```swift
    var arrayOfJSONObjects = [[String: AnyObject]]()
    // ...
    let names: AnyObject? = (arrayOfJSONObjects as NSArray).value(forKeyPath: "name")
    ```

- 思考是否有更 Swiftier 的方式：

    **Swiftier Correct**

    ```swift
    var arrayOfJSONObjects = [[String: AnyObject]]()
    // ...
    let names: [String] = arrayOfJSONObjects.compactMap { object in
      return object["name"] as? String
    }
    ```

### 类前缀

- 不要添加类前缀。不同于 Objective-C，Swift 类型由包含它们的模块自动命名空间。如果来自不同模块的两个名称发生冲突，可以通过在类型名称前加上模块名称来消除歧义。

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

  > 来源：[Swift Style Guide（Raywenderlich）](https://github.com/raywenderlich/swift-style-guide/blob/master/README.markdown#class-prefixes)

### 可选型

#### 强制解包

- **避免使用 `!` 或 `as!` 强制解包**，因为当该值为 `nil` 时 App 会崩溃。优先使用诸如 `guard let`、`if let`、`guard let as?`、`if let as?` 和可选链等安全地解包。强制解包的一个罕见原因是，如果你希望某个值永不为 `nil`，而当由于某些错误实现导致该值为 `nil` 时，App 直接崩溃，比如 `@IBOutlet` 意外销毁时。但这是一个边缘情况，你应该重新考虑是否有不必强制解包的方案。

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

#### `if let` 末日金字塔（Pyramid of Doom）

- 为避免 [末日金字塔（Pyramid of Doom）（Wiki）](https://en.wikipedia.org/wiki/Pyramid_of_doom_(programming))，尽量把多个可选绑定放在一个 `if let` 语句中：

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

- 为提高可读性，如果一次创建多个解包变量（Unwrapped variables），将每个变量放在单独的一行上（如上例所示）。

#### 多个解包

- 当使用 `guard`、`if` 或 `while` 解开多个可选型时，将每个变量放在单独的一行上，然后加上 `,`。

    **Incorrect**

    ```swift
    guard let constantOne = valueOne,
      let constantTwo = valueTwo,
      let constantThree = valueThree else {
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
      let constantThree = valueThree
    else {
      return
    }

    if
      let constantOne = valueOne,
      let constantTwo = valueTwo,
      let constantThree = valueThree {
      // Code
    }
    ```

- 在 `guard`、`if` 或 `while` 后换行，并把每个常量或变量单独列为一行。

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

### 错误/异常处理

#### Forced-try 表达式

- **避免使用 Forced-try 表达式** `try！`，来忽略抛出方法中的错误（异常），否则当有错误抛出时，App 将会崩溃。应使用 `do`、`try`、`catch` 来安全地处理错误。使用 Forced-try 表达式的一个罕见原因与强制解包类似：你希望通过 App 崩溃来发现错误（最好是在应用发布之前的调试过程中）。例如：加载 Bundle resource 时，该 Bundle resource 应该一直存在，避免你忘记导入它或对其重命名。

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

- 但凡可以，用 `let` 代替 `var`。
- 使用 `let` 声明的对象或结构体，不应该在其生命周期内发生改变（内存地址/值）。
- 即便属性直到创建时才知晓具体值，仍然可以在初始化时声明为 `let`：[Assigning constant properties during initialization（Apple）](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/Initialization.html#//apple_ref/doc/uid/TP40014097-CH18-ID212)。

### 访问控制

- 尽量 `private` 属性和方法，来封装和限制对内部对象状态的访问。

- 对于位于某个类型之外的、文件顶层的私有声明，应将该声明显式指定为 `fileprivate`。这在功能上与 ` private` 相同，但阐明了作用域：

    **Incorrect**

    ```swift
    import Foundation

    // Top level declaration
    private let foo = "bar"

    struct Baz {
      // ...
    }
    ```

    **Correct**

    ```swift
    import Foundation

    // Top level declaration
    fileprivate let foo = "bar"

    struct Baz {
      // ...
    }
    ```

- 如果你需要向其他模块公开一些属性和方法，请尽可能选择 `public` 类和类成员，以确保功能不会意外被重写。最好是在需要时将类 `open` 以便子类化。

- 明确不允许继承的类（如一些工具类等），应使用 `final` 修饰，以标明不可继承，并且编译器可以更好地进行优化。

### 间距

- 在语句的末尾（同一行）打开花括号，在新一行关闭。

- 不要在开始的大括号（除 Class/Struct/Enum 声明处外）之后、或结束的大括号之前添加空行。

- 将 `else` 放在毗邻的 `if` 同一行。

- 所有冒号左拥抱（前无空格、后有空格），除了与三元运算符（前后都有空格）一起使用时。

    **Incorrect**

    ```swift
    class SomeClass : SomeSuperClass
    {

      private let someString:String

      func someFunction(someParam :Int)
      {
            
        let dictionaryLiteral : [String : AnyObject] = ["foo" : "bar"]

        let ternary = (someParam > 10) ? "foo": "bar"

        if someParam > 10 { /* ... */ }

        else {
          // ...
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
          // ...
        } else {
          // ...
        }
      }
    }
    ```

- 注释起始符 `//` 右边有一个空格，如：`// MARK:`、`// TODO:`、`// FIXME:`、`// do something...` 等。
  > 特别的，`// MARK:` 与 `// MARK: -` 的区别在于，后者在文档化时会在前一行多一个分割线，也即类似于 Markdown 中的 `---` 的作用。

### 内存管理

- 不指定时，默认引用是 `strong` 引用，它增加了底层对象的引用计数，并能确保该对象始终存在。

- `weak` 引用不会增加底层对象的引用计数，因此该对象可能会销毁。如果该对象销毁了，`weak` 引用将变成 `nil`。

- `unowned` 引用也不会增加底层对象的引用计数，因此该对象可能会销毁。如果该对象销毁了，`unowned` 引用将成为**野指针**，这很**危险**。[Automatic Reference Counting（Swift）](https://docs.swift.org/swift-book/LanguageGuide/AutomaticReferenceCounting.html#ID54) 说明“如果你在实例被释放后，尝试访问野指针，则会出现运行时错误。”

- **不要使用 `unowned`，除非你有非常明确的理由说明这样做是正确的。**

  > 按：
  >
  > 当你明确知道对象在作用域内不会销毁时，可以使用 `unowned`。
  > 不是很确定时，无脑使用 `weak` 也无不可。

### 闭包

#### 简写参数语法（Shorthand Argument Syntax）

- 仅将简写参数语法（Shorthand Argument Syntax）用于简单的单行闭包实现：

    ```swift
    let doubled = [2, 3, 4].map { $0 * 2 } // [4, 6, 8]
    ```

- 对于所有复杂情况，应显式定义实参：

    ```swift
    let names = ["George Washington", "Martha Washington", "Abe Lincoln"]
    let emails: [String] = names.map { fullname in
      let dottedName = fullname.replacingOccurrences(of: " ", with: ".")
      return dottedName.lowercased() + "@whitehouse.gov"
    }
    ```

#### 捕获列表

- 使用捕获列表来 [解决闭包中的循环引用（Apple）](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/AutomaticReferenceCounting.html#//apple_ref/doc/uid/TP40014097-CH20-ID57)。比如我们经常使用 `[weak self]` 避免在闭包内强引用 `self`，以预防内存泄漏：

    **Correct**

    ```swift
    UserAPI.registerUser(user) { [weak self] result in
      guard let self = self else { return }

      if result.success {
        self.doSomethingWithResult(result)
      }
    }
    ```

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

### 协议

#### 协议一致性

- 当将协议添加到某个类时，应为协议方法使用单独的扩展（extension）。这样可以将相关的方法与协议组合在一起，并可以简化将协议添加到具有其相关方法的类的说明。

- 使用 `// MARK: - SomeDelegate` 注释，以让代码井井有条。

    **Incorrect**

    ```swift
    class MyViewcontroller: UIViewController, UITableViewDataSource, UIScrollViewDelegate {
      // All methods
    }
    ```

    **Correct**

    ```swift
    class MyViewcontroller: UIViewController {
      // ...
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

#### 协议代理

- 可以把 `AnyObject` 添加到协议继承列表，以将代理限制为类（而不是结构体或枚举）。（参考：[Class-Only Protocols（Apple）](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/Protocols.html#//apple_ref/doc/uid/TP40014097-CH25-ID281)）。

  > 按：
  >
  > 以前官方推荐 `class`，Swift 4~5 时期修改为 `AnyObject`。猜测原因大概是 `class` 本身也是一个关键词，为了避免引起困惑？（但是他们怎么才想起来这茬……）

- ~~如果你的协议中有 [Optional Methods（Apple）](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/Protocols.html#//apple_ref/doc/uid/TP40014097-CH25-ID284)，则必须使用 `@objc` 进行声明。~~

  > 按：
  >
  > 我们更推荐使用 `extension protocol` 的方式添加默认实现。

- 将协议声明在使用代理的类中，而不是实现代理的类中。

- 当多个类使用相同的协议时，请在其自己单独的文件中声明它。

- 用 `weak` 修饰 `var delegate` 以避免循环引用。

    ```swift
    // SomeTableCell.swift

    protocol SomeTableCellDelegate: AnyObject {
      func cellButtonWasTapped(cell: SomeTableCell)
    }

    class SomeTableCell: UITableViewCell {
      weak var delegate: SomeTableCellDelegate?
      // ...
    }
    ```

    ```swift
    // SomeTableViewController.swift

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

### Array 和 Dictionary

#### 类型简写语法（Type Shorthand Syntax）

- 如 Apple 在 [Array Type Shorthand Syntax（Apple）](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/CollectionTypes.html#//apple_ref/doc/uid/TP40014097-CH8-ID107) 推荐的，尽量使用类型简写语法（Type Shorthand Syntax）：

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

#### 尾部逗号

- 对于 Array 和 Dictionary 的内容项/键值对，除非字面量非常短，否则应将其拆分为多行。每一项/键值对应以逗号结尾，以方便将来进行插入/编辑（如：`cmd + opt + [` 操作等）。Xcode 能自动处理对齐。

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
      object3 // no trailing comma
    ]

    let aDictionary = ["key1": value1, "key2": value2] // how can you even read that?!
    ```

### 常量

- 代码中不变的数据应定义为常量。例如：用于单元格高度的 `CGFloat` 常量，用于单元格标识符、键名（用于 KVC 和 Dictionary）或 segue 标识符的 `String` 常量等。

- 常量尽量私有到与其相关的文件中。

- 文件级常量必须用 `fileprivate let` 声明。

- 文件级常量必须用首字母大写驼峰式（或大家熟悉的 `k*`，如：`kXXCount`）表示，以表明它们是常量而不是属性。

- 如果常量将在文件外使用，则不能添加 `fileprivate`。

- 如果常量将在模块外使用，则必须将其声明为 `public`（主要用于 Pods 和共享库）。

- 如果在类或结构中声明常量，则必须将其声明为 `static`，以避免在每个实例中声明一个常量。

    ```swift
    // SomeTableCell.swift

    // not declared private since it is used in another file
    let SomeTableCellIdentifier = "SomeTableCell"

    class SomeTableCell: UITableViewCell {
      // ...
    }
    ```

    ```swift
    // ATableViewController.swift

    // declared fileprivate since it isn't used outside this file
    fileprivate let RowHeight: CGFloat = 150.0

    class ATableViewController: UITableViewController {

      // ...

      private func configureTableView() {
        tableView.rowHeight = RowHeight
      }

      func tableView(tableView: UITableView, cellForRowAtIndexPath indexPath: NSIndexPath) -> UITableViewCell {
        return tableView.dequeueReusableCellWithIdentifier(SomeTableCellIdentifier, forIndexPath: indexPath)
      }
    }
    ```

### 函数参数

- 遵循《Swift API Design Guidelines》正确地处理 [函数参数（Function Parameters）（Swift）](https://swift.org/documentation/api-design-guidelines/#parameter-names) 和 [参数标签（Argument Labels）（Swift）](https://swift.org/documentation/api-design-guidelines/#argument-labels)。

### 分号

- 相濡以沫，不如相忘于江湖。
- 尽量不要把多条语句放在同一行上。

### 类型别名（Typealiases）

- 创建 `typealias` 以赋予常用数据类型和闭包以语义含义。

- `typealias` 等效于 C 语言中的 `typedef`，应用于为类型创建别名。

- 可以将 `typealias` 嵌套在与其相关的父类型（类、结构体等）中。

    ```swift
    typealias IndexRange = Range<Int>

    typealias JSONObject = [String: AnyObject]

    typealias APICompletion = (jsonResult: [JSONObject]?, error: NSError?) -> Void

    typealias BasicBlock = () -> Void
    ```

### 控制流

- 对于单条件语句，不要添加括号。

- 为清晰起见或明确操作顺序，推荐在复合条件语句中适当使用括号。

- 当布尔值是可选型时，明确检查是否为 `true` 或 `false`，而不要使用空合（nil-coalescing）运算符：

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

### Switch 语句

- `case` 语句之间不需要 `break`（默认情况下它们不会 fallthrough）

- 在适当的时候，一次可以匹配多个 `case` 值：

    ```swift
    var someCharacter: Character

    // ...

    switch someCharacter {
    case "a", "e", "i", "o", "u":
      print("\(someCharacter) is a vowel")
    // ...
    }
    ```

- 当匹配具有关联值的枚举时，使用 `case .CASENAME (let ...)` 而不是 `case let ...` 语法来进行值绑定。

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

### 隐式 Getter

- 如 Apple 在 [Read-Only Computed Properties（Apple）](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/Properties.html#//apple_ref/doc/uid/TP40014097-CH14-ID259) 中的建议，在只读、计算属性和只读下标上省略 `get` 关键字：

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

### 隐式 `return`

- 在单行闭包、隐式 Getter 和其他闭包中，当开头的 `{`、内部语句和结束的 `}` 都在一行时，省略 `return`。

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

### 循环

- 如果你需要遍历序列（Sequence）并使用索引，使用 `enumerated()` 函数：

    ```swift
    for (index, element) in someArray.enumerated() {
      // ...
    }
    ```

- 需要转换数组时使用 `map`（对于可选数组，使用 `compactMap`；对于 Array 数组使用 `flatMap`）：

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


- 如果你没有执行转换，或者情况不适合时，不要使用 `map`/`flatmap`，改用 `for in` 循环（[Tips](http://www.mokacoding.com/blog/when-to-use-map-flatmap-for/)）。

- 除了简单的一行闭包外，避免使用 `forEach`，它类似于 Objective-C 中的 `makeObjectsPerformSelector:`。

## Shoulds

### 变量声明

- 声明变量时应详细，但不应太详细。

- 使名称具有描述性。但避免重复描述，一个特征描述一次就足够了。

    **Incorrect**

    ```swift
    var someDictionary: Dictionary = [String: String]() // Dictionary is redundant
    var somePoint: CGPoint = CGPoint(x:100, y: 200) // CGPoint is repeated
    var b = Bool(false) // b is not a descriptive name
    var someTextString: String = "" // text and string is verbose
    ```

    **Correct**

    ```swift
    var someArray = [String]()
    var someArray: [String] = []
    var someDictionary = [String: Int]()
    var someDictionary: [String: Int] = [:]
    var countOfCats: UInt32 = 12
    var isMadeOfCheese = false
    var somePoint = CGPoint(x:100, y: 200)
    ```

### 可选型

#### `guard let` vs. `if let`

- 在条件允许时，使用 `guard let` 而不是 `if let` 来提高可读性，并且还可以保持代码的扁平来减少嵌套（参考 [Focusing on the happy-path](http://natashatherobot.com/swift-guard-better-than-if/)）：

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

- 由于 `guard let` 在失败时会退出当前作用域，因此当你在解包失败后仍需继续执行时，使用 `if let` 更适合：

    ```swift
    if let anImage = UIImage(named: ImageNames.background) {
      imageView.image = anImage
    }

    // Do more configuration
    // ...
    ```

### 间距

- 当用链式语句将 3 个或更多的方法链接在一起时，对每个语句换行：

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

- 对于 `guard` 语句，`else {` 可以放在最后一个条件之后的另一行中（不必放与最后一个条件相同的行中）。对于单条件的 `guard`，如果 `else {` 在该行，条件也应该在该行。如果 `guard` 语句的 `else` 子句是一个简单的 `return` 语句，那么他们可以放在一行。参考如下：
  > `guard ... return` 后建议换行，因为代码有退出 `return` 指令，同时也有利于代码阅读。

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

    ```swift
    guard
      let constantOne = valueOne,
      let constantTwo = valueTwo,
      let constantThree = valueThree else { return }
    ```

    **Correct** (⭐️ lots of choices here)

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
      let constantThree = valueThree
    else { return }
    ```

- 当在一个 `switch` 语句中一次匹配多个 `case` 时，除非 `case` 很少并且它们都很短，否则最好将每个 `case` 放在单独一行上，或者适当分组排列。

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

    ```swift
    switch something {
    case .oneLongCase, .anotherLongCase,
         .thereAreMoreCases, .thisIsInASanerPlace:
      return false
    case .sanity:
      return true
    }
    ```

### self

- 如非必要，避免使用 `self`。一般只当你有一个与属性名冲突的局部变量，或者在 `self` 被捕获的上下文中，你才有可能需要使用 `self`。

### 循环

- 如果你有一个 Array 数组，并且想遍历所有内容请考虑使用 `joined(separator:)` 代替嵌套的 `for in` 循环：

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

### 闭包

- 避免在闭包参数周围使用不必要的括号。

    **Incorrect**

    ```swift
    functionWithAClosure { (result) in
      // ...
    }
    ```

    ```swift
    functionWithAClosure { (result) -> Int in
      // ...
    }
    ```

    **Correct**

    ```swift
    functionWithAClosure { result in
      // ...
    }
    ```

    ```swift
    functionWithAClosure { result -> Int in
      // ...
    }
    ```

    ```swift
    functionWithAClosure { (result: String) in
      // ...
    }
    ```

#### 尾随闭包语法

- 当函数或方法的唯一或最后一个参数是闭包并且只有一个闭包参数时，请使用尾随闭包语法。

    ```swift
    // a function that has a completion closure/block
    func registerUser(user: User, completion: (Result) -> Void)
    ```

    **Correct**

    ```swift
    UserAPI.registerUser(user) { result in
      if result.success {
        // ...
      }
    }
    ```

    **Incorrect**

    ```swift
    UserAPI.registerUser(user, completion: { result in
      if result.success {
        // ...
      }
    })
    ```

- 当闭包是唯一的参数时，省略空的括号 `()`。

    **Correct**

    ```swift
    let doubled = [2, 3, 4].map { $0 * 2 }
    ```

    **Incorrect**

    ```swift
    let doubled = [2, 3, 4].map() { $0 * 2 }
    ```

- **注意：** 闭包的参数列表是直接放在开头 `{` 之后，还是放在下一行，取决于个人喜好。但在整个项目中应该保持一致。在我们的编码标准中，一般将其放在同一行上，以节省空间。

### 运算符重载 + 自定义运算符

- 强烈建议不要使用运算符重载和自定义运算符，因为这会损害可读性，并可能给协作项目中的其他开发人员造成较大的困惑。当然在某些情况下这是有必要的，例如：重载 `==` 以符合 `Equatable`。在编写自定义运算符或使现有运算符重载时，运算符函数应调用另一个执行该实际工作的 **显式命名** 的函数。关于此问题的最佳实践和更多讨论，参考 [NSHipster Article（NSHipster）](http://nshipster.com/swift-operators/#guidelines-for-swift-operators)。
  > 按：
  >
  > 运算符重载应同协作项目的其他开发人员一并讨论后实施。

### 元组

- 这个词读起来像 “tuh-ple”

- 与“couple”和“flexible”押韵

- 在创建或分解元组时，尽量命名元组成员，除非该元组作用域非常小且并不影响代码阅读：

    ```swift
    let foo = (something: "cats", somethingElse: 909_099)
    let (something, somethingElse) = foo
    ```

### 常量

- 用 `static let` 声明常量以确保静态存储。

- 最好在相关作用域内声明常量，而不是在 `Constants.swift` 之类的全局共享常量文件中声明“局部”常量。

- 要小心大型常量文件，因为随着时间的推移，它们可能变得难以管理。针对这种情况，可以通过将主常量文件的相关部分重构为单独的文件来缓解。

- 可以使用 `enum` 将相关常量分组。`enum` 的名称应为单数，并且每个常量名都应按驼峰式编写。（使用 `enum` 可避免创建无用的实例）

    ```swift
    enum SegueIdentifier {
      static let onboarding = "onboardingSegue"
      static let login = "loginSegue"
      static let logout = "logoutSegue"
    }

    enum StoryboardIdentifier {
      static let main = "main"
      static let onboarding = "onboarding"
      static let settings = "settings"
    }

    print(SegueIdentifier.login) // "LoginSegue"
    ```

- 在适当的情况下，还可以使用带有 `rawValue` 类型的 `enum` 对常量进行分组，该类型与你需要处理的数据的类型有关。如果你没有为 `rawValue` 显示定义任何值，那么 `String` 类型的 `enum` 将会按 `case` 名隐式为其 `rawValue` 赋值（[Implicitly Assign（Apple）](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/Enumerations.html#//apple_ref/doc/uid/TP40014097-CH12-ID149)）：

    ```swift
    enum UserJSONKeys: String {
      case username
      case email
      case role
      // Explicitly defined rawValue
      case identifier = "id"
      // ...
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

### 默认初始化器（Default Initializers）

- 尽可能使用 [默认初始化器（Default Initializers）（Apple）](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/Initialization.html#//apple_ref/doc/uid/TP40014097-CH18-ID213)。

### 类与结构体

- 大多数自定义数据类型都应该是结构或枚举体。
- 这些情况下，你可能想要使用类：
    - 当你需要引用类型的一些特性时，如数据共享等。
    - 当你需要使用析构函数（deinitializers）来帮助释放资源时。
    - 当你需要运行时类型检查时。
    - 当你需要与 Objective-C 互相操作时。
    - 当你在考虑了所有其他可能性之后认为需要继承时，如：协议、协议继承、具有默认实现的协议扩展、泛型等。

- 更多信息参考 [Swift Programming Language Guidelines（Swift）](https://docs.swift.org/swift-book/LanguageGuide/ClassesAndStructures.html) 和 [Choosing Between Structures and Classes（Apple）](https://developer.apple.com/documentation/swift/choosing_between_structures_and_classes)。

## 提示和技巧

### 参数对齐

- 适当的时候，可以按参数换行然后对齐，以保持更好的可读性：

    **What Xcode Autocompletion Suggests**

    ```swift
    UIView.animate(withDuration: 0, delay: 0, options: []) {
      // ...
    } completion: { _ in
      // ...
    }
    ```

    **Gentle**

    ```swift
    UIView.animate(
      withDuration: 0,
      delay: 0,
      options: []
    ) {
      // ...
    } completion: { _ in
      // ...
    }
    ```