# Chapter 20 - Idioms
This chapter is about trying to share some of the best practices of Scala programming so you can write code in "the Scala way".

Before digging into recipes in this chapter, here are some of Scala's best practices:

### At the application level
- At the big-picture, application-design level, follow the 80/20 rule, and try to write 80% of your applications as pure functions, with a thin layer of other code on top of those functions for things like I/O
- Learn Expression-Oriented Programming (**Recipe 20.3**)
- Use the Actor classes to implement concurrency (**Chapter 13**)
- More behavior from classes into more granular traits. This is best described in the **Scala Stackable Trait Pattern**.

### At the coding level:
- Learn how to write pure functions. At the very least, they simplify testing
- Learn how to pass functions around as variables (**Recipes 9.2 to 9.4**)
- Learn how to use the Scala collections API. Know the most common classes and methods (**Chapter 10 and 11**)
- Prefer immutable code. Use `val`s and immutable collections first (**Recipe 20.2**)
- Drop the `null` keyword. Use the `Option`/`Some`/`None` and `Try`/`Success`/`Failure` instead.
- Use TDD and/or BDD testing tools like ScalaTest and specs2

### Outside the code
- Learn how to use SBT (**Chapter 18**)
- Follow the **Scala Style Guide**

## 20.1 Create Methods with No Side Effects (Pure Functions)
In keeping with the best practices of Functional Programming, you want to write "pure functions".

In general, when writing a function (or method), your goal should be to write it as a pure function. But firstly, what is a pure function? Before we tackle that question, we need to look at another term, *referential transparency*, because it's part of the description of a pure function.

### Referential transparency
An expression is referentially transparent (RT) **if it can be replaced by its resulting value without changing the behavior of the program**. This must be true regardless of where the expression is used in the program.

For example, you have `x + y` and you can define a `val z = x + y` and you can replace all `x + y` with `z` without affecting the result of the program.

### Pure functions
Wikipedia defines it as follows:
1. The function always evaluates to the same result value given the same argument value(s). It cannot depend on any hidden state or value, and it cannot depend on any I/O
2. Evaluation of the result does not cause any semantically observable side effect or output, such as mutation or mutable objects or output to I/O devices.

In *Functional Programing in Scala*:
```
A function f is pure if the expression f(x) is referentially transparent for all referentially transparent values x
```
In summary: a pure function is referentially transparent and has no side effects.

Regarding side effects, there's an observation:

"A telltale sign of a function with side effects is that its result type is Unit"

From these definitions, we can make these statements about pure functions:
1. A pure function is given 1/more input parameters.
2. Its result is based solely off of those parameters and its algorithm. The algorithm will not be based on any hidden state in the class or object it is contained in.
3. It WON'T mutate the parameter it's given.
4. It WON'T mutate the state of its class or object.
5. It doesn't perform any I/O operations, such as reading from/writing to disk, prompting for input, or reading input.

These are some examples of pure functions:
- Mathematical functions (addition, subtraction, multiplication)
- Methods like `split` and `length` on the `String` class.
- The `to*` methods on the `String` class.
- Methods on immutable collections (`map`, `drop`, `take`, `filter`, etc)

The following functions are NOT pure functions:
- Methods like `getDayOfWeek`, `getHour`, or `getMinute`. They return a different value depending on when they are called.
- A `getRandomNumber` function.
- A function that reads or prints output.
- A function that writes to an external data store, or reads from a data store.

### The Java approach
Let's convert the methods in an OOP class into pure functions. The following code shows how you might create a `Stock` class that follows the Java/OOP paradigm.

The following class intentionally has a few flaws. It not only has the ability to store information about a `Stock`, but it can also access the Internet to get the current stock price, and further maintains a list of historical prices for the stock:
```
class Stock (var symbol: String, var company: String,
             var price: BigDecimal, var volume: Long) {

    var html: String = _
    def buildUrl(stockSymbol: String): String = { ... }
    def getUrlContent(url: String): String = { ... }

    def setPriceFromHtml(html: String) { this.price = ... }
    def setVolumeFromHtml(html: String) { this.volume = ... }
    def setHighFromHtml(html: String) { this.high = ... }
    def setLowFromHtml(html: String) { this.low = ... }

    // some DAO-like functionality
    private val _history: ArrayBuffer[Stock] = { ... }
    val getHistory = _history

}
```
Beyond attempting to do many things, from an FP perspective, it has these other problems:
- All of its fields are mutable
- All of the `set` methods mutate the class fields
- The `getHistory` method returns a mutable data structure

### Fixing the problems
The first fix is to separate two concepts that are buried in the class. First, there should be a concept of a `Stock`, where a `Stock` consists only of a `symbol` and `company` name.

You can make this a case class:
```
case class Stock(symbol: String, company: String)
```
Second, at any moment in time there is information related to a stock's performance on the stock market. You can call this data structure a `StockInstance`, and also define it as a case class:
```
case class StockInstance(symbol; String, datetime: String,
                         price: BigDecimal, volume: Long)
```
Going back to the original class, the `getUrlContent` method isn't specific to a stock, and should be moved to a different object, such as a general-purpose `NetworkUtils` object:
```
object NetworkUtils {
  def getUrlContent(url: String): String = { ... }
}
```
This method takes a URL as a parameter and returns the HTML content from that URL.

Similarly, the ability to build a URL from a stock symbol should be moved to an object. Because this behavior is specific to a stock, you'll put it in an object named `StockUtils`:
```
object StockUtils {
  def buildUrl(stockSymbol: String): String = { ... }
}
```
Now, the ability to extract the stock price from the HTML can also be written as a pure function and should be moved into the same object:
```
object StockUtils {
  def buildUrl(stockSymbol: String): String = { ... }
  def getPrice(html: String): String = { ... } // new addition
}
```
All the methods named `set*` in the previous class should be `get*` methods in `StockUtils`:
```
object StockUtils {
  def buildUrl(stockSymbol: String): String = { ... }
  def getPrice(symbol: String, html: String): String = { ... } // changed
  def getVolume(symbol: String, html: String): String = { ... } // new addition
  def getHigh(symbol: String, html: String): String = { ... } // new addition
  def getLow(symbol: String, html: String): String = { ... } // new addition
}
```
The methods `getPrice`, `getVolume`, `getHigh`, and `getLow` are all pure functions: given the same HTML string and stock symbol, they will always return the same values, and they don't have side effects.

Following this thought process, the `date` and `time` are moved to a `DateUtils` object:
```
object DateUtils {
  def currentDate: String = { ... }
  def currentTime: String { ... }
}
```
Now, you create an instance of a `Stock` for the current date and time as a simple series of expressions.

First, retrieve the HTML that describes the stock from a web page:
```
val stock = new Stock("AAPL", "Apple")
val url = StockUtils.buildUrl(stock.symbol)
val html = NetUtils.getUrlContent(url)
```
Once you have the HTML, you can get the desired stock information, get the date, and create the `Stock` instance:
```
val price = StockUtils.getPrice(html)
val volume = StockUtils.getVolume(html)
... // and so on
```
Notice that all of the variables are immutable, and each line is an expression.

The code is simple, so you can eliminate all the intermediate variables, if desired.

As a final note about this example, there's no need for the `Stock` class to maintain a mutable list of stock instances. Assuming that the stock information is stored in a database, you can create a `StockDao` to retrieve data:
```
object StockDao {
  def getStockInstances(symbol: String): Vector[StockInstance] = { ... }
}
```
Though `getStockInstances` isn't a pure function, the `Vector` class is immutable.

### Discussion
A benefit of this coding style is that pure functions are easier to test.

**StockUtils or Stock object?**
The methods that were moved to the `StockUtils` class in the previous examples could be placed in the companion object of the `Stock` class. This means that you can place the `Stock` **class and object** in a file named *Stock.scala*:
```
case class Stock(symbol: String, company: String)

object Stock {
  def buildUrl(stockSymbol: String): String = { ... }
  def getPrice(symbol: String, html: String): String = { ... }
  def getVolume(symbol: String, html: String): String = { ... }
  def getHigh(symbol: String, html: String): String = { ... }
  def getLow(symbol: String, html: String): String = { ... }
}
```

## 20.2 Prefer Immutable Objects
