# Introducing Value Wrappers

* Proposal: [00000006](00000006-value-wrapper.md )
* Implementation: [fang-ling/core-foundation-kit#6](https://github.com/fang-ling/core-foundation-kit/pull/6), [fang-ling/foundation-kit#14](https://github.com/fang-ling/foundation-kit/pull/14)

## Summary of changes

Introduces `CoreFoundationValue` and `FoundationValue`, immutable wrapper types
that allow scalar values, pointers, and structures to be stored in
CoreFoundation and Foundation collections.

`FoundationNumber` becomes a specialized subclass of `FoundationValue`.

## Motivation

Foundation and CoreFoundation collections are designed to store object
references. While this model works well for most Objective-C and CoreFoundation
APIs, it becomes cumbersome when developers need to work with scalar values,
pointers, or structures.

We already recognizes numbers as first-class objects through
`CoreFoundationNumber`/`FoundationNumber`, but there is no corresponding
abstraction for arbitrary value types.

Providing a general-purpose value wrapper would establish a common
representation for arbitrary C and Objective-C values, allowing them to
participate naturally in Foundation and CoreFoundation collections while
improving API consistency and reducing the need for custom wrapper
implementations.

## Proposed solution

This proposal introduces two new types:

  - `CoreFoundationValue`
  - `FoundationValue`

These types act as immutable containers for arbitrary C and Objective-C values.

`FoundationValue` is implemented as a class cluster and bridges directly to
`CoreFoundationValue`, following the same design used by other
Foundation/CoreFoundation paired types.

The proposal also makes `FoundationNumber` a subclass of `FoundationValue`,
allowing numeric values to participate naturally in the value-wrapper hierarchy.

When a value object is created, the specified bytes are copied into internal
storage.

```objective-c
let point = (Point){ .x = 10, .y = 20 };

let value = [FoundationValue makeValueWithBytes:&point size:sizeof(Point)];
```

The stored value remains valid even after the original memory is modified or
released.

## Detailed design

### `FoundationValue`

`FoundationValue` is an immutable class cluster that stores a copy of an
arbitrary value.

```objective-c
/**
 * A simple container for a single C or Objective-C data item.
 *
 * A ``FoundationValue`` object can hold any of the scalar types such as
 * ``CInteger``, ``CUnsignedInteger``, and ``CFloatingPoint``, as well as
 * pointers, structures, and object `id` references. Use this class to work with
 * such data types in collections (such as ``FoundationArray`` and
 * ``FoundationDictionary``), and other APIs that require Objective-C objects.
 * ``FoundationValue`` objects are always immutable.
 *
 * ### Subclassing Notes
 *
 * The abstract ``FoundationValue`` class is the public interface of a class
 * cluster consisting mostly of private, concrete classes that create and return
 * a value object appropriate for a given situation. It is possible to subclass
 * ``FoundationValue``, but doing so requires providing storage facilities for
 * the value (which is not inherited by subclasses) and implementing two
 * primitive methods.
 *
 * #### Methods to Override
 *
 * Any subclass of ``FoundationValue`` must override the primitive instance
 * methods ``copyValue:`` and ``size``. These methods must operate on the
 * storage that you provide for the value.
 *
 * You might want to implement an initializer for your subclass that is suited
 * to the storage you provide. The ``FoundationValue`` class does not have a
 * designated initializer, so your initializer need only invoke the ``init``
 * method of `super`. The ``FoundationValue`` class adopts the
 * ``ObjectiveCCopyable`` protocol; if you want instances of your own custom
 * subclass created from copying, override the methods in this protocol.
 *
 * #### Alternatives to Subclassing
 *
 * If you need only to use ``FoundationValue`` objects for wrap a custom data
 * types or structures defined by your app, you need not create an
 * ``FoundationValue`` subclass.
 *
 * ## Topics
 *
 * ### Working with Raw Values
 *
 * - ``size``
 * - ``makeValueWithBytes:size:``
 * - ``copyValue``
 */
@interface FoundationValue: ObjectiveCObject

/**
 * The size of the data contained in the value object.
 *
 * This property provides the same value produced by the `sizeof` compiler
 * directive.
 */
@property (nonatomic, readonly) CInteger size;

/**
 * Creates a value object containing the specified value with the specified
 * size.
 *
 * - Parameters:
 *   - bytes: A pointer to data to be stored in the new value object.
 *   - size: The size in bytes of the data, as provided by the `sizeof` compiler
 *     directive. Do not hard-code this parameter as a `CInteger`.
 *
 * - Returns: A new value object that contains value.
 */
+ (instancetype)makeValueWithBytes:(const void*)bytes size:(CInteger)size;

/**
 * Copies the value into the specified buffer.
 *
 * - Parameter value: A buffer into which to copy the value. The buffer must be
 *   large enough to hold the value.
 */
- (void)copyValue:(void*)value;

@end
```

### `CoreFoundationValue`

`CoreFoundationValue` provides the CoreFoundation equivalent functionality.

```c
/* CoreFoundationValue.c */
struct CoreFoundationValue {
  CoreFoundationObject object;

  void* value;
  CInteger size;
};

/* CoreFoundationValue.h */
/**
 * A simple container for a single C data item.
 *
 * A ``CoreFoundationValue`` object can hold any of the scalar types such as
 * ``CInteger``, ``CUnsignedInteger``, and ``CFloatingPoint``, as well as
 * pointers, structures, and CoreFoundation object references. Use this class to
 * work with such data types in collections (such as ``CoreFoundationArray`` and
 * ``CoreFoundationDictionary``), and other APIs that require CoreFoundation
 * objects. ``CoreFoundationValue`` objects are always immutable.
 */
typedef struct CoreFoundationValue CoreFoundationValue;

/**
 * Creates a value object containing the specified value with the specified
 * size.
 *
 * - Parameters:
 *   - bytes: A pointer to data to be stored in the new value object.
 *   - size: The size in bytes of the data, as provided by the `sizeof` compiler
 *     directive. Do not hard-code this parameter as a `CInteger`.
 *
 * - Returns: A new value object that contains value.
 */
CoreFoundationValue* CoreFoundationValueInitializeWithBytesAndSize(
  const void* bytes,
  CInteger size
);

/**
 * Returns the size of the data contained in the value object.
 *
 * This property provides the same value produced by the `sizeof` compiler
 * directive.
 *
 * - Parameter value: The value object upon which to operate.
 *
 * - Returns: The size of the data contained in the value object.
 */
CInteger CoreFoundationValueGetSize(CoreFoundationValue* value);

/**
 * Copies the value into the specified buffer.
 *
 * - Parameters:
 *   - value: The value object upon which to operate.
 *   - buffer: A buffer into which to copy the value. The buffer must be
 *     large enough to hold the value.
 */
void CoreFoundationValueCopyValue(CoreFoundation* value, void* buffer);
```

### `FoundationCoreFoundationValue`

Like other CoreFoundation–Foundation paired types, `FoundationValue` is bridged
to a CoreFoundation counterpart, `CoreFoundationValue`. The concrete
implementation of `FoundationValue` is a private subclass,
`_FoundationCoreFoundationValue`, which shares the same memory layout as
`CoreFoundationValue`. This enables bridging between the two representations.

```objective-c
/* FoundationCoreFoundationValue.m */
@interface _FoundationCoreFoundationValue() {
  void* value;
  CInteger size;
}

@end
```

### `FoundationNumber`

`FoundationNumber` becomes a subclass of `FoundationValue`.

```diff
- @interface FoundationNumber: ObjectiveCObject
+ @interface FoundationNumber: FoundationValue
```

This change reflects the fact that a number is fundamentally a specialized value
wrapper.

`FoundationNumber` must implement the primitive requirements inherited from
`FoundationValue`.

## Implications on adoption

This proposal is largely additive. Changing the superclass of `FoundationNumber`
may affect code that relies on the exact class hierarchy. Such code is expected
to be uncommon.
