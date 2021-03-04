title: Swift optional type
date: 2016-01-06 22:57:11
category: Swift
tags: OptionalType
---

## 为什么会出现optional类型?
在只有基础数据类型(整形Int、浮点数Double和Float、布尔类型Bool以及字符串类型String)时，Swift语言不会为默认为变量赋初始值，所有的变量必须手动赋值。不初始化就会报错, 例如下面的例子：

```Swift
var plainString: String
//error: Variable used before being initialized
```
但是实际项目中存在需要某些变量先定义，随后在某个时机被初始化的需求。比如说View中的一个输入框`emailTextField`，它的属性`emailTextField.text`的值就只是在用户输入数据后才不为空。

后来曾思考，若都把可能为空的变量初始化为nil，之后再根据需求在不同时机改变它为非空的值，是不是可以解决这个问题？答案是否定的，因为我们随时可能会在这个变量上调用一些方法，但如果当时变量的值凑巧为nil时，就会导致程序crash。

为了解决问题：`定义一个不需要初始化的变量，并且当这个变量在以后的调用过程中可以被安全使用，不论其是否为nil`， Optional类型就顺理成章地出现了。

<!-- more -->

## optional类型是什么？

从Swift源码看，Optional 类型是一个enum（枚举）类型。

```Swift
public enum Optional<Wrapped> : _Reflectable, NilLiteralConvertible {
    case None
    case Some(Wrapped)
    /// Construct a `nil` instance.
    public init()
    /// Construct a non-`nil` instance that stores `some`.
    public init(_ some: Wrapped)
    /// If `self == nil`, returns `nil`.  Otherwise, returns `f(self!)`.
    @warn_unused_result
    public func map<U>(@noescape f: (Wrapped) throws -> U) rethrows -> U?
    /// Returns `nil` if `self` is nil, `f(self!)` otherwise.
    @warn_unused_result
    public func flatMap<U>(@noescape f: (Wrapped) throws -> U?) rethrows -> U?
    /// Create an instance initialized with `nil`.
    public init(nilLiteral: ())
}

```

它有`None`和`Some`两种类型。其实`nil`就是`Optional.None`,非`nil`是`Optional.Some(Wrapped)`,这个`Optional.Some`是一个被包装了的类型，因此当获取它的值的时候，需要unwrap(拆包)。

```Swift
var optionalString: String? = "Hello world"
```
debug时会看到`optionalString`的值为

```
{Some "Hello world"}
```

## optional类型的拆包

与拆包相关的操作符有 `!` 和 `?`

以下有几个例子

```Swift
//Optional type variable should be unwrapped before use. Use ! or ? to unwrap.
var optionalNilString:String?

var optionalNilStringTest1 = optionalNilString?.lowercaseString

//Attention! Force unwrap with ! will lead to crash here! You can unwrap the comment and try
//var optionalNilStringTest2 = optionalNilString！.lowercaseString

var optionalPresentString:String? = "optional present string"

var optionalPresentStringTest1 =

optionalPresentString?.lowercaseString

var optionalPresentStringTest2 = optionalPresentString!.lowercaseString

```

### About ?
如果Optional type variable的值不为nil，则会进行下一步操作，执行后面的方法调用。

### About !
强制拆包，如果Optional type variable的值为nil，就会crash.
