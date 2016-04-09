翻自 [How to Read a Type Annotation](https://github.com/elm-guides/elm-for-js/blob/master/How%20to%20Read%20a%20Type%20Annotation.md)

# 怎样读懂一个类型注释

如果你来自一些动态类型语言如JavaScript或Ruby，或是一些静态的C家族的语言如Java，那么Elm的类型注释看上去有点奇怪。然而，你一旦读懂他们，他将比像`int strcmp(const char *s1, const char *s2)`之类的更具有意义。虽然对于编译器而言并没有什么暖用（因为编译器有类型推到可以自行计算出来）但是对于你，程序员来说，真真是极好的……

弄明白类型注释一个重要的原因是，当你阅读标准库的函数时，会发现每个函数都有类型注释。类型注释告诉你这个函数需要多少个参数，他们的类型是什么，或者说要传递什么样的函数，他的返回类型是什么。这些信息已经全部都在类型注释当中了。

类型注释一旦搞明白了，写起来是非常容易的。虽然写不写都是可以的，但极力推荐把他协商。类型注释可以
帮助你思考函数应该做些什么，并且可以帮助编译器验证代码。（你知道么？一个过时的注释比彻底没有注释更糟糕，那么好的，类型注释永远不会过时。）此外，如果你想要发布一个第三方的包，你也会需要类型注释。

## 定义

第一件需要知道的事情是关于`:`，他表示“类型是”。

```elm
answer : Int
answer = 42
```

你可以这么读：“answer 的类型是 Int；answer 等于 42”。

常见的基本类型包括`Int`，`Float`，`Bool`和`String`。也可以使用一对类型，他表示一个元组，举个栗子：`(Int, Bool)`。可以扩展到任意多个元素，比如`(Int, Float, Int)`是一个三元素，第一个元素是个`Int`，第二个是`Float`，第三个是`Int`。

```elm
myTuple : (String, Int, Bool)
myTuple = ("the answer", 42, True)
```

不知道你有没有注意到，类型总是首字母大写（或在括号中）

有一种很特殊的类型，他只有一种类型的值。类型和值读做“unit”，写为`()`。Unit经常被用作占位符，通常是我们已经知道这个类型有一个值的时候。

## 函数

`->`用于将函数中参数和返回值的类型分隔开。他有这明显的“到”的意思，例如，`String.length : String -> Int`，很显然，“String.length的类型是String到Int”。只需要像一个句子一样从左往右读就好了。哦，顺便一提，`String.length`意味着`length`函数在`String`模块中。无论何时，大写单词后面跟个点，表示他是个模块，并不是一个类型。

有趣的是当出现多个箭头时，比如`update: Action -> Model -> Model`。这个函数取 Action 和一个 Model 作为参数（按照这个顺序），并且返回一个 Model。或者说“update 有一个 Action 到 Model 到 Model 的类型”。

类型注释真正想要告诉你的其实关于_偏应用_的东东：你可以只给函数一些参数，这样会得到一个函数作为结果。你得到新函数的类型注释是原函数类型注释覆盖左侧之后的一部分。

```elm
example : Model -> Model
example = update someAction
```

其实注释中隐藏了一对括号，我们也可以把他写为：`update : Action -> (Model -> Model)`。

不需要太去在意偏应用或是柯里化什么的，太多新东西了。仅仅把最后一个箭头最为返回值，其他的作为函数的参数就好。

## 高阶函数

就像JavaScript那样，函数可以作为另一个函数的参数传递。（我们已经见到过柯里化是如何让他们返回函数的。）

让我们看下专业定制版`List.map`函数，他取一个函数并且将他应用于列表的每个Float元素，然后返回一个新的Int列表作为结果。

```elm
specialMap : (Float -> Int) -> List Float -> List Int
```

第一个参数需要是一个函数，他去一个 Float 作为参数并且返回一个 Int。当读到这个注释时，你可能会对“Float 到 Int”一脸懵逼，先暂停一下。理解括号的重要性。他不同于`Int -> Float -> List Int -> List Float`，这个取的是两个数字和一个列表，但并不是一个函数。

我们知道`round : Float -> Int`，所以我们可以这么写：

```elm
roundMap : List Float -> List Int
roundMap = specialMap round
```

即便`roundMap`不带有任何参数，他的类型明显就是specialMap调用round后返回函数的类型，这要感谢柯里化。我们同样可以写为`roundMap xs = specialMap round xs`；这只是风格问题。

## 类型变量

如果你看过 List 库，会发现那其实并不是[List.map](http://package.elm-lang.org/packages/elm-lang/core/latest/List#map)的定义。相反，他有着小写的类型名字，那就是类型变量：

```elm
List.map : (a -> b) -> List a -> List b
```

这里的类型变量意味着函数将作用于任何类型的 a 和 b，当我们在知道传递的参数具体是什么类型的时候，才会去固定这些类型变量的值。所以我们可以写为`(Float -> Int)`和一个`List Float`，或是`(String -> Action)`和一个`List String`等等。（比起JavaScript，使用类型变量更像是在做数学运算。）

按照惯例，类型变量从a开始，虽然你可以用的其他小写字母。当有一个以上的类型变量时，偶尔也会使用其他字母或词来帮助理解。例如`Dict k v`告诉我们类型变量表示键和值。一个类型可以有任何的类型变量，但超过两个是很罕见的。

类型变量可以让我们编写泛型代码，如列表和其他容器，可以用来容纳任何类型的值。每个特定的容器只能容纳一个类型，但你可以自由选择。然后可以用`List.map`遍历列表并应用一个函数，无需知道列表中放着些什么。仅仅在每个元素上应用的函数需要知道这些元素的类型。

`List a`表示一个任意类型的列表，那仅仅是`List`又是什么的？他被称为一个类型构造器，但更好的回答是，他真的什么都不是。他并不能独自存在。最好把他想象成是列表的基类，有时会被类型变量替换成一个真正的类型。

## 记录

记录很想JS中的 object @todo。像JS一样，他们使用大括号。但不同于JS，记录的键和值之间使用等号；冒号用于记录的类型。下面是一个简单的记录：

```elm
point : { x : Float, y : Float }
point = { x = 3.2, y = 2.5 }
```

大多数情况下，你需要知道记录的类型。但也有在写函数时用到记录的某些字段，忽略其他字段的情况。

```elm
planarDistance : { a | x : Float, y : Float } -> { b | x : Float, y : Float } -> Float
planarDistance p1 p2 =
  let dx = p2.x - p1.x
      dy = p2.y - p1.y
  in
      sqrt (dx ^ 2 + dy ^ 2)
```

`{a |`表示为一个记录的类型的一部分。