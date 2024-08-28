+++
title = "Batch 'em up, Move 'em on"
author = ["Sam Pillsworth"]
date = 2024-08-28T14:22:00-04:00
tags = ["scala", "fs2"]
draft = false
creator = "Emacs 29.2 (Org mode 9.8 + ox-hugo)"
+++

I really like working with fs2. I like saving little code examples for myself. I
want to use the blog more and get out of the habit of reaching for gists. Also I need somewhere to put my intrusive-thought song parodies[^fn:1].


## The Problem {#the-problem}

We have a stream of "things". We want to batch those things up, and do something
to each batch. We need to count which batch we're on. Oh and also, if the stream
of "things" is empty, we want to fallback to some other operation.


## The Code {#the-code}

```scala
import cats.effect.IO
import fs2.Stream

def getAndProcessThings(batchSize: Int): IO[Unit] = {
  val things: Stream[IO, Int] = Stream("a", "b", "c", "d", "e", "f", "g", "h", "i", "j").covary[IO]
  val fallback: Stream[IO, Unit] = Stream.eval(IO.println("falling back"))

  things
    .chunkN(batchSize) // split stream into chunks of batchSize
    .zipWithIndex // zip the chunks together with an index
    .evalMap { case (chunk, batchNumber) =>
      IO.println(s"Processing batch $batchNum: $chunk")
    }
    .ifEmpty(fallback)
    .compile
    .drain
}
```

If that function is run with a batch size of 3, we'll get the following output

```shell
processing batch number 0; Chunk(a, b, c)
processing batch number 1; Chunk(d, e, f)
processing batch number 2; Chunk(g, h, i)
processing batch number 3; Chunk(j)
```

If `things` was instead set to a value of `Stream.empty`, we'll instead see `falling back` printed.

[^fn:1]: Rawhidddeeeee
