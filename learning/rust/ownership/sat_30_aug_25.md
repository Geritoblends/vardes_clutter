# Ownership

## Contents

- Ownership Moves
- `std::mem::take` and `std::mem::replace`
- Arc

### Ownership moves

#### Overview
To avoid unnecessary heap allocations Rust generally implicitly moves ownership. This allows us to cheaply move data across systems like the custom ones on top of Spine.

#### Example use case
```rust
let my_payload = ... // owned heap-allocated data

let msg = Message::new(
    id: "Custom",
    payload: MessagePayload::Custom {
        type_id: "MySystem:MyAction",
        payload: my_payload // moved ownership here
    }
);

spine.publish(msg); // moved msg here
``` 

### `std::mem::{take, replace};`

#### Overview
Sometimes, allocating a new element on the heap is necessary when we want to move the ownership of an element but still keep it valid for use. This is when std::mem take and replace become useful:
```rust
struct MyStruct {
    my_string: String
}

let my_object = MyStruct {
    my_string: String::from("hello") // Heap-allocated string (of course)
};

let take_my_string: String = std::mem::take(&mut my_object.my_string);

println!("{}", take_my_string); // Prints "hello"
println!("{}", my_object.my_string); // Prints "", my_object still works.
```

my_object.my_string is now an empty string because std::mem::take heap-allocated a new empty String, and moved the previous one to take_my_string. Heap-allocation of an empty element is often cheaper than copying the previous value.

Replace syntax is similar, except you have to provide the new value that will be exchanged.

```rust
let take_my_string: String = std::mem::replace(&mut my_object.my_string, String::from("goodbye"));
```


### Arc

Arc (Atomic Reference Counter) is a wrapper that lets concurrent threads access the same immutable data by extending the lifetime up to until the last thread drops the counter.
Internally, for every block/thread that owns the element, it adds +1 to the counter, and when that block or thread finishes, the counter is decreased by 1. When the counter reaches 0, the memory is released.

#### Use case

In concurrent Rust, moving or borrowing an object like a Message is often impossible if we want to send it to multiple receivers. In Spine, one message is often mapped to multiple handlers, which means that we can't simply move ownership to one handler because the other ones wouldn't be able to obtain that message, and tokio::spawn can't have borrowing either (you can't borrow to multiple threads at once naively). Simply wrapping the message in Arc lets you tie the lifetime to multiple "ownerships" and memory will be freed once all the threads' lifetimes finish.

```rust
pub fn publish(&self, msg: Message) {
    let wrapped_msg = Arc::new(msg);
    let router = self.message_router.load();

    if let Some(subscribers) = router.get(wrapped_msg.id) {
        for system_id in subscribers {
            if let Some(transmitter) = channels.get(system_id) {
                transmitter.send(Arc::clone(&wrapped_msg));
            }
        }
    }
}
