# Async single-threaded rust types

#### RefCell
Allows mutable references to be checked at runtime. Panics if the rules are broken.
For example, we might use tokio::main(flavor = "current_thread") on a single threaded WASM scenario and we still might take advantage of async rust for IO/network heavy workflows:
```rust
let mut x: i32 = 5;

tokio::task::spawn_local( async {
    x = 7; // -> error, can't move mutable values inside an async block
});
```
Solution:
```rust
let x = RefCell::new(5); // Notice how there's no mut

tokio::task::spawn_local( async {
    let mut x = x.borrow_mut(); // this works
    x = 7;
});
```

However, this might cause a panic in a scenario where a task is paused by the tokio async runtime task scheduler

```rust
let x = RefCell::new(5);

tokio::task::spawn_local( async {
    let mut x = x.borrow_mut();
    let val = get_val_from_file().await; // Task sleeps, scheduler starts the next task
    x = val;
});

tokio::task::spawn_local( async {
    let mut x = x.borrow_mut(); // Panic. Two mutable borrows at once
    x = 8;
});
```  
Band-aid solution
```rust
tokio::task::spawn_local( async {
    let val = get_val_from_file().await;
    let mut x = x.borrow_mut(); // Move the mutable borrow after any await call
    x = val;
});
```

#### Rc
Reference counter: similar to Arc, but with a plain integer: it has less overhead than atomic operations

