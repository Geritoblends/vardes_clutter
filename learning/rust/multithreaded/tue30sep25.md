# Async types and what they do

#### Arc
Atomic reference counter: it's a wrapper type that lets us have multiple immutable references at the same time across threads and cpu cores. Internally, it holds an "atomic counter" that increments or decrements the number of references at the moment. When it reaches 0 the value's memory is freed.

#### Mutex
Mutual Exclusion: Ensures there can't be two owners at the same time; its a pessimistic lock that blocks the lock even if the operation is immutable (readonly). It can be used in multithreaded, and tokio::sync::Mutex can be used in async.

#### RwLock
Read-Write Lock: It's a type wrapper lock similar to Mutex, except that the lock only happens when theres a mutable owner: there can be many concurrent readers, but always only one writer.

## non-std or custom

#### DashMap
It's a sharded hashmap, kinda like
```rust
struct DashMap {
    shards: Vec<RwLock<HashMap<K, V>>>
}
```
It lets us have a mutable map with smaller locks compared to something like `RwLock<HashMap<K, V>>`; the KV pairs are distributed in a `let shard_id = hash(key) % shards_amount` manner.

#### DashMap<K, Arc<Mutex<V>>>
Allows us to have mutable V across threads with fast access. Perfect for something like a chunk map (`V: [Block; 1024]`), where there's regular worldgen DashMap insertions (cpu-bound, which needs parallelism) and a lot of I/O/network calls



