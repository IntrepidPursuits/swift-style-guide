![Intrepid Pursuits](intrepid-logo.png)
# Swift Style Guide

This is the documentation of Intrepid's best practices and style regarding the Swift language.

## Making / Requesting Changes

Feedback and change requests are encouraged!  The current maintainers of this style guide are the developers of [Intrepid](http://www.intrepid.io).  If you have an issue with any of the existing rules and would like to see something changed, please see the [contribution guidelines](CONTRIBUTING.md),
then open a pull request. :zap:

## Goals

Attempt to encourage patterns that accomplish the following goals:

 1. Increased rigor, and decreased likelihood of programmer error
 2. Increased clarity of intent
 3. Aesthetic consistency

## Table Of Contents

* [General](#general)
	* [SwiftLint](#swiftlint)
	* [Whitespace](#whitespace)
	* [Code Grouping](#code-grouping)
	* [Guard Statements](#guard-statements)
* [Classes, Structs, and Protocols](#classes-structs-and-protocols)
	* [Structs vs Classes](#structs-vs-classes)
    * [Protocol Naming](#protocol-naming)
* [Types](#types)
	* [Type Specifications](#type-specifications)
	* [Let vs Var](#let-vs-var)
	* [Parameterized Types](#parameterized-types)
	* [Operator Definitions](#operator-definitions)
	* [Dictionaries](#dictionaries)
	* [Type Inference](#type-inference)
    * [Enums](#enums)
* [Optionals](#optionals)
	* [Force-Unwrapping of Optionals](#force-unwrapping-of-optionals)
	* [Optional Chaining](#optional-chaining)
	* [Implicitly Unwrapped Optionals](#implicitly-unwrapped-optionals)
* [Access Control](#access-control)
	* [Getters](#getters)
	* [Referring to self](#referring-to-self)
	* [Classes final by default](#make-classes-final-by-default)
* [Functions](#functions)
	* [Declarations](#declarations)
    * [Naming](#naming)
	* [Calling](#calling)
* [Closures](#closures)
	* [Closure Specifications](#closure-specifications)
	* [Shorthand](#shorthand)
	* [Trailing closures](#trailing-closures)
	* [Multiple closures](#multiple-closures)
	* [Referring to `self`](#referring-to-self)
	* [Should I use `unowned` or `weak`](#should-i-use-unowned-or-weak)

## General

### SwiftLint

We use the [SwiftLint](https://github.com/realm/SwiftLint) library from Realm to enforce code style practices across our projects. This will be enabled by default for projects using Jenkins Pipelines. There is a `.swiftlint.yml` file [available here](.swiftlint.yml) which is the current general standard for Intrepid projects (used by default for Jenkins). The rules can be changed on a project-by-project basis, but this should be avoided if possible. To use your own SwiftLint file on a project be sure to include the it within the project's directory.

### Whitespace

 * Indent using 4 spaces. Never indent with tabs. Be sure to set this preference in Xcode.
 * End files with a newline.
 * Make liberal use of vertical whitespace to divide code into logical chunks.
 * Don’t leave trailing whitespace.
   * Not even leading indentation on blank lines.

### Code Grouping

Code should strive to be separated into meaningful chunks of functionality.  These larger chunks should be indicated by using the `// MARK: - ` keyword.

Using just `// MARK: section title` inserts the `section title` into the function menu, bolding it, and giving it a unique section icon (see `View Lifecycle`).

![marked section](mark.png)

Adding the dash, `// MARK: - section title`, not only inserts the `section title` into the function menu, bolding it, and giving it a unique section icon but also adds a separator into the function menu (see light gray line above `View Lifecycle`). This makes it easier to identify grouped code.

![marked with dash section](mark-with-dash.png)

When grouping protocol conformance, always use the name of the protocol and only the name of the protocol

##### Like this:
```Swift
// MARK: - UITableViewDelegate
```

###### Not this:
```Swift
// MARK: - UITableViewDelegate Methods
```

###### -- or --

```Swift
// MARK: - Table View Delegate
```

### Guard Statements

Guard statements are meant to be used as early return logic only. They should not be used for regular control flow in place of a traditional control flow statement.

##### Single assignment `guard`
```Swift
guard let value = someMethodThatReturnsOptional() else { return nil }
```

##### Multi assignment `guard`
```Swift
guard
    let strongSelf = self,
    let foo = strongSelf.editing
    else { return }
```

##### Complex Returns `guard`
This is any situation where you'd want to do more work in the guard than just return or throw.
```Swift
guard let value = someMethodThatReturnsOptional() else {
    doSomeNecessaryThing()
    return nil
}
```

##### Complex Returns (Multi-assignment) `guard`
```Swift
guard
    let strongSelf = self,
    let foo = strongSelf.editing
    else {
        doSomeNecessaryThing()
        throw Error.FooUnknown
    }
```

## Classes, Structs, and Protocols

### Structs vs Classes

Unless you require functionality that can only be provided by a class (like identity or deinitializers), implement a struct instead.

Note that inheritance is (by itself) usually _not_ a good reason to use classes, because polymorphism can be provided by protocols, and implementation reuse can be provided through composition.

For example, this class hierarchy:

```swift
class Vehicle {
    let numberOfWheels: Int

    init(numberOfWheels: Int) {
        self.numberOfWheels = numberOfWheels
    }

    func maximumTotalTirePressure(pressurePerWheel: Float) -> Float {
        return pressurePerWheel * Float(numberOfWheels)
    }
}

class Bicycle: Vehicle {
    init() {
        super.init(numberOfWheels: 2)
    }
}

class Car: Vehicle {
    init() {
        super.init(numberOfWheels: 4)
    }
}
```

could be refactored into these definitions:

```swift
protocol Vehicle {
    var numberOfWheels: Int { get }
}

extension Vehicle {
    func maximumTotalTirePressure(pressurePerWheel: Float) -> Float {
        return pressurePerWheel * Float(numberOfWheels)
    }
}

struct Bicycle: Vehicle {
    let numberOfWheels = 2
}

struct Car: Vehicle {
    let numberOfWheels = 4
}
```

**_Rationale:_** Value types are simpler, easier to reason about, and behave as expected with the `let` keyword.

#### Protocol Naming

Protocols that describe _what something is_ should be named as nouns.
```Swift
Collection, ViewDelegate, etc.
```
Protocols that describe the _capability_ of something should be named using the suffixes `able`, `ible`, or `ing`.
```Swift
Equatable, Reporting, Sustainable, etc.
```

## Types

### Type Specifications

When specifying the type of an identifier, always put the colon immediately
after the identifier, followed by a space and then the type name.

```swift
class SmallBatchSustainableFairtrade: Coffee { ... }

let timeToCoffee: NSTimeInterval = 2

func makeCoffee(type: CoffeeType) -> Coffee { ... }

func swap<T: Swappable>(inout a: T, inout b: T) { ... }
```

### Let vs Var

Prefer `let`-bindings over `var`-bindings wherever possible

Use `let foo = …` over `var foo = …` wherever possible (and when in doubt). Only use `var` if you absolutely have to (i.e. you *know* that the value might change, e.g. when using the `weak` storage modifier).

**_Rationale:_** The intent and meaning of both keywords is clear, but *let-by-default* results in safer and clearer code.

A `let`-binding guarantees and *clearly signals to the programmer* that its value is supposed to and will never change. Subsequent code can thus make stronger assumptions about its usage.

It becomes easier to reason about code. Had you used `var` while still making the assumption that the value never changed, you would have to manually check that.

Accordingly, whenever you see a `var` identifier being used, assume that it will change and ask yourself why.

### Parameterized Types

Methods of parameterized types can omit type parameters on the receiving type when they’re identical to the receiver’s.

##### Like this:
```swift
struct Composite<T> {
	…
	func compose(other: Composite) -> Composite {
		return Composite(self, other)
	}
}
```

###### Not this:
```swift
struct Composite<T> {
	…
	func compose(other: Composite<T>) -> Composite<T> {
		return Composite<T>(self, other)
	}
}
```

**_Rationale:_** Omitting redundant type parameters clarifies the intent, and makes it obvious by contrast when the returned type takes different type parameters.

### Operator Definitions

Use whitespace around operators when defining them.

##### Like this:
```swift
func <| (lhs: Int, rhs: Int) -> Int
func <|< <A>(lhs: A, rhs: A) -> A
```

###### Not this:
```swift
func <|(lhs: Int, rhs: Int) -> Int
func <|<<A>(lhs: A, rhs: A) -> A
```

**_Rationale:_** Operators consist of punctuation characters, which can make them difficult to read when immediately followed by the punctuation for a type or value parameter list. Adding whitespace separates the two more clearly.

#### As Part Of Protocol Conformance

When defining an operator to conform to a protocol such as `Equatable`, define it within the scope of the extension or type definition that declared the protocol conformance.

##### Like this:
```swift
extension Person: Equatable {
    static func == (lhs: Person, rhs: Person) -> Bool {
        return lhs.id == rhs.id
    }
}
```

###### Not this:

```swift
extension Person: Equatable { }

func == (lhs: Person, rhs: Person) -> Bool {
    return lhs.id == rhs.id
}
```

**_Rationale:_** Better code organization, easier to tell at a glance which operator belongs to which type.

### Dictionaries

When specifying the type of a dictionary, always leave one space after the colon, and no extra spaces around the brackets.

##### Like this:
```swift
let capitals: [Country: City] = [Sweden: Stockholm]
```

###### Not this:
```swift
let capitals: [Country: City] = [ Sweden: Stockholm ]
```

For literal dictionaries that exceed a single line, newline syntax is preferable:

##### Like this:
```swift
let capitals: [Country: City] = [
    Sweden: Stockholm,
    USA: WashingtonDC
]
```

###### Not this:
```swift
let capitals: [Country: City] = [Sweden: Stockholm, USA: WashingtonDC]
```

### Type Inference

Unless it impairs readability or understanding, it preferable to rely on Swift's type inference where appropriate.

##### Like this:
```Swift
let hello = "Hello"
```

###### Not this:
```Swift
let hello: String = "Hello"
```

This does not mean one should avoid those situations where an explicit type is required.

##### Like this:
```Swift
let padding: CGFloat = 20
var hello: String? = "Hello"
```

**_Rationale:_** The type specifier is saying something about the _identifier_ so
it should be positioned with it.

### Enums

Enum cases should be defined in `camelCase` with leading lowercase letters. This is counter to Swift 2.x where uppercase was preferred.

##### Like This

```Swift
enum Directions {
    case north
    case south
    case east
    case west
}
```

###### Not This

```Swift
enum Directions {
    case North
    case South
    case East
    case West
}
```

**_Rationale:_** Uppercase syntax should be reserved for typed declarations only.

## Optionals

### Force-Unwrapping of Optionals

If you have an identifier `foo` of type `FooType?` or `FooType!`, don't force-unwrap it to get to the underlying value (`foo!`) if possible.

Instead, prefer this:

```swift
if let foo = foo {
    // Use unwrapped `foo` value in here
} else {
    // If appropriate, handle the case where the optional is nil
}
```

Or when unwrapping multiple optionals, prefer this:

```swift
if let foo = foo.optionalProperty as? SomeType,
    let bar = bars.filter({ $0.isMyBar }).first,
    foo.hasBizz,
    bar.hasBazz { // Notice the new line between conditionals and execution code

    foo.bizz()
    bar.bazz()
} else {
    // If appropriate, handle the case where the optionals are nil
}
```

**_Rationale:_** Explicit `if let`-binding of optionals results in safer code. Force unwrapping is more prone to lead to runtime crashes.

### Optional Chaining

Optional chaining in Swift is similar to messaging `nil` in Objective-C, but in a way that works for any type, and that can be checked for success or failure.

Use optional chaining if you don’t plan on taking any alternative action if the optional is `nil`.

```Swift
let cell: YourCell = tableView.ip_dequeueCell(indexPath)
cell.label?.text = “Hello World”
return cell
```

**_Rationale:_** The use of optional binding here is overkill.

### Implicitly Unwrapped Optionals

Implicitly unwrapped optionals have the potential to cause runtime crashes and should be used carefully. If a variable has the possibility of being `nil`, you should always declare it as an optional `?`.

Implicitly unwrapped optionals may be used in situations where limitations prevent the use of a non-optional type, but will never be accessed without a value.

If a variable is dependent on `self` and thus not settable during initialization, consider using a `lazy` variable.

```Swift
lazy var customObject: CustomObject = CustomObject(dataSource: self)
```

**_Rationale:_** Explicit optionals result in safer code. Implicitly unwrapped optionals have the potential of crashing at runtime.

## Access Control

Top-level functions, types, and variables should always have explicit access control specifiers:

```swift
public var whoopsGlobalState: Int
internal struct TheFez {}
private func doTheThings(things: [Thing]) {}
```

However, definitions within those can leave access control implicit, where appropriate:

```swift
internal struct TheFez {
	var owner: Person = Joshaber()
}
```

When dealing with functionality that relies on ObjC systems such as the target-selector pattern, one should still strive for appropriate access control.  This can be achieved through the `@objC` attribute.

##### Like this:
```Swift
@objc private func handleTap(tap: UITapGestureRecognizer)
```

###### Not this:
```Swift
public func handleTap(tap: UITapGestureRecognizer)
```

**_Rationale:_** It's rarely appropriate for top-level definitions to be specifically `internal`, and being explicit ensures that careful thought goes into that decision. Within a definition, reusing the same access control specifier is just duplicative, and the default is usually reasonable.

### Getters

When possible, omit the `get` keyword on read-only computed properties and
read-only subscripts.

##### Like this:
```swift
var myGreatProperty: Int {
	return 4
}

subscript(index: Int) -> T {
    return objects[index]
}
```

###### Not this:
```swift
var myGreatProperty: Int {
	get {
		return 4
	}
}

subscript(index: Int) -> T {
    get {
        return objects[index]
    }
}
```

**_Rationale:_** The intent and meaning of the first version is clear, and results in less code.

### Referring to `self`

When accessing properties or methods on `self`, leave the reference to `self` implicit by default:

```swift
private class History {
	var events: [Event]

	func rewrite() {
		events = []
	}
}
```

Only include the explicit keyword when required by the language—for example, in a closure, or when parameter names conflict:

```swift
extension History {
    init(events: [Event]) {
        self.events = events
    }

    var whenVictorious: () -> Void {
        return {
            self.rewrite()
        }
    }
}
```

**_Rationale:_** This makes the capturing semantics of `self` stand out more in closures, and avoids verbosity elsewhere.

### Make classes `final` by default

Classes should start as `final`, and only be changed to allow subclassing if a valid need for inheritance has been identified. Even in that case, as many definitions as possible _within_ the class should be `final` as well, following the same rules.

**_Rationale:_** Composition is usually preferable to inheritance, and opting _in_ to inheritance hopefully means that more thought will be put into the decision.

## Functions

Specifications around the preferable syntax to use when declaring, and using functions.

### Declarations

With Swift 3, the way that parameter names are treated has changed. Now the first parameter will always be shown unless explicitly requested not to. This means that functions declarations should take that into account and no longer need to use long, descriptive names.

##### Like this:
```Swift
func move(view: UIView, toFrame: CGRect)

func preferredFont(forTextStyle: String) -> UIFont
```

###### Not this:
```Swift
func moveView(view: UIView, toFrame frame: CGRect)

func preferredFontForTextStyle(style: String) -> UIFont
```

If you absolutely need to hide the first parameter name it is still possible by using an `_` for its external name, but this is not preferred.
```Swift
func moveView(_ view: UIView, toFrame: CGRect)
```

**_Rationale:_** Function declarations should flow as a sentence in order to make them easier to understand and reason about.

### Naming

Avoid needless repetition when naming functions. This is following the style of the core API changes in Swift 3.

##### Like this:
```Swift
let blue = UIColor.blue
let newText = oldText.append(attributedString)
```

###### Not this:
```Swift
let blue = UIColor.blueColor()
let newText = oldText.appendAttributedString(attributedString)
```

### Calling

... some specifications on calling functions

- Avoid declaring large arguments inline
- For functions with many arguments, specify each arg on a new line and the `)` on the final line
- Use trailing closure syntax for simple functions
- Avoid trailing closures at the end of functions with many arguments.  (3+)

## Closures

### Closure Specifications

It is preferable to associate a closure's type from the left hand side when possible.

##### Like this:
 ```Swift
 let layout: (UIView, UIView) -> Void = { (view1, view2) in
   view1.center = view2.center
   // ...
 }
 ```

###### Not this:
```Swift
let layout = { (view1: UIView, view2: UIView) in
  view1.center = view2.center
  // ...
}
```

### Void arguments/return types

It is preferable to omit `Void` in closure arguments and return types whenever possible.

##### Like this:
```Swift
let noArgNoReturnClosure = { doSomething() } // no arguments or return types, omit both
let noArgClosure = { () -> Int in return getValue() } // void argument, use '()'
let noReturnClosure = { (arg) in doSomething(with: arg) } // void return type, omit return type
```

###### Not this:
```Swift
let noArgNoReturnClosure = { (Void) -> Void in doSomething() }
let noArgClosure = { (Void) -> Int in return getValue() }
let noReturnClosure = { (arg) -> Void in doSomething(with: arg) }
```
**_Rationale:_** A `Void` return type can be inferred, thus it is unnecessarily verbose to include it

When defining closure type, prefer `()` for parameters, and `Void` for return types.

##### Like this:
```Swift
typealias NoArgNoReturnClosure = () -> Void
typealias NoArgClosure = () -> Int
typealias NoReturnClosure = Int -> Void
```

###### Not this:
```Swift
typealias NoArgNoReturnClosure = (Void) -> ()
typealias NoArgClosure = (Void) -> Int
typealias NoReturnClosure = Int -> ()
```

**_Rationale:_** `Void` is more readable than `()`, especially when wrapped in more parentheses. `() -> Void` is also the standard in Apple APIs and documentation.

### Shorthand

Shorthand argument syntax should only be used in closures that can be understood in a few lines.  In other situations, declaring a variable that helps identify the underlying value is preferred.

##### Like this:
```Swift
doSomethingWithCompletion() { result in
  // do things with result
  switch result {
    // ...
  }
}
```

###### Not this:
```Swift
doSomethingWithCompletion() {
  // do things with result
  switch $0 {
    // ...
  }
}
```

Using shorthand syntax is preferable in situations where the arguments are well understood and can be expressed in a few lines.

```Swift
let sortedNames = names.sort { $0 < $1 }
```
### Trailing Closures

Use trailing closure syntax only if there's a single closure expression parameter at the end of the argument list.

##### Like this:
```swift
UIView.animateWithDuration(1.0) {
  self.myView.alpha = 0
}
```

###### Not this:
```swift
UIView.animateWithDuration(1.0, animations: {
  self.myView.alpha = 0
})
```

### Multiple Closures

When a function takes multiple closures as arguments it can be difficult to read. To keep it clean, use a new line for each argument and avoid trailing closures. If you're not going to use the variable from the closure input, name it with an underscore `_`.

##### Like this:
```swift
UIView.animateWithDuration(
    SomeTimeValue,
    animations: {
        // Do stuff
    },
    completion: { _ in
        // Do stuff
    }
)
```
###### Not this:
```swift
UIView.animateWithDuration(SomeTimeValue, animations: {
    // Do stuff
    }) { complete in
        // Do stuff
}
```
(Even though the default spacing and syntax from Xcode might do it this way)

### Referring to `self`

When referring to `self` within a closure you must be careful to avoid creating a [strong reference cycle](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/AutomaticReferenceCounting.html#//apple_ref/doc/uid/TP40014097-CH20-ID56). Always use a capture list, such as `[weak self]` when it's necessary to use functions or properties outside of the closure.

##### Like this:
```swift
lazy var someClosure: () -> String? = { [weak self] in
    guard let safeSelf = self else { return nil }
    return safeSelf.someFunction()
}
```

##### Not this:
```swift
viewModel.someClosure { self in
    self.outsideFunction()
}
```

**_Rationale_** If a closure holds onto a strong reference to a property being used within it there will be a strong reference cycle causing a memory leak.

### Should I use `unowned` or `weak`?

* `weak` is always preferable as it creates an optional so that crashes are prevented. `unowned` is only useful when you are guaranteed that the value will never be `nil` within the closure. Since this creates the possibility for unsafe access it should be avoided.
