---
name: handling-xff-values
description: Use when constructing, inspecting, converting, or matching XffValue enums from the aequa (or athena/nabu) libraries in memory.
---

# Handling XffValues

## Overview
`XffValue` is the core dynamic data enum defined in the `aequa` library. It has an **enormous footprint** across the entire Pantheon ecosystem, serving as the default in-memory format for data interchange.

> [!IMPORTANT]
> **Stability Guarantee**: The public API of `XffValue` (its variants, `is_*`, `as_*`, and `into_*` methods) is **strictly immutable**. Breaking changes are prohibited to preserve logical stability across all Pantheon projects.

## Ergonomic Access Patterns
To avoid long file lookup loops:

### 1. Test Assertions
Use the boolean checkers (`is_string()`, `is_boolean()`, etc.) primarily for test assertions:
```rust
assert!(value.is_boolean());
assert!(value.is_object());
```

### 2. Extracting & Matching
Avoid chaining `is_*` check methods in code.
* **Single Variant Extraction**: Use `if let` with the `into_*` or `as_*` helpers:
  ```rust
  if let Some(obj) = value.into_object() {
      // use obj
  }
  ```
* **Multiple Variant Handling**: Use a standard `match` block with a catch-all:
  ```rust
  match value {
      XffValue::String(s) => { ... }
      XffValue::Number(n) => { ... }
      _ => { ... }
  }
  ```

### 3. Function Signatures & Boundaries
* **Type Signatures**: Define internal functions with concrete types (e.g. `Object`, `Array`, `Table`) instead of passing generic `XffValue`s.
* **Cast at Boundaries**: Cast down immediately at function start, and cast up right before return:
  ```rust
  fn process_data(value: XffValue) -> Result<XffValue, &'static str> {
      // Downcast immediately at the boundary
      let mut obj = value.into_object().ok_or("Expected object")?;
      
      // Populate concrete type
      obj.insert("status".to_string(), XffValue::from(true));
      
      // Upcast at return boundary
      Ok(XffValue::from(obj))
  }
  ```

## API Reference Quick-Lookup
* **Conversion Helpers (`into_...`)**: `into_string()`, `into_number()`, `into_array()`, `into_object()`, `into_ordered_object()`, `into_table()`, `into_uuid()`, `into_data()`, `into_boolean()`, `into_datetime()`, `into_duration()`, `into_graph()`, `into_hp_float()`.
* **Borrowing Helpers (`as_...` / `as_..._mut`)**: `as_string()`, `as_number()`, `as_array()`, `as_object()`, `as_table()`, `as_data()`, `as_uuid()`, `as_boolean()`, `as_string_mut()`, `as_number_mut()`, `as_array_mut()`, `as_object_mut()`, etc.
* **Macros**:
  * `xff!(val)`: Converts Rust primitives to `XffValue`. Example: `xff!(42)`.
  * `tvec_to_xff_value!(type; el1, el2)`: Direct typed vector construction.

## Common Mistakes
* Reading `aequa/src/value/mod.rs` in its entirety to check method names (use this skill instead!).
* Implementing nested `is_*` check blocks instead of `if let Some(x) = val.into_x()`.
