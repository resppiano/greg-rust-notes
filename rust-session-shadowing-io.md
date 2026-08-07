# Rust Learning Session Recap

**Topics:** Shadowing vs. `mut`, `println!` mechanics, `String::new()` vs. literals, reading user input, `std` and modules

---

## 1. Shadowing vs. `mut`

- **Shadowing** (`let x = x + 2;`) creates a *new* variable that hides the old one — same name, different memory slot.
- **`mut`** reuses the *same* variable, letting you reassign its value in place.
- Key distinction: shadowing can change a variable's **type**; `mut` cannot.

```rust
fn main() {
    let x = 5;
    let x = x + 2; // shadowing — new variable, still named x
    println!("x = {}", x); // 7
}
```

Working example shadowing a `&str` into a `usize` via `.len()`:

```rust
fn main() {
    let spaces = "     ";
    let spaces = spaces.len(); // type changes: &str -> usize
    println!("There are {} spaces", spaces);
}
```

---

## 2. `println!` mechanics

- `{}` placeholders go in the string; the values that fill them go *after*, outside the quotes, separated by commas.
- Placeholder count has to match argument count.

```rust
println!("Hello {} {}", first_name, last_name); // two {} , two args
```

---

## 3. `String::new()` vs. string literals

- `"     "` is a `&str` — baked into the binary at compile time, already has content.
- `String::new()` starts **empty** and heap-allocated — for values you'll build/mutate at runtime.

```rust
let spaces = "     ";        // &str, already has 5 spaces
let mut spaces = String::new(); // String, starts empty — needs .push_str() etc.
```

---

## 4. Reading user input

- `read_line` writes into a mutable `String` reference (`&mut name`) rather than returning one.
- It returns a `Result`, so `.expect("message")` handles the failure case.
- Reading two lines of input means **two complete, separate** `io::stdin()...expect(...)` chains — not one chain trying to do both.

Final working version from this session:

```rust
use std::io;

fn main() {
    let mut first_name = String::new();
    let mut last_name = String::new();
    println!("please enter your first name");

    io::stdin()
        .read_line(&mut first_name)
        .expect("That's not a real name");

    println!("Now enter your last name");
    io::stdin()
        .read_line(&mut last_name)
        .expect("That's not a real last name");

    println!("Hello {} {}", first_name, last_name);
}
```

**Bugs debugged along the way:**
- Missing `mut` on a variable that needed to be written into by `read_line`.
- Broken method chain — a `.read_line()` left dangling after a semicolon had already ended the previous statement.
- Mismatched `println!` placeholders vs. arguments.
- Printing a variable before it had been read into (order-of-operations logic bug).

**Note for later:** `read_line` captures the newline (`\n`) from pressing Enter as part of the string. Cleaned up later with `.trim()`.

---

## 5. `std`, modules, and `use`

- `std` is Rust's standard library — always available, no extra setup needed.
- `std::io` is one module inside it (input/output tools).
- `use std::io;` creates a shorthand — it runs **once**, at the top of the file, not once per call — so `io::stdin()` works anywhere below it without repeating the full path `std::io::stdin()`.
- Some things (`String`, `println!`, basic types) are part of the **prelude** and are automatically in scope without any `use` statement. `io` is not part of that automatic set.

```rust
use std::io; // shorthand set up once

fn main() {
    io::stdin(); // used here...
    // ...
    io::stdin(); // ...and again, no second `use` needed
}
```

---

## Summary

This session covered four real, self-debugged compiler-style errors (missing `mut`, broken method chains, placeholder mismatches, and read-before-write ordering) using shadowing, `mut`, string types, and stdin input — all built up through guided questions rather than handed-over code.
