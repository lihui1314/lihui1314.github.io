---
layout: post
title: "SwiftUI之ViewBuilder"
excerpt: "SwiftUI是苹果推出的一种构建用户界面的声明式框架,它采用数据驱动的方式来构建界面"
---
### ViewBuilder

swiftUI是苹果推出的一种构建用户界面的声明式框架,它采用数据驱动的方式来构建界面。学了一段时间SwiftUI，感觉这种写代码的方式还是挺好的。这里主要总结ViewBuilder。以及swift是如何实现swiftUI这种DSL的。

首先看一段简单的代码：

```swift
struct ContentView: View {
    var body: some View {
        Text("Hello, world!")
        Text("Hello, world!")
    }
}
```

思考一个问题，这种写法，swiftUI是怎么把两个Text都加到view上的呢? 其实这里就是用到ViewBuilder的能力。说到ViewBuilder之前，先说一下**@resultBuilder**

#### @resultBuilder 结果构造器

+ 一个结果构建器类型必须满足两个基本要求
  1. 它必须通过`@resultBuilder`进行标，并允许它作为一个自定义属性使用.
  2. 它必须至少实现一个名为`buildBlock`的类型方法
  
简单的说，结果构造起是可以将多个内容构建为一个结果，举个例子。

```swift
struct ContentView: View {
    var body: some View {
        Text("Hello, world!")
        Text("Hello, world!")
            .onAppear {
                getStringArray {
                    "梅西"
                    "德保罗"
                    "恩佐"
                }
            }
    }
}

@resultBuilder struct StringArrayBuilder {
    static func buildBlock(_ items: String...) -> [String] {
        return items
    }
}

func getStringArray(@StringArrayBuilder _ content: ()->[String]) {
    print(content())
}
// ["梅西", "德保罗", "恩佐"]

```

#### ViewBuilder都实现了哪些方法

下面的api可以看到 ViewBuilder 支持了，条件语句还有版本判断（buildLimitedAvailability）。

```swift
@resultBuilder public struct ViewBuilder {

    /// Builds an empty view from a block containing no statements.
    public static func buildBlock() -> EmptyView

    /// Passes a single view written as a child view through unmodified.
    ///
    /// An example of a single view written as a child view is
    /// `{ Text("Hello") }`.
    public static func buildBlock<Content>(_ content: Content) -> Content where Content : View
}

@available(iOS 13.0, macOS 10.15, tvOS 13.0, watchOS 6.0, *)
extension ViewBuilder {

    /// Provides support for “if” statements in multi-statement closures,
    /// producing an optional view that is visible only when the condition
    /// evaluates to `true`.
    public static func buildIf<Content>(_ content: Content?) -> Content? where Content : View

    /// Provides support for "if" statements in multi-statement closures,
    /// producing conditional content for the "then" branch.
    public static func buildEither<TrueContent, FalseContent>(first: TrueContent) -> _ConditionalContent<TrueContent, FalseContent> where TrueContent : View, FalseContent : View

    /// Provides support for "if-else" statements in multi-statement closures,
    /// producing conditional content for the "else" branch.
    public static func buildEither<TrueContent, FalseContent>(second: FalseContent) -> _ConditionalContent<TrueContent, FalseContent> where TrueContent : View, FalseContent : View
}

@available(iOS 14.0, macOS 11.0, tvOS 14.0, watchOS 7.0, *)
extension ViewBuilder {

    /// Provides support for "if" statements with `#available()` clauses in
    /// multi-statement closures, producing conditional content for the "then"
    /// branch, i.e. the conditionally-available branch.
    public static func buildLimitedAvailability<Content>(_ content: Content) -> AnyView where Content : View
}
.....
```

上面我们用一个 SwiftUI 的例子来举例说明 SwiftUI 中 DSL 的运作原理，下一篇文章主要说一下swiftUI的数据驱动。





