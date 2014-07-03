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
    NodeX { axis = 0 : u2, left : u15, right: u15, axis : f32 },
    NodeY { axis = 1 : u2, left : u15, right: u15, axis : f32 },
    NodeZ { axis = 2 : u2, left : u15, right: u15, axis : f32 },
	Leaf  { tag  = 3 : u2, _: u2, tri0 : u20, tri1 : u20, tri2 : u20 }
}
```
This defines a 64-bit value, where the first two bits indicate the type of node
(internal node divided in x-, y- and z-axis, or a leaf node to the triangle
vertex data).

# Detailed design

## Syntax

The syntax needs to be extended with bit-sized integer literals. These are written
as `4u7`, or `-1i4`. In addition, bit-sized types of the form `u15` and `i9`
needs to be added. If the compiler needs to treat them as normal values,
zero- or sign-extension must take place.

```ebnf
bitdata-def       ::= "bitdata" "{" ( <bit-con> ("," <bit-con>)* "}" ( ":" <type> )?

bit-con           ::= <con-ident> "{" ( <bit-field> ("," <bit-field>)* "}"

bit-field         ::= tag-bits | labeled-bit-field 

tag-bits          ::= <expr>      ;;; Sized integer literals or sized integer constants

labeled-bit-field ::= <var-ident> ( "=" <expr> )? ":" <type>
```

## Limitations

* Each constructor must have the exact same bit-size. 

## Bit-field access

Access through the `.` operator is unchecked. In other words, this is valid

```rust
fn f(node : KdNode) -> f32 {
   let axis = node.NodeX.axis;
   axis
}
```

If there is only one bit-constructor in the bit data, the constructor name may
be elided:
```rust
fn bus(pci : PCI) -> u8 { pci.bus }
```

## Matching

Matching is not nescessarily exhaustive, as there may be "junk" values. For
instance, 
```rust
bitdata T { S { 0u5 }, N { 0b11111u5 } }
```
Here `T` is 5-bits, but if the value is anything else than 0 or 31, it is
considered "junk":
```rust
match t { S => "Zero", N => "Non-zero", _ => "Junk" }
```

## Compared to `enum`

The `bitdata` type is similar to the existing `enum` type with the following
differences: 

* The discriminator is not added automatically. 
* All bit-data constructors must have the exact same bit-size.

# Drawbacks

# Unresolved questions

This RFC does not discuss endianess issues. It is assumed that the bit-fields
are defined in target endianess.
