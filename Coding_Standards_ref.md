# iOS Team Coding Standards

> ref: https://engineering.vokal.io/iOS/CodingStandards/README.md.html

The standards in this file cover all iOS, macOS, watchOS, and tvOS work.

There are separate documents for [Objective-C Code Standards](Objective-C.md.html) and [Swift Code Standards](Swift.md.html).

---

- [Musts](#musts)

    - [Code Styling](#code-styling)
    - [Naming Guidelines](#naming-guidelines)
        - [Variables](#variables)
    - [AppDelegate Usage](#appdelegate-usage)
    - [Internationalization](#internationalization)
    - [Documentation](#documentation)
    - [Scripts](#scripts)
    - [Testing](#testing)

- [Shoulds](#shoulds)

    - [Code Organization](#code-organization)
    - [Code Signing and Provisioning Profile Selection](#code-signing-and-provisioning-profile-selection)
    - [Accessibility Best Practices](#accessibility-best-practices)
    - [Scripts](#scripts-1)
    - [App Transport Security](#app-transport-security)

## Musts

### Code Styling

- Use Xcode's automatic indention.

- Try to observe a 120 character wrap margin, but prioritize readability over line length.

- Break lines only after a punctuator:

    ```
    , . ; : { } ( [ = < > ? ! + - * / % ~ ^ | & == != <= >= += -= *= /= %= ^= |= &= << >> || &&
    ```

- For complex conditionals in `if` statements, linebreaks should be immediately before `||` and/or `&&`, so that new lines of the conditional start with the `||` or `&&`. This makes it obvious that the new line must be a continuation of the previous line and not a separate statement. Insert parentheses around conditional parts as necessary to satisfy the break-only-after-a-punctuator requirement and make order of operations obvious.

    - Example:

        ```objc
        if (self.migrationFailureOptions == VOKMigrationFailureOptionNumberOne
            || (self.migrationFailureOptions == VOKMigrationFailureOptionNumberTwo
                && self.isPizzaGreat == YES)
            || self.migrationFailureOptions == VOKMigrationFailureOptionNumberThree) {
            CDLog(@"This if statement is complicated");
            [...]
        }
        ```

- Put one space between arithmetic and assignment operators and other symbols:

    ```objc
    NSInteger total = count + 2;
    ```

- For array and dictionary literals, unless the literal is very short, it should be split into multiple lines, with the opening symbols on their own line, each item or key-value pair on its own line, and the closing symbol on its own line. The last item or key-value pair should have a trailing comma to facilitate future insertion/editing. Xcode will handle alignment sanely.

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

### Naming Guidelines

- See also: [Code Naming Basics for Cocoa](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/CodingGuidelines/Articles/NamingBasics.html) and [Swift API Design Guidelines: Naming](https://swift.org/documentation/api-design-guidelines.html#naming).

- Names must be descriptive. Avoid abbreviations and acronyms. You should be able to read the code out loud, over the phone.

- If an abbreviation or acronym is used, it must be in the form of an accepted name that is generally well-known.

- Class names must be descriptive. In particular (don't forget to add a three-letter prefix for Objective-C):

    - View Controllers: `YourViewNameViewController`
    - Subviews: `YourSubViewNameView`
    - Objects: `YourObject`
    - Custom Table View Cells: `YourCellNameCell`

- For setter methods, lead with the word set. Getter methods are the item being returned, with no prefix. For all other methods lead with a verb, not a noun or adjective.

    - Incorrect: `getProfile`, `makeProfileSuccess:`, `userTappedPhoto:`
    - Correct: `profile`, `setProfileSuccess:`, `tappedPhoto:`

#### Variables

- Write variable names in [camelCase](https://en.wikipedia.org/wiki/CamelCase), with the first word in lower case:

    ```objc
     NSInteger numberOfThings;
     ```

- File-level static variables (not local ones) are the only exception. Start them with a capital letter:

    ```objc
    static dispatch_queue_t SharedSerialWriteQueue;
    ```

- If a variable's scope exceeds a single class, treat it like a global variable (and prefix it with the project's three letter prefix in Objective-C).

- Form names from the 26 upper and lower case English letters (A...Z, a...z) and the 10 digits (0...9). Avoid use of international characters because they may not read well or be understood everywhere. Do not use emoji.

- Avoid temp variables. Be as descriptive as possible even if the variable does something very simple.

- If you have to use a non-descriptive variable name, keep a very limited scope and provide comments that explain what that specific block of code is doing. Especially avoid generic variable names like 'tmp', 'data', 'obj', 'res', etc.

### AppDelegate Usage

Do not use the `AppDelegate` class for anything except `AppDelegate`-related activities (launching the app, closing the app, and responding to `UIApplicationDelegate` messages). Utility methods and global variables do _not_ belong in the `AppDelegate`. Put utility methods in their own classes or categories/extensions. If necessary, global variables/constants can be exported/exposed from a specific class or singleton or other context—_anything_ but in the `AppDelegate`.

### Internationalization

Always use `NSLocalizedString` for user-facing strings. Even if your project is never localized, the advantages of `NSLocalizedString` outweigh the few extra keystrokes required. NSHipster [provides more background information](http://nshipster.com/nslocalizedstring/).

### Documentation

- Provide documentation of public methods, properties, classes, structs, enums, functions, types, and constants.

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

### Scripts

Any shell scripts used in an iOS project must conform to [Vokal's Shell Script Standards](../../Systems/sh.md.html), including but not limited to shell scripts contained within run script build phases.

### Testing

See the [main testing document](../../TESTING.md.html) for general guidelines regarding test coverage. Since no reliable and consistent UI testing framework exists for iOS at this time, we do not have a minimum percentage for test coverage of iOS apps; the last thing we want to encourage is test cases that exist solely to fulfill a quota without adding any value. Your tests should cover every part of the app that can reliably be tested, including UI elements. We recommend moving logic out of view controllers where possible, to make it easier to unit test that logic.

[CVR](https://cvr.vokal.io/repos) has a per-project _Minimum Passing Line Percent Coverage_ option that can be set low when a project begins, but that minimum should gradually climb as the app is built and tests are added. For example, if the app reaches 72% test coverage, that should be the new minimum on CVR: unless you're removing tests that have proven to be inconsistent, the test coverage rate on an app should not fall.

Here are some examples of cases where test coverage can be missing:

- `prepareForInterfaceBuilder` methods. These are only called by Interface Builder, so they never actually run in the app, whether testing or not.
- Animations. There's just not a good way to test these.
- Flaky UI tests. An inconsistent test is arguably worse than no test, and KIF is notoriously flaky about certain things. If a UI test is failing for different reasons or only part of the time, it should probably be removed, or at least reduced to a test that will perform reliably.
- Push notifications. You can (and should) build notification handling in a way that will allow you to test that handling by passing a mock payload, but the app delegate methods won't be called in unit tests.

When you create a new project using our [project template](https://github.com/vokal/Xcode-Template), testing targets are created for unit and UI tests. Both sets of tests are run on [Travis](../Fastlane-Travis-CI.md.html) for every pull request and merge build.

## Shoulds

### Code Organization

- Group `IBAction` methods together under a `#pragma mark - IBActions` or `// MARK: - IBActions`

- Use `#pragma mark - [section]` or `// MARK: - [section]` to mark the beginning of each logical group throughout the code. Doing this will make searching through the code for a specific method much simpler.

- Keep initialization logic at the top of the method when possible.

- Use vertical spacing to group similar logic in a method.

### Code Signing and Provisioning Profile Selection

In each target's Build Settings, the Code Signing Identities should always be generic, such as "iPhone Developer" and "Automatic". This allows the build server to select the right cert and key keychain at build time.

### Accessibility Best Practices

Accessibility is key both for UI testing and for VoiceOver, which reads accessibility labels out loud to blind users.

- Generally, elements which have text on them (buttons, labels, segmented controls) will use that text as the accessibility label. Double check that you actually _need_ to add an accessibility label before going to the trouble of adding one.
- Any string being set as an accessibility label is a user-facing string to a blind person, and should therefore be localized.
- Avoid setting Accessibility labels directly in Interface Builder, because this almost guarantees they will not be localized. Use an `IBOutlet` to your control and set them programmatically instead.
- Accessibility _identifiers_ are primarily for testing purposes. These are **NOT** read out loud to blind users, and therefore do not need to be localized.
- Use best naming practices for Accessibility labels to give blind users the best experience:
    - Very briefly describe the element. For example: Add, Delete, Favorites, or Volume.
    - Do **not** include the type of the control or view (Button, TextField, etc). This information is included automatically.
    - Begin the label string with a capitalized word. This helps VoiceOver read the label with the appropriate inflection.
    - Do not end the label string with a period. The label is not a sentence, so it should not end with a period.

### Scripts

Run script phases should be renamed to more specifically describe what the phase does. For any run script phase script beyond a couple lines, it is preferable to have the body of the run script phase call out to an external file, as it's easier to edit an external file and the diffs are easier to handle (Xcode mashes the whole body of a run script phase onto a single line in the project file).

### App Transport Security

In iOS 9, Apple introduced [App Transport Security](https://developer.apple.com/library/ios/documentation/General/Reference/InfoPlistKeyReference/Articles/CocoaKeys.html#//apple_ref/doc/uid/TP40009251-SW33) to encourage secure networking. In practice, this means that HTTP (as opposed to HTTPS) requests are blocked unless the app requests an exception to allow them. Starting January 1, 2017, secure connections will be required for apps in the App Store.

Exceptions for ATS can be put in the `Info.plist`, both for servers we control and for third-party servers that we don't. Such exceptions can be acceptable for pulling data from third-party servers, but the exceptions for domains under our control must be justified when the app goes to App Store review.

Unless you have a really, really good reason, do not use ATS exceptions.