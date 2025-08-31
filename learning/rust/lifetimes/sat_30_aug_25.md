# Lifetimes

## Contents

- `'a` Usage


### `'a` Usage

#### Overview

Rust types (like structs and traits) can have defined lifetimes for its elements

#### Examples

A rust struct might have (mutable) references to other elements that explicitly only live during the period its 'a attribute lives:

```rust
struct MyStruct<'a> {
    my_attribute: &'a str
}
```

Similarly with traits:

```rust
trait MyTrait<T>
where
    T: Serialize + for<'de> Deserialize<'de> 
{
    fn foo(&self);
}
```

(tho I don't understand the exact use case of serde Deserialize)

#### Use case

You often end up wanting to return references out of a function in zero-copy code:

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if (x.len() < y.len()) { y } else { x }
}
```

The output is a reference to the original Strings, which is useful in a heap-allocated scenario like this.


