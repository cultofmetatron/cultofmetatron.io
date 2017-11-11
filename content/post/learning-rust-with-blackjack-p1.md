---
Categories:
- Development
- Rust
Description: ""
Tags:
- Development
- Rust
date: 2017-03-21T12:33:04-04:00
menu: blog
title: Learning Rust With Blackjack Part 1
---

I never got very far with c.
I was still a noob at programming and constant segaults with no feedback plagued my path to success.
I ultimately moved on to ruby and eventually javascript and elixir.
C is honest and close to the metal and systems program has been on my list of things to get back into.

My two primary languages these days are javscript and elixir.
This leaves my repertoire of skills with a hole in systems programming.
Go was the first langauge I looked at.
Its got a short learning curve and I'll also learn it since there are more jobs for it.
Rust however, appealed to me.

With the [Rust](https://www.rust-lang.org/en-US/) language, I found myself lured back into a world without garbage collection, as close to the metal as c++.
Projects like [RedoxOS](https://www.redox-os.org/) mean I can start looking into realtime embedded design.
Its not the same as C++. 
Rust is a very diffrent beast.
No more allocating and deallocating; rust brings higher order abstractions that I've gotten used to in functional langauges.
It can detect segfaults and memory leaks at runtime with error messages that actually make sense.


This weekend, I finally decided to take the plunge.

For a first project, I decided to go with blackjack.

Rust uses cargo as a built tool.
This gives us an easy way to start a project.

```rust
cargo new blackjack --bin
```

Now we have a basic project with a `Cargo.Toml` and a `src/main.rs` file.

Inside the `main.rs` file is the main function.

```

fn main() {
    println!("Hello, world!");
}

```

I remember a typical c main program has the main function return an int with the proper exit value being 0.
Rust abstracts away that detail.

With cargo, we can run the app with `cargo run`.

First thing I need is to successfully model the card deck.
Rust, borrowing heavily from functional roots has great support for Enum types.
As my first goal, I want to write a program that creates a shuffled deck of 52 cards.

As part of this, my goal is to support the following features today.

1. model one card with a constructor
2. create a function that creates a deck of card

A card has both a rank and a suite.
Rust has rich support for enumerated types which make this surprisingly easy.

```

enum Suite {
    Clubs,
    Diamonds,
    Hearts,
    Spades,
}

enum Rank {
    Num(i32),
    Ace,
    Jack,
    King,
    Queen,
}

struct Card {
    suite: Suite,
    rank: Rank,
}

```

I can now create a a 3 of hearts and an ace of spades!

```

let three_of_hearts: Card = Card {
  suite: Suite::Hearts,
  rank: Rank::Num(3),
};

//type declaration is optional
let ace_of_spades = Card {
  suite: Suite::Spades,
  ranks:: Rank::Ace,
};

```

The type declaration is optional since the compiler can infer it from the right hand side.
Rust includes for support for methods via the `impl` keyword which we can use to creat a  `new()` constructor.

```

impl Card {
    //this function takes a rank and suite and returns a card
    fn new(rank: Rank, suite: Suite) -> Card {
        Card {
            suite: suite,
            rank: rank,
        }
    }
}

```

This is equivalent to a class method or a static function in OO langauges.
With it, we now have an abbreviated way to create cards via our new instantiator.

```

let three_of_hearts = Card::new(Rank::Ace,     Suite::Spades);
let ace_of_spades   = Card::new(Rank::Num(3),  Suite::Hearts);

```

My next goal is a function `make_deck()` that creates a deck of cards.
Rust vectors look like a good choice for such a collection.

To create a basic Vector, we user `vec!`

```
// a vector 32 bit integers
let ls: Vector<i32> = vec![1, 2, 3];
let card_list: Vector<Card> = vec![
  Card::new(Rank::Ace,     Suite::Clubs),
  Card::new(Rank::Num(2),  Suite::Clubs),
  ...
];

```

Rust supports generics where the type is declared inside the brackets.
A function for creating a deck that returns a vector of cards would look like

```

//returns a type Vec<Card>
fn make_deck() -> Vec<Card> {
    vec![
      Card::new(Rank::Ace,     Suite::Clubs),
        Card::new(Rank::Num(2),  Suite::Clubs),
        Card::new(Rank::Num(3),  Suite::Clubs),
        Card::new(Rank::Num(4),  Suite::Clubs),
        Card::new(Rank::Num(5),  Suite::Clubs),
        Card::new(Rank::Num(6),  Suite::Clubs),
        Card::new(Rank::Num(7),  Suite::Clubs),
        Card::new(Rank::Num(8),  Suite::Clubs),
        Card::new(Rank::Num(9),  Suite::Clubs),
        Card::new(Rank::Num(10), Suite::Clubs),
        Card::new(Rank::King,    Suite::Clubs),
        Card::new(Rank::Jack,    Suite::Clubs),
        Card::new(Rank::Queen,   Suite::Clubs),

        Card::new(Rank::Ace,     Suite::Spades),
        Card::new(Rank::Num(2),  Suite::Spades),
        Card::new(Rank::Num(3),  Suite::Spades),
        Card::new(Rank::Num(4),  Suite::Spades),
        Card::new(Rank::Num(5),  Suite::Spades),
        Card::new(Rank::Num(6),  Suite::Spades),
        Card::new(Rank::Num(7),  Suite::Spades),
        Card::new(Rank::Num(8),  Suite::Spades),
        Card::new(Rank::Num(9),  Suite::Spades),
        Card::new(Rank::Num(10), Suite::Spades),
        Card::new(Rank::King,    Suite::Spades),
        Card::new(Rank::Jack,    Suite::Spades),
        Card::new(Rank::Queen,   Suite::Spades),

        Card::new(Rank::Ace,     Suite::Diamonds),
        Card::new(Rank::Num(2),  Suite::Diamonds),
        Card::new(Rank::Num(3),  Suite::Diamonds),
        Card::new(Rank::Num(4),  Suite::Diamonds),
        Card::new(Rank::Num(5),  Suite::Diamonds),
        Card::new(Rank::Num(6),  Suite::Diamonds),
        Card::new(Rank::Num(7),  Suite::Diamonds),
        Card::new(Rank::Num(8),  Suite::Diamonds),
        Card::new(Rank::Num(9),  Suite::Diamonds),
        Card::new(Rank::Num(10), Suite::Diamonds),
        Card::new(Rank::King,    Suite::Diamonds),
        Card::new(Rank::Jack,    Suite::Diamonds),
        Card::new(Rank::Queen,   Suite::Diamonds),

        Card::new(Rank::Ace,     Suite::Hearts),
        Card::new(Rank::Num(2),  Suite::Hearts),
        Card::new(Rank::Num(3),  Suite::Hearts),
        Card::new(Rank::Num(4),  Suite::Hearts),
        Card::new(Rank::Num(5),  Suite::Hearts),
        Card::new(Rank::Num(6),  Suite::Hearts),
        Card::new(Rank::Num(7),  Suite::Hearts),
        Card::new(Rank::Num(8),  Suite::Hearts),
        Card::new(Rank::Num(9),  Suite::Hearts),
        Card::new(Rank::Num(10), Suite::Hearts),
        Card::new(Rank::King,    Suite::Hearts),
        Card::new(Rank::Jack,    Suite::Hearts),
        Card::new(Rank::Queen,   Suite::Hearts),
    ]
}

```

You may have noticed that I left off the semicolon.
Rust is a little wierd with statements.
They don't return a value so we have to leave them off in order for the vector to be implicitly returned.

now we can create a deck of cards.

```

let dec: Vec<Card> = shuffle_deck(make_deck());

```

Lastly for today, I want a function that shuffles the deck. 

> by default, all data structures are immutable by default. The `mut` keyword
is used to make it explicitly mutable.

## crates

The [rand package](https://crates.io/crates/rand) provides the [shuffle()](https://doc.rust-lang.org/rand/rand/trait.Rng.html#method.shuffle) function.

To install it, we add the package to the Cargo.Toml.

```

[package]
name = "blackjack"
version = "0.1.0"
authors = ["Peter de Croos <cultofmetatron@aumlogic.com>"]

[dependencies]
rand = "0.3"

```

Next time we run `cargo build` or `cargo run`, it will download the dependencies.

Now we just add to the top of the main.rs.

```
extern crate rand;
use rand::Rng;

```

To create the shuffle function for our deck,
  we need to instantiate a local random number generator and pass into it a mutable version of our deck.

```
fn shuffle_deck(mut deck: Vec<Card>) -> Vec<Card> {
    let mut rng = rand::thread_rng();
    rng.shuffle(&mut deck);
    deck
}
```

To test the shuffle, I need some way to convert a card to a string.
For now I'll just focus on a function `stringify_rank(rank: &Rank)` that prints the rank.
This means we need to match each rank to an associated string.
Rust provides a `match` keyword thats very similar to elixir's case keyword.
The function needs to borrow a reference to the rank and return a reference to a string.

```

fn stringify_rank(rank: &Rank) -> String {
    let rank_str = match *rank {
        Rank::Num(val) => val.to_string(),
        Rank::Ace => "Ace".to_string(),
        Rank::Jack => "Jack".to_string(),
        Rank::King => "King".to_string(),
        Rank::Queen => "Queen".to_string(),
    };
    rank_str
}

```

Rust has both literal string and a referece string and you have to explicitly handle them diffrently.
I added the `.to_string()` to make sure they all return the same type in order to satisfy the return type specified in type declaration.

Now we can write a small main function that we can run over and over to verify that the deck is being shuffled.

```
fn main() {
    let dec: Vec<Card> = shuffle_deck(make_deck());
    let ref first = dec[0];
    let ref rank = first.rank;
    let str_y = stringify_rank(rank);
    println!("the first rank is: {}", str_y);
}

```

If we run this over and over with `cargo run`, you should get multiple values for the deck.

It was a bit of a culture shock and friction with the borrow checker.
That said, its amazing to me that this code will run in the same performance ballpark as c++ while guaranteeing no segfaults or memory leaks.


