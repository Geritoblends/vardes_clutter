# Ergonomics for creating new Handlers

#### main.rs as the wireplumber

```rust
// Minimal MMO: just core systems
InventoryCore::init();

// Fancy MMO: core + lots of side effects  
InventoryCore::init()
    .with_side_effect(AchievementChecker)
    .with_side_effect(EconomyTracker)
    .with_side_effect(NotificationSender);
```
