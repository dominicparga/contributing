# Rust

The following conventions help keeping overview and the code clear.
Since there exists an [official Rust Style Guide][www_rust_style_guide], this file just adds or summarizes some useful info.
Keep the [guiding principles and rationale][www_rust_principles] in mind when coding rust.

1. [File style](#file-style)
1. [Some coding details](#coding-details)
    1. [Types](#types)
    1. [References](#refs)
    1. [Strings](#strings)
1. [Coding Conventions](#coding-conventions)
1. [Documentation](#doc)
1. [Howto setup a complex project](#complex-project)
    1. [`mod` vs `use`](#mod-vs-use)
    1. [Module and folder structure](#project-structure)

## File style <a name="file-style"></a>

* Maximum line width is `100`.
  Exceptions can be made for String-constants and similar.

  _This is a good trade off between `120` and `80`.
  Humans have trouble reading the code with increasing line width.
  In general, more than `80` is not recommended._

* Use `4 spaces` for indention (p.s.: [could help your salary][www_salary]!).

## Some coding details <a name="coding-details"></a>

### Types <a name="types"></a>

To get the type of a variable, consider "asking" the compiler by explicitly setting the type of a variable to `()`.

```rust
let my_number: () = 3.4;
// compiler:        ^^^ expected (), found floating-point variable
```

### References <a name="refs"></a>

Following code blocks from [rust-lang][www_rust_ref_keyword] show two identical lines.

`ref` on the left side of `=` is the same as `&` on the right side.

```rust
let ref x = 1;
let x = &1;
```

`&` on the left side of `=` is the same as `*` on the right side.

```rust
let &y = x;
let y = *x;
```

### Strings <a name="strings"></a>

Raw string literals allow writing escape characters without `\`.

[![Visualizing raw string literals][www_raw_strings_img]][www_raw_strings]

Example:

```rust
let toml = r#"
    [package]
    name = "osmgraphing"
    version = "0.1.0"
    authors = [
        "dominicparga <dominic.parga@gmail.com>",
        "jenasat <jena.s@outlook.de>"
    ]
    edition = "2018"

    [dependencies]
    quick-xml = "*"
"#;
```

## Coding Conventions <a name="coding-conventions"></a>

* Class names are written in `CamelCase`, functions, fields in `snake_case`.

* Use `constants` over `magic numbers`!

  _Even you as the author will not know the meaning of every number after several months.
  And if you know, you will probably forget the precision of your constant and the places, where you put them (-> bad for debugging)._

  ```rust
  // BAD

  let area = 3.1 * radius * radius;
  // (...)
  // somewhere else in the code
  let circum = 2 * 3.1415 * radius;
  // or
  let circum = 6.283 * radius;



  // GOOD

  // maybe inside a wrapper struct?
  let PI   = 3.1415;
  let PI_2 = 6.2832;

  let area = PI * radius * radius;
  // (...)
  // somewhere else in the code
  let circum = 2 * PI * radius;
  // or
  let circum = PI_2 * radius;
  ```

* Use __white spaces around binary operators__.
  Exceptions can be made for special cases to improve readability (see below).

  ```diff
  - let e = (- a) * b;
  + let e = (-a) * b;

  - let e = a*b;
  + let e = a * b;

  + let e = a * b + c * d;    // ok, but not recommended here
  + let e = a*b + c*d;        // improves readability
  ```

* In general, use `return` statements.
  Exception could be short functions to improve the overview.

  ```diff
  - fn blub() {
  -     3
  - }

  + fn blub() { 3 }

  + fn blub() {
  +     return 3;
  + }
  ```

## Documentation <a name="doc"></a>

* Separate module sections with `//---//` (whole line).
  Take the following code snippet for inspiration.

## Howto setup a complex project <a name="complex-project"></a>

### `mod` vs `use` <a name="mod-vs-use"></a>

While `mod` declares a module, `use` reduces verbose code by bringing namespaces into scope.
For more information, see [here][www_rust_mod_use_examples].

```rust
pub mod a {
    pub mod b {
        pub fn some_fn() {
            // ...
        }
    }
}

use a::b;

// bad style
// use a::b::somefn;

fn main() {
    b::some_fn();
    // instead of a::b::some_fn();
}
```

### Module and folder structure <a name="project-structure"></a>

Most of the following folder tree is from [Rust's Manifest Format doc][www_rust_project_overview].

```zsh
project_name
├── src/               # directory containing source files
│   ├── lib.rs         # the main entry point for libraries and packages
│   ├── main.rs        # the main entry point for packages producing executables
│   ├── bin/           # (optional) directory containing additional executables
│   │   └── *.rs
│   └── */             # (optional) directories containing multi-file executables
│       └── main.rs
├── examples/          # (optional) examples
│   ├── *.rs
│   └── */             # (optional) directories containing multi-file examples
│       └── main.rs
├── tests/             # (optional) integration tests
│   ├── *.rs
│   └── */             # (optional) directories containing multi-file tests
│       └── main.rs
└── benches/           # (optional) benchmarks
    ├── *.rs
    └── */             # (optional) directories containing multi-file benchmarks
        └── main.rs
```

Prefer package/folder/file management over class mangement if `meaningful`.  
__BUT:__ Think in an intuitive, handy and `deterministic`(!) way and don't take structuring and subfolding too far.

Always ask yourself:  
`How would most of the people search for this module/struct/file?`  
Someone without knowing your whole project structure should be able to find a file at the first try.  
`In every folder, there should be only one option to continue searching (-> determinism).`

Besides that, Rust provides other nice ways of managing source files.
One approach shows a submodule `module_a` as a file.

```zsh
project_name
├── src/
│   ├── module_a.rs
│   └── ...
└── ...
```

```rust
// project_name/src/module_a.rs
pub mod module_a {
    pub mod nice_content {
        // ...
    }
}
```

It is possible to split the module into multiple files easily as shown below.

```zsh
project_name
├── src/
│   ├── module_a
│   │   └── mod.rs
│   └── ...
└── ...
```

```rust
// project_name/src/module_a/mod.rs
pub mod nice_content {
    // ...
}
```

In both ways, main.rs can access the modules in the same way.

```rust
// project_name/src/main.rs
mod module_a;

fn main() {
    // access module_a::nice_content
}
```

[www_rust_style_guide]: https://github.com/rust-dev-tools/fmt-rfcs/blob/master/guide/guide.md
[www_rust_principles]: https://github.com/rust-dev-tools/fmt-rfcs/blob/master/guide/principles.md
[www_salary]: https://stackoverflow.blog/2017/06/15/developers-use-spaces-make-money-use-tabs

[www_rust_ref_keyword]: https://users.rust-lang.org/t/ref-keyword-versus/18818/4

[www_raw_strings]: https://rahul-thakoor.github.io/rust-raw-string-literals/
[www_raw_strings_img]: https://rahul-thakoor.github.io/img/rust_raw_string.png

[www_rust_mod_use_examples]: https://dev.to/hertz4/rust-module-essentials-12oi
[www_rust_project_overview]: https://doc.rust-lang.org/cargo/reference/manifest.html#configuring-a-target
