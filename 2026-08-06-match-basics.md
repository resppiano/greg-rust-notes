# Day 1 — `match` Basics

*Session date: August 6, 2026*
*Topic: `match` from zero — arms, exhaustiveness, ranges, and bindings*

---

## 1. The core idea

`match` compares a value against a series of **arms** (pattern `=>` action), checked top to bottom. The first pattern that fits wins, its code runs, and the match is over.

```rust
match number {
    1 => println!("one"),
    2 => println!("two"),
    3 => println!("three"),
    _ => println!("something else"),
}
```

Key facts:

- The **left side** of `=>` is the pattern being tested. The right side is only the action — text printed there is never compared against anything.
- **Only one arm ever runs.** Once a pattern matches, the rest are skipped — no fall-through like other languages.
- Arms are separated by commas.

## 2. Exhaustiveness — Rust's superpower

Every possible value must be covered by some arm, or the code **will not compile**:

```
error[E0004]: non-exhaustive patterns: `i32::MIN..=-1_i32` and `101_i32..=i32::MAX` not covered
```

We proved it live: deleting the `_` arm from an integer match made the compiler list exactly which ranges were left uncovered. Forgetting a case is a compile error, not a runtime bug.

## 3. `match` is an expression

It produces a value you can assign. No semicolon after each arm's value, and every arm must produce the same type:

```rust
let word = match number {
    1 => "one",
    2 => "two",
    _ => "something else",
};
```

## 4. Pattern forms learned today

| Pattern | Meaning | Example |
|---|---|---|
| `5` | exact value | `5 => ...` |
| `1 \| 3 \| 5` | any of these ("or") | `1 \| 3 \| 5 => println!("odd digit")` |
| `0..=59` | inclusive range | `0..=59 => println!("fail")` |
| `_` | matches anything, binds **nothing** | `_ => println!("other")` |
| `name` | matches anything, **binds** the value | `other => println!("{other}")` |
| `n @ 1..=9` | tests the range **and** binds | `n @ 1..=9 => println!("single digit {n}")` |

## 5. Rules discovered the hard way

**Specific patterns first, catch-all last.** A binding matches *anything*, so any arm below it is dead code:

```
warning: unreachable pattern
    x => println!("got {x}"),
    - matches any value
    5 => println!("five!"),
    ^ no value can reach this
```

**`_` never binds.** `_ => println!("{other}")` fails with `cannot find value 'other' in this scope`. If the arm's body needs the value, the name must appear in the pattern: `other => ...` or `n @ range => ...`. A name pattern *always* binds; `_` *never* does.

**Don't bind what you already know.** `0 => println!("zero")` beats `n @ 0 => println!("zero {n}")` — if the pattern only matches one value, there's nothing to capture. `@` earns its keep on ranges.

## 6. Code I wrote

Score grader (ranges + wildcard):

```rust
match score {
    0..=59 => println!("fail"),
    60..=89 => println!("pass"),
    90..=100 => println!("excellent"),
    _ => println!("invalid score"),
}
```

Digit classifier (@ bindings + named catch-all), after two rounds of real debugging:

```rust
match digit {
    0 => println!("zero"),
    n @ 1..=9 => println!("single digit {n}"),
    n @ 10..=99 => println!("double digit {n}"),
    other => println!("big number {other}"),
}
```

Test run output: `zero 0` / `single digit 7` / `double digit 42` / `big number 500` / `big number -3` — note the negative number landing in the catch-all, an edge case worth remembering.

## 7. Style notes

- `println!` with no space before the `!` (compiles either way; this is the convention)
- Use `_` when you don't care what the value was; use a name when you do
