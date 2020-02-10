# Chapter 13 - Actors and Concurrency - III

## 13.11 Switching Between Different States with `become`
You want a simple mechanism to allow an actor to switch between the different states it can be in at different times.

You use the Akka "become" approach. To do this, first define the different possible states the actor can be in.

Then, in the actor's `receive` method, switch between the different states based on the messages it receives.

The following example shows how the actor named `DavidBanner` might switch between its `normalState` and its `angryState`:
```
case object ActNormalMessage
case object TryToFindSolution
case object BadGuysMakeMeAngry

class DavidBanner extends Actor {
  import context._

  def angryState: Receive = {
    case ActNormalMessage =>
      println("Phew, I'm back to being David.")
      become(normalState)
  }

  def normalState: Receive = {
    case TryToFindSolution =>
      println("Looking for solution to my problem ...")
    case BadGuysMakeMeAngry =>
      println("I'm getting angry...")
      become(angryState)
  }

  def receive = {
    case BadGuysMakeMeAngry => become(angryState)
    case ActNormalMessage => become(normalState)
  }
}

object BecomeHulkExample extends App {
  val system = ActorSystem("BecomeHulkExample")
  val davidBanner = system.actorOf(Props[DavidBanner], name = "DavidBanner")
  davidBanner ! ActNormalMessage // init to normalState
  davidBanner ! TryToFindSolution
  davidBanner ! BadGuysMakeMeAngry
  Thread.sleep(1000)
  davidBanner ! ActNormalMessage
  system.shutdown
}
```
1. The `davidBanner` actor instance is created
2. The `davidBanner` instance is sent the `ActNormalMessage` to set an intiial state
3. After sending `davidBanner` a `TryToFindSolution` message, it ends a `BadGuysMakeMeAngry` message
4. When `davidBanner` receives the `BadGuysMakeMeAngry` message, it uses `become` to switch to the `angryState`
5. In the `angryState` the only message `davidBanner` can process is the `ActNormalMessage`
6. When `davidBanner` receives the final `ActNormalMessage`, it switches back to the `normalState` using the `become` method.

In summary, the general recipe for using the `become` approach to switch between different possible states is:
1. Define the different possible states
2. Define the `receive` method in the actor to switch to the different states based on the messages it can receive.

## 13.12 Using Parallel Collections
When creating a collection, use one of the Scala's parallel collection classes, or convert an existing collection to a parallel collection.

First, create a sequential collection, such as a `Vector`:
```
scala> val v = Vector.range(0, 10)
v: scala.collection.immutable.Vector[Int] = Vector(0, 1, 2, 3, 4, 5, 6, 7, 8, 9)
```
When you print it, it will always print `0123456789`

Now, call the `par` method on your collection to turn it into a parallel collection:
```
scala> v.par.foreach(print)
5678901234
```
You can also create a parallel collection directly:
```
scala> import scala.collection.parallel.immutable.ParVector
import scala.collection.parallel.immutable._

scala> val v = ParVector.range(0, 10)
v: scala.collection.parallel.immutable.ParVector[Int] =
    ParVector(0, 1, 2, 3, 4, 5, 6, 7, 8, 9)
```
Here's a list of some of the immutable parallel collection classes:
```
ParHashMap
ParHashSet
ParIterable
ParMap
ParRange
ParSeq
ParSet
ParVector
```
### When are parallel collections useful?
Remember that any algorithm that depends on the collection elements being received in a predictable order should not be used with a parallel collection. This means that algorithms like `sum`, `max`, `min`, `mean`, and `filter` will all work fine.

### Performance
Using parallel collections won't always make your code faster.

For a parallel algorithm to provide a benefit, a collection usually needs to be fairly large.

The documentation states:
> “As a general heuristic, speed-ups tend to be noticeable when the size of the collection is large, typically several thousand elements.”

Finally, if using a parallel collection won’t solve your problem, using Akka actors and futures can give you complete control over your algorithms.
