# memoize

A `#[memoize]` attribute for somewhat simple Rust functions. That's it.

Read the documentation (`cargo doc --open`) for the sparse details, or take a
look at the `examples/`, if you want to know more:

```rust
use memoize::memoize;
#[memoize]
fn hello(arg: String) -> bool {
     arg.len()%2 == 0
}

// `hello` is only called once here.
assert!(! hello("World".to_string()));
assert!(! hello("World".to_string()));
// Sometimes one might need the original function.
assert!(! memoized_original_hello("World".to_string()));
```

This is, aside from the `assert`s, expanded into:

```rust
// This is obviously further expanded before compiling.
lazy_static! {
  static ref MEMOIZED_MAPPING_HELLO : Mutex<HashMap<String, bool>>;
}

fn memoized_original_hello(arg: String) -> bool {
    arg.len()%2 == 0
}

fn hello(arg: String) -> bool {
    let mut hm = &mut MEMOIZED_MAPPING_HELLO.lock().unwrap();
    if let Some(r) = hm.get(&arg) {
        return r.clone();
    }
    let r = memoized_original_hello(arg.clone());
    hm.insert(arg, r.clone());
    r
}
```

Intentionally not yet on crates.rs.
