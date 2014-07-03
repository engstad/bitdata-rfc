- Start Date: 2014-07-03
- RFC PR #: (leave this empty)
- Rust Issue #: (leave this empty)

# Summary

This RFC proposes to add support for bit-data types.

# Motivation

Rust aims to be a systems level language, yet it does not support bit-level
manipulations to a satisfactory level. We support a macro `bitflags!` for
supporting individual bits, but there is no support for bit-ranges. For
instance, with this RFC accepted we can describe a PCI address:

```rust
bitdata PCI {
    PCI { bus : u8, dev : u5, fun : u3 }
}
```

This definition describes a 16-bit value whose most significant eight bits
identify a particular hardware bus.

Immediate values can be specified anywhere in the definition, which provides 
a way to discriminate values:

```rust
bitdata KdNode {
    NodeX { 0u2, left : u15, right: u15, axis : f32 },
    NodeY { 1u2, left : u15, right: u15, axis : f32 },
    NodeZ { 2u2, left : u15, right: u15, axis : f32 },
	Leaf  { 3u2, _ : u2, tri : [u20, ..3] }
}
```
This defines a 64-bit value, where the first two bits indicate the type of node
(internal node divided in x-, y- and z-axis, or a leaf node to the triangle
vertex data).

# Detailed design

The `bitdata` type is similar to the existing `enum` type with the following
differences: 

* The discriminator is not added automatically. 
* All bit-data constructors must have the exact same bit-size.

# Drawbacks

# Unresolved questions
