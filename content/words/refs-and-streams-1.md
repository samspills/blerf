+++
title = "Refs and Streams 1: Scheduling Updates"
author = ["Sam Pillsworth"]
tags = ["scala", "cats-effect", "fs2", "rough-thoughts"]
draft = true
creator = "Emacs 28.2 (Org mode 9.5 + ox-hugo)"
+++

The past month at `$work` I've been working on a fun little problem, and I decided to minimize
it down as a future example for myself. This is part one of two parts.


## The Problem {#the-problem}

We have some data, rows of recommendations. The data is stored on disk, and we want to write an API
that will return rows based on some query. The data, for now, is small enough that it can be read
into memory when we start the server. A different process will update the data on disk however, and
we need that update to propagate to the server without disrupting running requests. The update is
the piece I've been working on.


## The Code {#the-code}

I took this as an opportunity to build my understanding of [Cats Effect `Ref`](https://typelevel.org/cats-effect/docs/std/ref), since it "provides
safe concurrent access and modification of its content." I also wanted to use an
[fs2](https://fs2.io/#/) `Stream` to handle the scheduling task (mostly because I know the fs2
stream API reasonably well and I could see how to do it. I'm not sure if there is a better way to do
that piece.).

To start I defined a "simpler" version of my problem:

-   suppose I have a single `Ref`, in IO, with an integer value
-   I want to print the value every second
-   I want to add 1 to the value every 3 seconds

The implementation[^fn:1] for this reduced problem looks like so:

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

The `scheduledUpdate` and `printRefThing` methods both take in the ref as an argument. The
scheduling (printing every 1 second, or updating every 3 seconds) is handled by metering the
streams.

The entire experiment is orchestrated together in the `run` method, by creating `theRefThing`
**once**, and passing that same value to both methods. Those methods are run concurrently and
interrupted after 10 seconds.

I emphasize **once** because in the moment (and in the haze of the holidays[^fn:2]) I struggled here.
If I passed that initial `IO` around directly, then both streams would create a `Ref` each time
their respective programs ran. Both methods needs to be using the same `Ref` for any of this to
work.

The next step, which I'll write up separately, expands this reduced problem to include the API piece
(using [Http4s](https://http4s.org/)).

[^fn:1]: This code sample is also available as [a gist](https://gist.github.com/samspills/b1a3434e1bac21ac9c62004df2f25306). You can run it directly using `scala-cli`.
[^fn:2]: I didn't touch a computer for three whole weeks and it was glorious. I think my brain must have assumed I'd given up on tech and flushed my memory, because when I got back to work I could barely `println("Hello, world")`
