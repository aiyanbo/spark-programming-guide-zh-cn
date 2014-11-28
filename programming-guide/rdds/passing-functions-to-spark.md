# 传递函数到 Spark

Spark 的 API 很大程度上依靠在驱动程序里传递函数到集群上运行。这里有两种推荐的方式：

- [匿名函数 (Anonymous function syntax)](http://docs.scala-lang.org/tutorials/tour/anonymous-function-syntax.html)，可以在比较短的代码中使用。
- 全局单例对象里的静态方法。例如，你可以定义 `object MyFunctions` 然后传递 `MyFounctions.func1`，像下面这样：

```scala
object MyFunctions {
  def func1(s: String): String = { ... }
}

myRdd.map(MyFunctions.func1)
```

注意，它可能传递的是一个类实例里的一个方法引用(而不是一个单例对象)，这里必须传送包含方法的整个对象。例如：

```scala
class MyClass {
  def func1(s: String): String = { ... }
  def doStuff(rdd: RDD[String]): RDD[String] = { rdd.map(func1) }
}
```

这里，如果我们创建了一个 `new MyClass` 对象，并且调用它的 `doStuff`，`map` 里面引用了这个 `MyClass` 实例中的 `func1` 方法，所以这个对象必须传送到集群上。类似写成 `rdd.map(x => this.func1(x))`。

以类似的方式，访问外部对象的字段将会引用整个对象：

```scala
class MyClass {
  val field = "Hello"
  def doStuff(rdd: RDD[String]): RDD[String] = { rdd.map(x => field + x) }
}
```

相当于写成 `rdd.map(x => this.field + x)`，引用了整个 `this` 对象。为了避免这个问题，最简单的方式是复制 `field` 到一个本地变量而不是从外部访问它：

```scala
def doStuff(rdd: RDD[String]): RDD[String] = {
  val field_ = this.field
  rdd.map(x => field_ + x)
}
```