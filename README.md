# iOS 编码标准

> 按：
>
> 在整理编码标准时，找到一篇 [vokal.io](https://vokal.io/) 的相关文档：[iOS Team Coding Standards](Coding_Standards_ref.md)（包含在同级目录），发现他们的文档和我们目前的书写习惯重合度极高，因此直接汉化了该文档，再增减了一些和我们目前的习惯有冲突的地方，就酱～

该文档中的标准涵盖了 iOS，macOS，watchOS 和 tvOS 开发工作。

👉 **[Swift 编码标准](Swift.md)**

---

- [Musts](#musts)

    - [代码样式](#代码样式)
    - [命名准则](#命名准则)
        - [变量](#变量)
    - [AppDelegate 的用法](#appdelegate-的用法)
    - [本地化](#本地化)
    - [文档](#文档)
    - [脚本](#脚本)

- [Shoulds](#shoulds)

    - [代码结构](#代码结构)
    - [签名和配置文件](#签名和配置文件)

## Musts

### 代码样式

- **可读性比简洁性更重要。**

    > Readability is always preferred over brevity.
    >
    > ——[《The Swift Programming Language》（Swift）](https://docs.swift.org/swift-book/LanguageGuide/BasicOperators.html)

- 使用两个空格，作为单次缩进的步长。

    - Xcode 设置：

        ```
        Preferences -> Text Editing -> Indentation

        Prefer Indent Using: Spaces
                    Tab Width: 2 spaces
                 Indent Width: 2 spaces
        ```

- 空行不能有占位空格。

    - Xcode 设置：

        ```
        Preferences -> Text Editing -> Editing -> While Editing

        ☑️ Automatically trim trailing whitespace
        ☑️ Including whitespace-only lines
        ```

- 使用 120 个字符作为换行参考，但可读性比控制单行长度更重要。
  
    - Xcode 设置：

        ```
        Preferences -> Text Editing -> Editing

        ☑️ Page guide at column: 120
        ```

- 标点符号后应换行：

    ```
    , . ; : { } ( [ = < > ? ! + - * / % ~ ^ | & == != <= >= += -= *= /= %= ^= |= &= << >> || &&
    ```

- 对于 if 等语句中的复杂条件语句，换行符应紧接在 `||` 或 `&&` 之前，以便条件语句的新行以 `||` 或 `&&` 开头。显而易见，新行必须是前一行的延续，而不是单独的语句。必要时在条件左右插入括号，以满足“断在标点后（break-only-after-a-punctuator）”的要求，并使操作顺序显而易见。

    - Examples:

        ```objc
        if (self.migrationFailureOptions == VOKMigrationFailureOptionNumberOne
            || (self.migrationFailureOptions == VOKMigrationFailureOptionNumberTwo
                && self.isPizzaGreat == YES)
            || self.migrationFailureOptions == VOKMigrationFailureOptionNumberThree) {
            NSLog(@"This if statement is complicated");
            // ...
        }
        ```

- 在算术和赋值运算符，与其他符号之间，保持一个空格：

    ```objc
    NSInteger total = count + 2;
    ```

- 对于 Array 和 Dictionary 的内容项/键值对，除非字面量非常短，否则应将其拆分为多行。每一项/键值对应以逗号结尾，以方便将来进行插入/编辑（如：`cmd + opt + [` 操作等）。Xcode 能自动处理对齐。

    - Examples:

        ```objc
        NSArray *anArray = @[
                            object1,
                            object2,
                            object3,
                            ];

        NSDictionary *aDictionary = @{
                                    @"key1": value1,
                                    @"key2": value2,
                                    };
        ```

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

### 命名准则

- 可参考：[Code Naming Basics for Cocoa（Apple）](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/CodingGuidelines/Articles/NamingBasics.html) 和 [Swift API Design Guidelines: Naming（Swift）](https://swift.org/documentation/api-design-guidelines.html#naming)。

- 名称必须具有描述性。避免使用缩写和首字母缩写词（如：viewController -> vc 等）。代码应该像一篇文章一样可以被正常阅读。

- 如果使用缩写或首字母缩写词，则必须采用众所周知的公认名称的形式（如：ID、URL、JSON 等）。

- 类名必须是描述性的。如（在 Objective-C 中应添加第三方前缀）：

    - View Controllers: `YourViewNameViewController`
    - Subviews: `YourSubViewNameView`
    - Objects: `YourObject`
    - Custom Table View Cells: `YourCellNameCell`

- 对于 Setter 方法，请以单词 set 开头。Getter 方法是返回项，没有前缀。对于所有其他方法，以动词开头，而不是名词或形容词。

    - Incorrect: `getProfile`, `makeProfileSuccess:`, `userTappedPhoto:`
    - Correct: `profile`, `setProfileSuccess:`, `tappedPhoto:`

#### 变量

- 按 [camelCase（Wiki）](https://en.wikipedia.org/wiki/CamelCase) 规则命名变量，第一个单词小写：

    ```objc
     NSInteger numberOfThings;
     ```

- 文件级静态变量（不是局部变量）是唯一的例外。以大写字母开头：

    ```objc
    static dispatch_queue_t SharedSerialWriteQueue;
    ```

- 如果变量的作用域超出单个类，则将其视为全局变量（在 Objective-C 中应添加第三方前缀）。

- 命名由 26 个大写和小写英文字母（A...Z，a...z）和 10 个数字（0...9）组成，避免使用除此以外的其他语种字符，因为它们太利于阅读和理解。不推荐使用 emoji。

- 避免命名 temp 变量。即使变量很简单，也要尽可能地描述。

- 如果必须使用非描述性的变量名，如：tmpXX、data 等，应确保其作用域非常有限，另可提供注释以解释代码块。~~要避免使用通用变量名，例如 'tmp', 'data', 'obj', 'res' 等。~~

### AppDelegate 的用法

除了与 `AppDelegate` 相关的事件（启动 App、关闭 App 并响应 `UIApplicationDelegate` 消息等）以外，请勿将 `AppDelegate` 类用于其他用途。工具方法和全局变量，和 `AppDelegate` 无关，应将其放在相关的类、类别、扩展中。如有必要，可以统一在（除 `AppDelegate` 外的）特定的类或单例或其他上下文中，声明全局变量/常量。

### 本地化

对于用户可见的字符串，始终使用 `NSLocalizedString` 相关方法。即使您的项目从未本地化，`NSLocalizedString` 带来的好处，也超过了它所需的键入成本。更多说明参考 [NSLocalizedstring（NSHipster）](http://nshipster.com/nslocalizedstring/)。

### 文档

- 对于公共方法、属性、类、结构、枚举、函数、类型和常量，提供相关文档。

    - Example for methods

        ```objc
        /**
         *  A concise description of the functionality.
         *
         *  @param title     Describe what value should be passed in.
         *  @param message   Left-align param documentation.
         *  @param number    If there is a default value, explain.
         *
         *  @return Describe the return value of the method.
         */
        - (UILabel *)exampleMethodWithTitle:(NSString *)title
                                    message:(NSString *)message
                                     number:(NSNumber *)number;
        ```

    - Example for others

        - Multi-line

            ```objc
            /**
             *  Describe this property. Why is it public?
             *  Are there default values? What about nil?
             */
            @property (nonatomic, strong) NSArray *exampleArray;
            ```

        - Single-line

            ```objc
            /// Describe this constant.
            FOUNDATION_EXPORT NSInteger const VOKCoffeeConsumptionMultiplier;
            ```

### 脚本

~~iOS 项目中使用的任何 Shell 脚本都必须符合 [Vokal's Shell Script Standards（vokal.io）](https://engineering.vokal.io/Systems/sh.md.html)，包括但不限于运行脚本构建阶段中包含的 Shell 脚本。~~

## Shoulds

### 代码结构

- 将 `IBAction` 方法分组放在 `#pragma mark - IBActions` 或 `// MARK：- IBActions` 标记下

- 在代码中每片逻辑的开头，使用 `#pragma mark - [section]` 或 `// MARK: - [section]` 标记。这样可以使在代码中定位特定方法时更加简单。

- 尽可能将初始化逻辑放在方法的顶部。特殊的，把析构函数放在初始化方法之前（比如要做资源释放时），因为该方法有一定的重要性和唯一性。

- 可以把方法中的不同逻辑用空行隔开，有利于阅读。

### 签名和配置文件

~~在每个目标的构建设置中，代码签名身份应始终是通用的，例如“iPhone Developer”和“Automatic”。这允许构建服务器在构建时选择正确的证书和密钥钥匙串。~~

我们使用了 fastlane [match](https://docs.fastlane.tools/actions/match/) 做签名相关管理，详情查阅相关文档。
