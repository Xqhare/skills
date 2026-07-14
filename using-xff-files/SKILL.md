---
name: using-xff-files
description: Use when deciding whether to store data in the XFF (Xqhare File Format) binary format, or when choosing between XFF and text-based formats like JSON, TOML, CSV, or custom configurations.
---

# Using XFF Files

## Overview
XFF (Xqhare File Format) is a custom binary format designed for long-term data preservation. 
* **Nabu** handles the **physical** (on-disk) representation (CRC-32 checksums, even-parity marker bytes).
* **Aequa** (via `XffValue`) handles the **logical** (in-memory) representation.

## When to Use

```dot
digraph xff_decision {
    "Need to save data?" [shape=doublecircle];
    "Is it application configuration?" [shape=diamond];
    "Use TOML / JSON\n(Human editable)" [shape=box];
    "Does it change extremely frequently?" [shape=diamond];
    "Use SQLite / custom append-only\n(Avoid serde overhead)" [shape=box];
    "Need bit-level durability and\ncorruption detection?" [shape=diamond];
    "Use XFF (nabu)\n(Version 4 default)" [shape=box];
    "Use standard format\n(e.g., CSV/JSON)" [shape=box];

    "Need to save data?" -> "Is it application configuration?";
    "Is it application configuration?" -> "Use TOML / JSON\n(Human editable)" [label="yes"];
    "Is it application configuration?" -> "Does it change extremely frequently?" [label="no"];
    "Does it change extremely frequently?" -> "Use SQLite / custom append-only\n(Avoid serde overhead)" [label="yes"];
    "Does it change extremely frequently?" -> "Need bit-level durability and\ncorruption detection?" [label="no"];
    "Need bit-level durability and\ncorruption detection?" -> "Use XFF (nabu)\n(Version 4 default)" [label="yes"];
    "Need bit-level durability and\ncorruption detection?" -> "Use standard format\n(e.g., CSV/JSON)" [label="no"];
}
```

### Correct Circumstances (Use XFF)
* **Long-Term Storage**: When data durability and bit-level integrity are paramount.
* **Binary Blobs**: If you need to store binary data, prefer using `nabu` to store it as a native `XffValue::Data` blob. **Never base64/binary-to-text encode binary data inside JSON, TOML, or CSV.**
* **Whole Unit Read/Write**: When the files are naturally read or written in their entirety as a single logical unit.

### Incorrect Circumstances (Avoid XFF)
* **Configuration Files**: **Never use XFF for application configuration.** Configurations must be easily inspected and edited by hand without specialized tooling. Use TOML or JSON instead.
* **High-Frequency Writes**: The CPU cost of recalculating CRC-32 and parity bytes makes it inefficient for highly volatile data (e.g. caches).

## Version Lifecycle & Upgrades
* **Auto-Upgrades**: Standard `read` auto-detects older versions (v0-3) and parses them. Standard `write` always writes the latest version (v4). Reading an old file and writing it back automatically upgrades the format.
* **Production Rule**: In production, always use `write()` to keep files upgraded. `write_legacy()` is reserved exclusively for testing.

## Code Reference
```rust
use nabu::serde::{read, write};
use nabu::XffValue;

// Writing (implicitly defaults to latest v4)
let val = XffValue::from("data");
write("data.xff", val).unwrap();

// Reading (auto-detects and upgrades v0-3 to v4 in-memory)
let parsed = read("data.xff").unwrap();
```

## Common Mistakes
* Proposing XFF for `.toml` configuration replacements.
* Using `write_legacy()` in production environments.
