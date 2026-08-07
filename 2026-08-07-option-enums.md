# Day 2 — `Option` and Enums

*Session date: August 7, 2026*
*Topic: matching on `Option`, and the enum foundation underneath it*

---

## 1. The problem `Option` solves

Sometimes a value might not exist (first element of an empty list, a search that finds nothing). Most languages answer with null — and crash later when someone forgets to check. Rust makes "might not exist" explicit in the type system.

## 2. Enums — the foundation

An **enum** is a type defined by listing every value it can be:

```rust
enum Direction {
    North,
    South,
    East,
    West,
}

let heading = Direction::North;

match heading {
    Direction::North => println!("heading up"),
    Direction::South => println!("heading down"),
    Direction::East => println!("heading right"),
    Direction::West => println!("heading left"),
}
```

Enums and `match` are made for each other: the enum declares the complete list of possibilities, and the compiler checks the match against that list. Add a variant later and every match on that type becomes a compile error until it's handled — the compiler finds every spot the change affects.

## 3. `Option` is just an enum that carries data

Variants can carry data. `Option` comes from the standard library, defined roughly as:

```rust
enum Option<T> {
    Some(T),   // a value exists, and here it is
    None,      // no value
}
```

Read `Option<i32>` as **"a box that either contains an i32 or is empty."** Key things that confused me at first, now settled:

- `None` is **not a string** — no quotes. It's a variant, like tails on a coin.
- `Option<i32>` doesn't mean "a number" — the annotation describes the box, not its contents.
- `T` is a placeholder for whatever type the Option holds.
- `Some` and `None` can be written bare (no `Option::` prefix) because Option is so common.

## 4. `Some` does two different jobs

```rust
let maybe_age: Option<i32> = Some(34);   // job 1: BUILDING a value (34 is just test data)

match maybe_age {
    Some(age) => println!("age is {age}"),   // job 2: PATTERN — any Some, bind contents to `age`
    None => println!("age unknown"),
}
```

`Some(age)` is not looking for 34 specifically — `age` is a plain name, and a name in a pattern binds *anything*. It matches `Some(34)`, `Some(7)`, `Some(-200)`. To match one specific value, put a literal inside the pattern instead:

```rust
Some(34) => println!("exactly thirty-four!"),
Some(age) => println!("some other age: {age}"),
None => println!("age unknown"),
```

Same left-side-of-`=>` rules as Day 1 — the patterns just go inside the `Some(...)` now.

## 5. Exhaustiveness makes the null check mandatory

Deleting the `None` arm:

```
error[E0004]: non-exhaustive patterns: `None` not covered
```

The compiler knows `Option` has exactly two shapes, sees only one handled, and names the missing one. The classic forgot-the-null-check bug is a **compile error** in Rust. The only way to get the value out of an `Option` is pattern matching — so the empty case can never be silently skipped.

## 6. Open homework

```rust
let position: Option<i32> = Some(12);
```

Write a match that prints `found at index ` plus the number for `Some`, and `word not found` for `None`.

## 7. Up next

Custom enums with `match`, matching enums that carry data, and `if let` — the shorthand for when only one pattern matters.
