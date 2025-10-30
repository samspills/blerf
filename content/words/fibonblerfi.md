+++
title = "Fibonblerfi"
author = ["Sam Pillsworth"]
date = 2025-10-30T17:53:00-04:00
tags = ["rust"]
draft = false
creator = "Emacs 29.2 (Org mode 9.7.34 + ox-hugo)"
+++

I'm learning Rust and as my first little thing I've written a small script that
takes an input N and generates the corresponding Fibonacci number. The input is
limited to a `u8` that is 100 or less, and the output type is `u128` because
F<sub>100</sub> is a really big number. I would like to follow up with a version where I
implement my own input type with my own `parse` implementation that does the
bounds check.

The actual Fibonacci implementation feels a bit clunky, if only because I'm so
used to reaching for tail recursion and I don't think I have that in Rust (there
is maybe a crate, but a lot of what I was reading was a few years old and the
current state was unclear and I got overwhelmed).

If anyone is so moved: I am very much a beginner on my Rust journey, and any
tips towards idiomatic Rust would be welcome.

```rust
use std::io;

fn main() {
    println!("Enter fibonacci number to generate.");
    let n: u8 = loop {
        let mut string_n = String::new();
        io::stdin()
            .read_line(&mut string_n)
            .expect("Failed to read number!");

        match string_n.trim().parse() {
            Ok(num) => {
                if num > 100 {
                    println!("Please enter a positive integer less than 101.");
                    continue;
                } else {
                    break num;
                }
            }
            Err(_) => {
                println!("Please enter a positive integer.");
                continue;
            }
        };
    };

    let generated = match n {
        0 => 0,
        1 => 1,
        _ => fibonacci(n),
    };

    println!("The {n}th fibonacci number is {generated}!")
}

fn fibonacci(n: u8) -> u128 {
    let mut minus2: u128 = 0;
    let mut minus1: u128 = 1;
    let mut maybe_nth: u128 = 1;
    let mut counter = n;
    while counter > 1 {
        maybe_nth = minus2 + minus1;
        minus2 = minus1;
        minus1 = maybe_nth;
        counter -= 1;
        println!("debug counter: {counter}, maybe_nth: {maybe_nth}")
    }
    maybe_nth
}
```
