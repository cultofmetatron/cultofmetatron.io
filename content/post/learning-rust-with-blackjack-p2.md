---
Categories:
- Development
- GoLang
Description: ""
Tags:
- Development
- golang
date: 2017-03-24T11:34:11-04:00
menu: blog
title: learning rust with blackjack p2
---


I realize I've grown soft relying on garbage collection and rust is a harsh but fair taskmaster. 
Its been 4 days since I picked up rust and I'm finally starting to get the hang of the borrow checker.
Learning rust is a very different experience than learning c++.
A tight feedback loop is essential to the learning process.

As I continue to build this blackjack, I find that the compiler picks up issues that would lead to segfaults when I learned c.
One mechanism I've discovered is the `clonable` trait.
Lets say you have struct with a property set to another struct.
Rust can deep clone the entire object provided the struct and all subcomponents of the struct implement clonable.

I still don't quite understand the pragma yet but I've found it requires some annotations to the enums and structs.

```rust

#[derive(Copy)]
enum Suite {
    Clubs,
    Diamonds,
    Hearts,
    Spades,
}

#[derive(Copy)]
enum Rank {
    Num(u32),
    Ace,
    Jack,
    King,
    Queen,
}

#[derive(Copy)]
struct Card {
    suite: Suite,
    rank: Rank,
}

```

From there, I use a special case of the `impl` keyword

```rust


impl Clone for Suite {
    fn clone(&self) -> Suite {
        *self
    }
}


impl Clone for Rank {
    fn clone(&self) -> Rank {
        *self
    }
}

impl Clone for Card {
    fn clone(&self) -> Card { 
        *self
    }
}


```

Now Its trivial to clone a deck of cards

```rust
let deck: Vec<Card> = shuffle_deck(make_deck());
let newDeck: Vec<Card> = deck.clone();

```
Whats amazing is that the ownership and lifetime rules guarantee that the resources will be released when the current scope finishes.
