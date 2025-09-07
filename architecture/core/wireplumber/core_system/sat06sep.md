# CoreSystem Concept

#### General concept
A core system will be an arbitrary system crate that provides functionality naturally as in normal Rust. Most things will be handled through method calls, structs, and whatever. The difference between normal Rust is that a core system will have Events that will be passed as messages through a broadcast channel, where Side Effects will react to them however they want.

Examples of Core Systems
* Entity Position Core
* Netcode Core

Examples of NOT Core Systems
* Redis Dumb Abstraction
* MongoDB Wrapper 
