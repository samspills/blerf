+++
title = "Refs and Streams 1: Scheduling Updates"
author = ["Sam Pillsworth"]
tags = ["scala", "cats-effect", "fs2"]
draft = true
creator = "Emacs 28.2 (Org mode 9.5 + ox-hugo)"
+++

The past few weeks at `$work` I've been working on a fun little problem. We have some data on disk,
and we have an API that returns rows from this dataset. This is a prototype/first version and the
data is reasonably small (for now) so we can hold it in memory. But a different process will update
the data, and we need to be able to grab those updates while our API is still serving requests.

I decided to take this as an opportunity to build up my understanding of cats-effect `Ref`. A `Ref`
"provides safe concurrent access and modification of its content,"[^fn:1] so it seemed a reasonable
opportunity to take advantage of it. I also wanted to use an fs2 Stream to handle the scheduling
task (mostly because I know the fs2 stream API reasonably well and I could see how to do it).

To start I minimized the problem down:

-   I have a `Ref`, in IO, with an integer value
-   I want to print the value every second
-   I want to add 1 to the value every 3 seconds

We can do this by defining two methods (one for printing and one for updating). Both methods take in
the `Ref` and return a metered stream that does it's task. The two streams are run concurrently (and
interrupted after 10 seconds).

The code[^fn:2] looks like so:

```scala
//> using lib "co.fs2::fs2-core:3.4.0"
//> using lib "org.typelevel::cats-effect:3.4.4"

import fs2.Stream
import cats.effect.IO
import cats.effect.IOApp
import scala.concurrent.duration._
import cats.effect.kernel.Ref

object Scheduled extends IOApp.Simple {
  val theRefThing: IO[Ref[IO, Int]] = Ref[IO].of(1)

  def scheduledUpdate(ref: Ref[IO, Int]): Stream[IO, Unit] = {
    val prog = ref
      .updateAndGet(int => int + 1)
      .flatMap(int => IO.println(s"updated the ref to $int"))
    Stream.repeatEval(prog).metered(3.seconds)
  }

  def printRefThing(ref: Ref[IO, Int]): Stream[IO, Unit] = {
    val prog = ref.get
      .flatMap(int => IO.println(s"the current ref value is ${int}"))
    Stream.repeatEval(prog).metered(1.second)
  }

  def run: IO[Unit] = theRefThing.flatMap(ref =>
    printRefThing(ref)
      .concurrently(scheduledUpdate(ref))
      .interruptAfter(10.seconds)
      .compile
      .drain
  )
}
```

Note that setting up the `Ref`, via `Ref[IO].of(1)`, returns an `IO[Ref[IO, Int]]`. In my head, I
describe this to myself as "a program that will produce a `Ref[IO, Int]`". In the moment (and in the
haze of the holidays[^fn:3]) I struggled here. If I passed that initial `IO` around directly, then
both streams would create a `Ref` each time their respective programs run. Instead what I had to do
was create the `Ref` first at the very beginning and then pass that created `Ref` around:
`theRefThing.flatMap( ref => ... use it here ... )`

[^fn:1]: <https://typelevel.org/cats-effect/docs/std/ref>
[^fn:2]: This code sample is also available as a gist: <https://gist.github.com/samspills/b1a3434e1bac21ac9c62004df2f25306>. You can run it directly using `scala-cli`.
[^fn:3]: I didn't touch a computer for three whole weeks and it was glorious. I think my brain must have assumed I'd given up on tech and flushed my memory, because when I got back to work I could barely `println("Hello, world")`
