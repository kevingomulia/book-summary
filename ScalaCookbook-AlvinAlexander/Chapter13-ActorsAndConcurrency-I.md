# Ch 13 - Actors and Concurrency

In Scala, the Actor model is the preferred approach for concurrency. Actors give you the benefit of offering a high level of abstraction for achieving concurrency and parallelism.

Beyond that, the Akka actor library adds these additional benefits:
- Lightweight, event-driven processes.
- Fault tolerance. Akka actors can be used to create self healing systems [](http://letitcrash.com)
- Location transparency. Akka actors can span multiple JVMs and servers; they're designed to work in a distributed environment using pure message passing

I like to think of an actor as being like a web service on someone else’s servers that I can’t control. I can send messages to that web service to ask it to do something, or I can query it for information, but I can’t reach into the web service to directly modify its state or access its resources; I can only work through its API, which is just like sending immutable messages. In one way, this is a little limiting, but in terms of safely writing parallel algorithms, this is very beneficial.

### The Actor Model
The concept of an *actor*:
- An actor is the smallest unit when building an actor-based system (similar to an object in an OOP system)
- Like an object, an actor encapsulates state and behavior.
- You can't peek inside an actor to get its state. You can instead send an actor a message requesting state information.
- An actor has a mailbox, and its purpose is to process the messages in that mailbox.
- You communicate with an actor by sending it an immutable message. These messages go into the actor's mailbox.
- When an actor receives a message, it opens the message, processes it, then moves on to the next message in the mailbox.

In an application, actors form hierarchies:
- An actor has one parent: the actor that created it.
- An actor may have children, and siblings.
- A best practice of developing actor system is to delegate tasks, especially if behavior will block.

A final piece of the Actor model is handling failure. When an exception is thrown, an actor suspends itself and all of its children, and sends a message to its supervisor, signaling that a failure has occurred.

The supervisor has 4 choices:
1. Resume the subordinate, keeping its internal state
2. Restart the subordinate, giving it a clean state
3. Terminate the subordinate
4. Escalate the failure

In addition to those general statements about actors, there are a few important things to know about Akka’s implementation of the Actor model:
- You can’t reach into an actor to get information about its state. When you instantiate an Actor in your code, Akka gives you an `ActorRef`, which is essentially a façade between you and the actor.
- Behind the scenes, Akka runs actors on real threads; many actors may share one
thread.
- There are different mailbox implementations to choose from, including variations of unbounded, bounded, and priority mailboxes. You can also create your own mailbox type.
- Akka does not let actors scan their mailbox for specific messages.
- When an actor terminates (intentionally or unintentionally), messages in its mailbox go into the system’s “dead letter mailbox

## 13.1 Getting Started
Create an actor by extending the `akka.actor.Actor` class and write a `receive` method in your class. The `receive` method should be implemented with a `case` statement that allows the actor to respond to different messages.

For example, we are defining an actor that responds when it receives the `String` literal `hello` as a message. Then, save the following code to a file named `Hello.scala` in the root directory of your SBT project.

```
import akka.actor.Actor
import akka.actor.ActorSystem
import akka.actor.Props

class HelloActor extends Actor {
    def receive = {
    case "hello" => println("hello back at you")
    case _ => println("huh?")
  }
}

object Main extends App {
  // an actor needs an ActorSystem
  val system = ActorSystem("HelloSystem")

  // create and start the actor
  val helloActor = system.actorOf(Props[HelloActor], name = "helloactor")

  // send the actor two messages
  helloActor ! "hello"
  helloActor ! "buenos dias"

  // shut down the system
  system.shutdown

```
Then, use `sbt run` to run the application. Here is the following output:
```
[info] Running Main
hello back at you
huh?
```
- `HelloActor`'s behavior is implemented by defining a `receive` method, which is implemented using a match expression.
- In `Main`, an `ActorSystem` is needed to get things started, so one is created.
- Actors can be created at the `ActorSystem` level, or inside other actors. At the `ActorSystem` level, actor instances are created with the `system.actorOf` method. The `helloActor` line shows the syntax to create an `Actor` with a constructor that takes no arguments.
- Actors are automatically started asynchronously when they are created. There is no need to call any "start" or "run" method.
- Messages are sent to actors with the `!` method.
- Your method should handle **ALL** potential messages that can be sent to the actor. Otherwise, an `UnhandledMessage` will be published to the `ActorSystem`'s `EventStream`.

### ActorSystem
An `ActorSystem` is the structure that allocates one or more threads for your application, so you typically create one `ActorSystem` per logical application.

### ActorRef
When you call the `actorOf` method on an `ActorSystem`, it starts the actor asynchronously and returns an instance of an `ActorRef`. This reference is a handle to the actor, to prevent you from breaking the Actor model.

An `ActorRef`:
- is immutable
- has a one-to-one relationship with the `Actor` it represents
- is serializable and network-aware, which lets you pass the `ActorRef` around the network.

## 13.2 Creating an Actor Whose Class Constructor Requires Arguments
Let's say you want to create an Akka actor, and you want your actor's constructor to have one or more arguments.

You can create the actor as such, where `HelloActor` takes one constructor parameter:
```
val helloActor = system.actorOf(Props(new HelloActor("Fred"))), name = "helloactor"
```

When creating an actor whose constructor takes one or more arguments, you still use the `Props` class, but with different syntax:

```
// an actor with a no-args constructor
val helloActor = system.actorOf(Props[HelloActor], name = "helloactor")

// an actor whose constructor takes one argument
val helloActor = system.actorOf(Props(new HelloActor("Fred")), ¾
name = "helloactor")
```

The code below shows a modified version of the example in Recipe 13.1:
```
import akka.actor._

// (1) constructor changed to take a parameter
class HelloActor(myName: String) extends Actor {
  def receive = {
    // (2) println statements changed to show the name
    case "hello" => println(s"hello from $myName")
    case _ => println(s"'huh?', said $myName")
  }
}

object Main extends App {
  val system = ActorSystem("HelloSystem")

  // (3) use a different version of the Props constructor
  val helloActor = system.actorOf(
                    Props(new HelloActor("Fred")), name = "helloactor")

  helloActor ! "hello"
  helloActor ! "buenos dias"
  system.shutdown
}
```
An actor instance is instantiate and started when the `actorOf` method is called, so the only ways to set a property in an actor instance are:
- By sending the actor a message
- In the actor's constructor
- In the actor's `preStart` method (in Recipe 13.4)

## 13.3 How to communicate between Actors
Actors should be sent immutable messages with the `!` method.

When an actor receives a message from another actor, it also receives an implicit reference named `sender`, and it can use that reference to send a message back to the originating actor.

For example, if you have an actor instance named `car`, you can send it a `start` message like: `car ! "start"`

Then, a simple version of a `receive` method for `car` could be:
```
def receive = {
  case "start" =>
    val result = tryToStart()
    sender ! result
  case _ => // do nothing
}
```

## 13.4 Understanding the Methods in the Akka Actor Lifecycle
In addition to its constructor, an `Actor` has the following lifecycle methods:
- `receive`
- `preStart`
- `postStop`
- `preRestart`
- `postRestart`

| Method | Description |
| --- | --- |
| The actor’s constructor | An actor’s constructor is called just like any other Scala class constructor, when an instance of the class is first created. |
| preStart | Called right after the actor is started. During restarts it’s called by the default implementation of postRestart. |
| postStop | Called after an actor is stopped, it can be used to perform any needed cleanup work. According to the Akka documentation, this hook “is guaranteed to run after message queuing has been disabled for this actor.” |
| preRestart | According to the Akka documentation, when an actor is restarted, the old actor is informed of the process when preRestart is called with the exception that caused the restart, and the message that triggered the exception. The message may be None if the restart was not caused by processing a message. |
| postRestart | The postRestart method of the new actor is invoked with the exception that caused the restart. In the default implementation, the preStart method is called |

## 13.5 Starting an Actor
Akka actors are started asynchronously when they're passed into the `actorOf` method using a `Props`.

At the `ActorSystem` level of your application, you create actors by calling the `system.actorOf` method.

```
val system = ActorSystem("HelloSystem")

// the actor is created and started here
val helloActor = system.actorOf(Props[HelloActor], name = "helloactor")

helloActor ! "hello"
```

Within an actor, you create a child actor by calling the `context.actorOf` method.

```
class Parent extends Actor {
  val child = context.actorOf(Props[Child], name = "Child")
  // more code here ...
}
```

## 13.6 Stopping Actors
There are several ways to stop Akka actors:
- call the `system.stop(actorRef)` at the `ActorSystem` level
- call the `context.stop(actorRef)` inside the actor
- an actor can stop itself by calling `context.stop(self)`
- send the actor a `PoisonPill` message: `actor ! PoisonPill`
- program a `gracefulStop`

The differences of the methods:
- `stop` method: The actor **will continue** to process its current message (if any), but no additional messages will be processed
- `PoisonPill` message: A `PoisonPill` message will stop an actor when the `PoisonPill` message is processed. The `PoisonPill` message is queued just like an ordinary message and will be handled according to its queue order.
- `gracefulStop` method lets you attempt to terminate actors gracefully, waiting for them to timeout. This is a good way to terminate actors in a specific order.

Termination of an actor is performed asynchronously; the `stop` method  may return before the actor is actually stopped.

An actor terminates in 2 steps:
1. It suspends its mailbox and sends a `stop` message to all of its children.
2. Then it processes termination messages from its children until they're all gone, at which point it terminates itself.
3. If one of the actors doesn't respond (e.g. because it's blocking), the process has to wait for that actor and may get stuck.
4. When additional messages aren't processed, they're sent to the `deadLetters` actor of the `ActorSystem`. You can access these with the `deadLetters` method on an `ActorSystem`.
5. The `postStop` method is invoked when an actor is fully stopped, which lets you clean up resources if needed.

The `gracefulStop` method returns a `Future` that will be completed with success when existing messages of the target actor has been processed and the actor has been terminated.   
If the actor isn't terminated within the timeout, the `Future` results in an `ActorTimeoutException`.

### "Killing" an actor
There is a concept called "supervisor strategies". When you implement a supervisor strategy, you can send an actor a `Kill` message, which can be used to restart the actor.

Within the default supervisory strategy, the `Kill` message terminates the target actor. In general, this approach is used to kill an actor to allow its supervisor to restart it. If you want to stop an actor, use one of the other approaches described in this recipe.
