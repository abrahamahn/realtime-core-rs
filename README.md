# realtime-core-rs

This repository contains only the `realtime-core` Rust crate and language-neutral JSON
conformance fixtures. It has no npm, Node.js, or TypeScript build dependency.

The crate provides dependency-free primitives for ordered delivery, subscription hubs, bounded
client recovery, reconnect state, heartbeat liveness, and application-defined realtime envelopes.

```rust
use std::collections::HashSet;
use realtime_core::{DeliveryCursor, DeliveryRecovery, SubscriptionHub};

let mut hub = SubscriptionHub::new("server-start-2026-08-26", 128)?;
hub.subscribe("document:42", "browser:1");
hub.plan_delivery("document:42", 3, "changed")?;
let cursor = DeliveryCursor::new("server-start-2026-08-26", 0)?;
let authorized = HashSet::from(["document:42"]);
match hub.recover_after(&cursor, &authorized) {
    DeliveryRecovery::Replay { entries, .. } => assert_eq!(entries.len(), 1),
    DeliveryRecovery::SnapshotRequired { .. } => unreachable!(),
}
# Ok::<(), realtime_core::RealtimeError>(())
```

Transport, authorization, persistence, clocks, epoch generation, serialization, and application
stream semantics are owned by the consuming server.

Delivery sequences, stream versions, capacities, and reconnect delays are capped at JavaScript's
exact-integer boundary so Rust-produced state remains lossless for TypeScript consumers. Reconnect
attempts use the same unsigned 32-bit range in both implementations.

```bash
cargo build --all-targets --locked
cargo check --all-targets --locked
cargo fmt --all -- --check
cargo clippy --all-targets --locked -- -D warnings
cargo test --all-targets --locked
```
