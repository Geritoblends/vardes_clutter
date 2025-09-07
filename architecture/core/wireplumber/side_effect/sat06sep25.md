# SideEffect Concept

#### General concept
A SideEffect will perform a "side effect" given a core system event. This allows extensibility while keeping core systems minimal.

**How it works**
It will react to messages from a core system. Those messages will be passed throught tokio broadcast channels (or whatever spmc implementation fits best).
To keep things fast, Messages passed must preferably implement the Copy trait.
