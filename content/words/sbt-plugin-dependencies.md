+++
title = "My plugin depends on WHAT?"
author = ["Sam Pillsworth"]
date = 2025-02-24T12:51:00-05:00
tags = ["scala", "sbt"]
draft = false
creator = "Emacs 29.2 (Org mode 9.7.11 + ox-hugo)"
+++

The Question: how do I know which sbt plugins are being brought in by this other sbt plugin in
my build?

This is something that comes up infrequently enough for me that I
always remember it's possible but infuriatingly can NEVER remember how to do it.
It's also really difficult to search for, so I spend a while feeling very frustrated
with sbt, the sbt plugin that has made me ask the question, and myself.

I'm writing it down here in the hopes that I will remember I wrote it down, and have just one place on the internet to search!

The Steps:

1.  In an sbt console, run `reload plugins` to get into the root project's `project/` build definition.
2.  Once in the project build, you can run a lot of your usual commands. For this particular problem, I run `dependencyBrowseTree` (from `sbt-dependency-tree` which I have defined as a global plugin) to open a searchable tree of all project dependencies.
3.  Once finished, `reload return` switches you back to the original build.

Pretty straightforward, just difficult to remember!
