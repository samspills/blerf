+++
title = "Fibonblerfi Part 2"
author = ["Sam Pillsworth"]
date = 2025-10-31T22:50:00-04:00
tags = ["rust"]
draft = false
creator = "Emacs 29.2 (Org mode 9.7.34 + ox-hugo)"
+++

This is a refactor of the previous code example.
First, I took [Ross Baker's kind advice from his reply on Mastodon](https://social.rossabaker.com/@ross/115466163662270864).
But then I went down quite a rabbit hole as I followed my "custom input type" plan.

To start I created the `FibonacciNumber` struct, and started implementing `FromStr`. I struggled to implement the `Err` for the `Result` type at first, because I was trying to combine an `IntErrorKind::PosOverflow` (to be returned for numbers greater than 100) and a `ParseIntError`. I couldn't figure out how to limit my error enum to **just** the `PosOverflow` variant, the compiler kept telling me to use `IntErrorKind`. Eventually I gave up and just created my own variants. But once I had the enum it needed to be a proper `std::error::Error` which required implementing `Debug` (easy, because I could derive it) and `Display` (more toil but an easy enough pattern to follow).

I got momentarily distracted by the power of `Display` and implemented one for `FibonacciNumber` that gave each number the proper ordinal indicator. This was very satisfying.

Eventually I remembered the goal was `FromStr` and finished implementing that, returning either a `Ok(FibonacciNumber)` or an `Err(FibonacciNumberError)` in each branch.

The only nuance about using the `FibonacciNumber` was that I had to update my `fibonacci` function to borrow the number instead of moving it. I'm still learning about ownership in Rust so I'm not going to comment further on that other than to say: the compiler told me exactly what to do and I listened to it!

As previously: constructive feedback is much appreciated!

```rust
use std::{fmt::Display, io, str::FromStr};

#[derive(Debug)]
enum FibonacciNumberError {
    TooBigError,
    ParseError,
}
impl Display for FibonacciNumberError {
    fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
        match self {
            FibonacciNumberError::TooBigError => {
                write!(f, "Number must be a positive integer less than 101.")
            }
            FibonacciNumberError::ParseError => write!(f, "Number must be a positive integer."),
        }
    }
}
impl std::error::Error for FibonacciNumberError {}

struct FibonacciNumber {
    n: u8,
}
impl Display for FibonacciNumber {
    fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
        match self.n % 10 {
            1 => write!(f, "{}st", self.n),
            2 => write!(f, "{}nd", self.n),
            3 => write!(f, "{}rd", self.n),
            _ => write!(f, "{}th", self.n),
        }
    }
}
impl FromStr for FibonacciNumber {
    type Err = FibonacciNumberError;
    fn from_str(s: &str) -> Result<Self, Self::Err> {
        match s.trim().parse() {
            Ok(num) => {
                if num > 100 {
                    Err(FibonacciNumberError::TooBigError)
                } else {
                    Ok(FibonacciNumber { n: num })
                }
            }
            Err(_) => Err(FibonacciNumberError::ParseError),
        }
    }
}

fn main() {
    println!("Enter fibonacci number to generate.");
    let n: FibonacciNumber = loop {
        let mut string_n = String::new();
        io::stdin()
            .read_line(&mut string_n)
            .expect("Failed to read number!");

        match string_n.parse() {
            Ok(num) => break num,
            Err(e) => {
                println!("{e}");
                continue;
            }
        }
    };

    let generated = fibonacci(&n);
    println!("The {n} fibonacci number is {generated}.")
}

fn fibonacci(fib_num: &FibonacciNumber) -> u128 {
    match fib_num {
        FibonacciNumber { n: 0 } => 0,
        _ => {
            let mut minus2: u128 = 0;
            let mut minus1: u128 = 1;
            let mut maybe_nth: u128 = 1;
            let mut counter = fib_num.n;
            while counter > 1 {
                maybe_nth = minus2 + minus1;
                minus2 = minus1;
                minus1 = maybe_nth;
                counter -= 1;
            }
            maybe_nth
        }
    }
}
```
