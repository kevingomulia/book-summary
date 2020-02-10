# Chapter 13 - Actors and Concurrency - Part II

## 13.7 Shutting down the Akka Actor System
You can call the `shutdown` method on your `ActorSystem` instance:
```
object Main extends App {
  // create the ActorSystem
  val system = ActorSystem("HelloSystem")

  // put your actors to work here ...

  // shut down the ActorSystem when the work is done
  system.shutdown
}
```
If the system.shutdown is not called, your application will continue to run indefinitely.

## 13.8 Monitoring the Death of an Actor
Use the `watch` method of an actor's `context` object to declare that the actor should be notified when an actor it is monitoring is stopped.

Here, the `Parent` actor creates an actor instance named kenny, and then declares that it wants to “watch” kenny:

```
class Parent extends Actor {
val kenny = context.actorOf(Props[Kenny], name = "Kenny")
context.watch(kenny)
// more code here ...
```
If `kenny` is killed or stopped by any means, the `Parent` actor is sent a `Terminated(kenny)` message.

**Note:** If `kenny` throws an exception, this doesn't kill it. Instead it will be restarted.

### Looking up actors
This is one way to look up an actor:
```
val kenny = system.actorSelection("/user/Parent/Kenny")
```
You look up actors with the `actorSelection` method, and you can specify a full path to the actor as shown  above. The `actorSelection` method is available on an `ActorSystem` instance, and on the `context` object in an `Actor` instance.

You can also look up actors using a relative path. If `kenny` had a sibling actor, it could have looked up `kenny` using its own `context`, like such:

```
// in a sibling actor
val kenny = context.actorSelection("../Kenny")
```

## 13.9 Simple Concurrency with Futures
A *future* gives you a way to run an algorithm concurrently. A *future* starts running concurrently when you create it and returns a result at some point in the future.

The following examples show a variety of ways to create futures and work with their eventual results:

### Run one task, but block
This shows how to create a future and then block to wait for its result. Blocking is not a good thing - you should only block if you really have to.

The following code performs the calculation `1 + 1` at some time in the future. When it is finished, it returns its result.:

```
package actors

// 1 - the imports
import scala.concurrent.{Await, Future}
import scala.concurrent.duration._
import scala.concurrent.ExecutionContext.Implicits.global

object Futures1 extends App {
  // used by 'time' method
  implicit val baseTime = System.currentTimeMillis

  // 2 - create a Future
  val f = Future {
    sleep(500)
    1 + 1
  }

  // 3 - this is blocking (blocking is bad)
  val result = Await.result(f, 1 second)
  println(result)

  sleep(1000)
}
```
- The `ExecutionContext.Implicits.global` import statement imports the "default global execution context". You can think of an *execution context* as being a thread pool, and this is a simple way to get access to a thread pool.
- A `Future` is created after the second comment. To create a future, you just pass it a block of code you want to run. This is the code that will be executed at some point in the future.
- The `Await.result` method call declares that it will wait for up to one second for the `Future` to return. If the `Future` doesn't return within that time, it throws a `Java.util.concurrent.TimeoutException`
- The `sleep` statement at the end of the code is used so the program will keep running while the `Future` is off being calculated. You won't need this in a real-world program.

### Run one task, but don't block - use callback
A better approach is to use callback methods. There are 3 callback methods: `onComplete`, `onSuccess`, and `onFailure`. This snippet shows how to use `onComplete`:

```
...
val f = Future {
  sleep(Random.nextInt(500))
  42
}

f.onComplete {
  case Success(value) => println(s"Got the callback = $value")
  case Failure(e) => e.printStackTrace
}
... // do your work here
```
The `f.onComplete` method call sets up the callback. Whenever the `Future` completes, it makes a callback to `onComplete`, at which time that code will be executed.

Since the `Future` is running concurrently somewhere, the output is nondeterministic.

### onSuccess and onFailure callback methods
There might be times where you don't want to use `onComplete`, then you can use `onSuccess` or `onFailure`, and handle their expected conditions only.

### Creating a method to return a Future[T]
The following example defines a method named `longRunningComputation` that returns a `Future[Int]`

```
import ...

object Futures2 extends App {
  implicit val baseTime = System.currentTimeMillis

  def longRunningComputation(i: Int): Future[Int] = future {
    sleep(100)
    i + 1
  }

  // this does not block
  longRunningComputation(11).onComplete {
    case Success(result) => println(s"result = $result")
    case Failure(e) => e.printStackTrace
  }
}

```
The `future` method shown in this example is another way to create a future. It starts the computation asynchronously and returns a `Future[T]` that will hold the result of the computation. This is a common way to define methods that return a future.

### Run multiple things; something depends on them; join them together
You may occasionally want to run several operations concurrently, wait for them all to complete, and then do something with their combined results.

The following example shows how to call an algorithm that may be running in the cloud. It makes 3 calls to `Cloud.runAlgorithm`, which is defined elsewhere to return a `Future[Int]`.

The code starts with 3 futures running, and then joins them back together in for the comprehension:

```
import ...

object RunningMultipleCalcs extends App {

  println("starting futures")
  val result1 = Cloud.runAlgorithm(10)
  val result2 = Cloud.runAlgorithm(20)
  val result3 = Cloud.runAlgorithm(30)

  println("before for-comprehension")
  val result = for {
    r1 <- result1
    r2 <- result2
    r3 <- result3
  } yield (r1 + r2 + r3)

  println("before onSuccess")
  result onSuccess {
    case result => println(s"total = $result")
  }

  println("before sleep at the end")
  sleep(2000) // keep the jvm alive
}
```
Here's how this code works:
- The three calls to `Cloud.runAlgorithm` create the `result1`, `result2`, and `result3` variables, which are of type `Future[Int]`
- The for comprehension is used as a way to join the results back together. When all 3 futures return, their `Int` values are assigned to the variables `r1`, `r2`, and `r3`, and the sum of those three values is returned from the `yield` expression, and assigned to the result variable.
- Note that `result` can't just be printed after the `for` comprehension. That's because the `for` comprehension returns a new future, so `result` has the type `Future[Int]`. Hence, the correct way to print the example is with the `onSuccess` method call.
- This code is nondeterministic

### A future and ExecutionContext
The following statements describe the basic concepts of a future, as well as the `ExecutionContext` that a future relies on.
- A `Future[T]` is a container that runs a computation concurrently, and at some future time may return either: a result, or an exception
- Computation of your algorithm starts at some nondeterministic time after the future is created, running on a thread assigned to it by the execution context.
- The result of the computation becomes available once the future completes.
- An `ExecutionContext` executes a task it's given.

### Callback methods
- Callback methods are called asynchronously when a future completes.
- The callback methods are `onComplete`, `onSuccess`, `onFailure`
- A callback is executed by some thread some time after the future is completed.
- The order in which callbacks are executed is not guaranteed.
- `onComplete` takes a callback function of type `Try[T] => U`
- `onSuccess` and `onFailure` take partial functions, you only need to handle the desired case
- `onComplete`, `onSuccess`, and `onFailure` have the result type Unit, so they can’t be chained.

### For comprehensions (combinators: map, flatMap, filter, foreach, recoverWith, fallbackTo, andThen)
When you need to run multiple computations in parallel and join the results together when they're finished running, use *combinators* to provide a more concise and readable code.

The `recover`, `recoverWith`, and `fallbackTo` combinators provide ways of handling failure with futures.

## 13.10 Sending a Message to an Actor and Waiting for a Reply
Use the `?` or `ask` methods to send a message to an Akka actor and wait for a reply, as demonstrated:

```
case object AskNameMessage

class TestActor extends Actor {
  def receive = {
    case AskNameMessage => // respond to the 'ask' request
    sender ! "Fred"
    case _ => println("that was unexpected")
  }
}

object AskTest extends App {

  // create the system and actor
  val system = ActorSystem("AskTestSystem")
  val myActor = system.actorOf(Props[TestActor], name = "myActor")

  // (1) this is one way to "ask" another actor for information
  implicit val timeout = Timeout(5 seconds)
  val future = myActor ? AskNameMessage
  val result = Await.result(future, timeout.duration).asInstanceOf[String]
  println(result)

  // (2) a slightly different way to ask another actor for information
  val future2: Future[String] = ask(myActor, AskNameMessage).mapTo[String]
  val result2 = Await.result(future2, 1 second)
  println(result2)

  system.shutdown
}
```

Both the `?` or `ask` methods use the `Future` and `Await.result` approach demonstrated in Recipe 13.9. The recipe is:
1. Send a message to an actor using either `?` or `ask` instead of the `!` method.
2. The `?` or `ask` methods create a `Future`, so you use `Await.result` to wait for the response from the other actor
3. The actor that's called should send a reply back using the `!` method, as shown in the example, where the `TestActor` receives the `AskNameMessage` and returns an answer using `sender ! "Fred"`.
