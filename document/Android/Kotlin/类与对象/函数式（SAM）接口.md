# 函数式（SAM）接口

只有一个抽象方法的接口称为*函数式接口* 或 *单一抽象方法（SAM）*接口。函数式接口可以有多个非抽象成员，但只能有一个抽象成员。

可以用 `fun` 修饰符在 Kotlin 中声明一个函数式接口。

```kotlin
fun interface KRunnable {
   fun invoke()
}
```

## SAM 转换

对于函数式接口，可以通过 [lambda 表达式](https://book.kotlincn.net/text/lambdas.html#lambda-%E8%A1%A8%E8%BE%BE%E5%BC%8F%E4%B8%8E%E5%8C%BF%E5%90%8D%E5%87%BD%E6%95%B0)实现 SAM 转换，从而使代码更简洁、更有可读性。

使用 lambda 表达式可以替代手动创建实现函数式接口的类。 通过 SAM 转换， Kotlin 可以将签名与接口单个方法的签名匹配的任何 lambda 表达式转换为代码，从而动态实例化接口实现。

例如，有这样一个 Kotlin 函数式接口：

```kotlin
fun interface IntPredicate {
   fun accept(i: Int): Boolean
}
```

如果不使用 SAM 转换，那么你需要像这样编写代码：

```kotlin
// 创建一个类的实例
val isEven = object : IntPredicate {
   override fun accept(i: Int): Boolean {
       return i % 2 == 0
   }
}
```

通过利用 Kotlin 的 SAM 转换，可以改为以下等效代码：

```kotlin
// 通过 lambda 表达式创建一个实例
val isEven = IntPredicate { it % 2 == 0 }
```

可以通过更短的 lambda 表达式替换所有不必要的代码。

```kotlin
fun interface IntPredicate {
   fun accept(i: Int): Boolean
}

val isEven = IntPredicate { it % 2 == 0 }

fun main() {
   println("Is 7 even? - ${isEven.accept(7)}")
}
```

你也可以使用 [Java 接口上的 SAM 转换](https://book.kotlincn.net/text/java-interop.html#sam-%E8%BD%AC%E6%8D%A2)。

## 从具有构造函数的接口迁移到函数接口

从 1.6.20 开始，Kotlin 支持对函数式接口构造函数的 [callable references](https://book.kotlincn.net/text/reflection.html#%E5%8F%AF%E8%B0%83%E7%94%A8%E5%BC%95%E7%94%A8)，这增加了一种源代码兼容的方式，可以从具有构造函数的接口迁移到函数式接口。请考虑以下代码：

```kotlin
interface Printer { 
    fun print() 
}

fun Printer(block: () -> Unit): Printer = object : Printer { override fun print() = block() }
```

启用对函数式接口构造函数的可调用引用后，此代码可以仅替换为函数式接口声明：

```kotlin
fun interface Printer { 
    fun print()
}
```

它的构造函数将被隐式创建，任何使用`::P rinter`函数引用的代码都将被编译。例如：

```lang-kotlin
documentsStorage.addPrinter(::Printer)
```

通过使用 [`@Deprecated`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-deprecated/) 注释和 `DeprecationLevel.HIDDEN` 标记旧函数 `Printer` 来保持二进制兼容性：

```kotlin
@Deprecated(message = "Your message about the deprecation", level = DeprecationLevel.HIDDEN)
fun Printer(...) {...}
```

## 函数式接口与类型别名比较

您也可以简单地使用函数类型的 [type alias](https://book.kotlincn.net/text/type-aliases.html) 重写上述内容：

```kotlin
typealias IntPredicate = (i: Int) -> Boolean

val isEven: IntPredicate = { it % 2 == 0 }

fun main() {
   println("Is 7 even? - ${isEven(7)}")
}
```

然而，函数式接口和[类型别名](https://book.kotlincn.net/text/type-aliases.html)用途并不相同。 类型别名只是现有类型的名称------它们不会创建新的类型，而函数式接口却会创建新类型。 您可以提供特定于特定功能接口的扩展，使其不适用于普通函数或其类型别名。

类型别名只能有一个成员，而函数式接口可以有多个非抽象成员以及一个抽象成员。 函数式接口还可以实现以及继承其他接口。

函数式接口比类型别名更灵活并且提供了更多的功能, 但是，它们在语法上和运行时的成本都可能更高，因为它们可能需要转换为特定接口。当您选择要在代码中使用哪一个时，请考虑您的需求：

* 如果您的 API 需要接受具有某些特定参数和返回类型的函数（任何函数），请使用简单的函数类型或定义类型别名，以便为相应的函数类型提供更短的名称。
* 如果您的 API 接受比函数更复杂的实体（例如，它具有无法用函数类型的签名表示的非平凡的协定和/或操作），请为其声明一个单独的函数接口。
