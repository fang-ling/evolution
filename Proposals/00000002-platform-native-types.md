# Add Platform-Native Types

* Proposal: [00000002](00000002-platform-native-types.md)
* Implementation: [fang-ling/c-kit#6](https://github.com/fang-ling/c-kit/pull/6), [fang-ling/core-foundation-kit#5](https://github.com/fang-ling/core-foundation-kit/pull/5), [fang-ling/foundation-kit#12](https://github.com/fang-ling/foundation-kit/pull/12)

## Summary of changes

Introduces platform-native numeric type aliases (`CInteger`, `CUnsignedInteger`,
and `CFloatingPoint`) that adapt to the target architecture. The proposal also
completes the numeric type family with 16-bit integer aliases and extends
CoreFoundationKit and FoundationKit to support these types.

## Motivation

CKit currently provides fixed-width numeric types such as `CInteger64`,
`CUnsignedInteger32`, and `CFloatingPoint64`. These types are appropriate when a
specific storage size is required, but they are less suitable when code should
naturally adapt to the architecture of the target platform.

Many platform APIs use architecture-dependent integer types (`long`,
`unsigned long`, `NSInteger`, and `NSUInteger`) whose size matches the platform
word size. When interacting with such APIs, developers frequently need explicit
casts between fixed-width and platform-native types. This adds verbosity and can
introduce portability issues when code is compiled for both 32-bit and 64-bit
environments.

Similarly, some APIs represent counts, indexes, and dimensions using
platform-sized integers rather than fixed-width integers. Providing dedicated
aliases for these concepts makes APIs more idiomatic and allows implementations
to follow established platform conventions.

This proposal introduces platform-native numeric aliases that complement the
existing fixed-width types. The existing types remain available when exact
storage requirements are important, while the new aliases provide a convenient
choice for architecture-dependent values.

## Proposed solution

This proposal introduces three platform-native numeric aliases:

| Type | Definition |
|------|------------|
| `CInteger` | `long` |
| `CUnsignedInteger` | `unsigned long` |
| `CFloatingPoint` | `float` on 32-bit targets, `double` on 64-bit targets |

The proposal also adds the previously missing fixed-width aliases:

- `CInteger16`
- `CUnsignedInteger16`

To ensure consistent usage, CoreFoundationKit and FoundationKit are extended:

- `CoreFoundationNumberType` gains entries for the new types, missing 16-bit
  integer variants, and 32-bit floating-point variants.
- `FoundationNumber` gains convenience APIs to create and retrieve values using
  the new aliases.
- Existing functions and methods that previously used `CUnsignedInteger64` or
  `CFloatingPoint64` are updated to use `CInteger`, `CUnsignedInteger`, or
  `CFloatingPoint` as appropriate.

Adoption of these aliases across the ecosystem ensures code is consistent,
idiomatic, and aligned with platform-native sizes, while retaining existing
fixed-width types for cases that require them.

## Detailed design

### CKit Type Definitions

All new type aliases are defined in `CNumber.h`:

```c
/**
 * A signed integer value type.
 *
 * When building 32-bit applications, `CInteger` is a 32-bit integer. A 64-bit
 * application treats `CInteger` as a 64-bit integer.
 */
typedef long CInteger;

/**
 * An unsigned integer value type.
 *
 * When building 32-bit applications, `CUnsignedInteger` is a 32-bit unsigned
 * integer. A 64-bit application treats `CUnsignedInteger` as a 64-bit unsigned
 * integer.
 */
typedef unsigned long CUnsignedInteger;

#if C_TARGET_ARCHITECTURE_WASM32
#  define _C_FLOATING_POINT_TYPE float
#else
#  define _C_FLOATING_POINT_TYPE double
#endif

/**
 * The basic type for floating-point scalar values.
 *
 * When building 32-bit applications, `CFloatingPoint` is a 32-bit, IEEE
 * single-precision floating point type. A 64-bit application treats
 * `CFloatingPoint` a 64-bit, IEEE double-precision floating point type.
 */
typedef _C_FLOATING_POINT_TYPE CFloatingPoint;

/**
 * A 16-bit signed integer value type.
 */
typedef int16_t CInteger16;

/**
 * A 16-bit unsigned integer value type.
 */
typedef uint16_t CUnsignedInteger16;

/**
 * A 32-bit floating point type.
 */
typedef float CFloatingPoint32;
```

- `CInteger` and `CUnsignedInteger` map to the platform-native signed and
  unsigned integer types.
- `CFloatingPoint` is `float` on 32-bit targets and `double` on 64-bit targets.
- `CInteger16`, `CUnsignedInteger16` and `CFloatingPoint32` complete the
  fixed-width numeric value family.

### CoreFoundation Number Types

`CoreFoundationNumberType` is extended to include the new types:

- Signed integer: `kCoreFoundationNumberTypeInteger8`,
  `kCoreFoundationNumberTypeInteger16`, `kCoreFoundationNumberTypeInteger32`,
  `kCoreFoundationNumberTypeInteger`
- Unsigned integer: `kCoreFoundationNumberTypeUnsignedInteger8`,
  `kCoreFoundationNumberTypeUnsignedInteger16`,
  `kCoreFoundationNumberTypeUnsignedInteger32`,
  `kCoreFoundationNumberTypeUnsignedInteger`
- Floating-point: `kCoreFoundationNumberTypeFloatingPoint32`,
  `kCoreFoundationNumberTypeFloatingPoint`

### Foundation Number API

`FoundationNumber` gains new methods and properties to support new types, for
example:

```objective-c
/**
 * The number object's value expressed as a ``CInteger``, converted as
 * necessary.
 */
@property (nonatomic, readonly) CInteger integerValue;

/**
 * Creates and returns an ``FoundationNumber`` object containing a given value,
 * treating it as a ``CInteger``.
 *
 * - Parameter value: The value for the new number.
 *
 * - Returns: An ``FoundationNumber`` object containing value, treating it as a
 *   ``CInteger``.
 */
+ (instancetype)makeNumberWithInteger:(CInteger)value;
```

### Adoption in existing APIs

- Functions previously using `CUnsignedInteger64` now adopt `CInteger` or
  `CUnsignedInteger`.
- Functions previously using `CFloatingPoint64` now adopt `CFloatingPoint`.
- Example changes:

  ```diff
  -CUnsignedInteger64
  +CInteger
   CStringConvertUTF8CharactersToUTF32Characters(
     CInteger32* nillable destination,
     CString source,
  -  CUnsignedInteger64 maximumAllowedSize,
  -  CUnsignedInteger64 destinationSize
  +  CInteger maximumAllowedSize,
  +  CInteger destinationSize
   );
  ```
All similar functions in `Array`, `String`, `Geometry`, and other
CoreFoundationKit or FoundationKit modules are updated consistently.

## Source compatibility

Adoption of the new aliases in existing APIs is source-breaking:

  - Code that previously relied on `CUnsignedInteger64` or `CFloatingPoint64`
    may require changes to adopt `CInteger`, `CUnsignedInteger`, or
    `CFloatingPoint`.
  - Existing fixed-width types remain available, so code depending on
    exact-width behavior is unaffected.

New APIs, enums, and type aliases are fully additive, allowing incremental
adoption.

## Implications on adoption

- Developers adopting the new aliases must update function calls and type
  declarations where old fixed-width types were replaced.
- Libraries depending on CoreFoundationKit and FoundationKit must adopt these
  aliases to maintain consistent type usage.

## Alternatives considered

### Use `intptr_t` for `CInteger`

Define `CInteger` as `intptr_t` instead of `long`.

`intptr_t` would guarantee pointer-sized integers but is semantically for
pointer values, not general-purpose integers, and is available in C99.

### Use `size_t` for unsigned integers

Define `CUnsignedInteger` as `size_t`.

`size_t` is intended for object sizes; using `unsigned long` maintains symmetry
with CInteger and avoids conflating concepts.

### Always use `double` for `CFloatingPoint`

Simpler, but misses platform-specific optimizations. Conditional definition
allows automatic adaptation.
