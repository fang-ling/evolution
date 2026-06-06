# Add `ObjectiveCEquatable` to ObjectiveCKit

* Proposal: [00000005](00000005-equatable.md)
* Implementation: [fang-ling/c-kit#7](https://github.com/fang-ling/c-kit/pull/7), [fang-ling/objective-c-kit#8](https://github.com/fang-ling/objective-c-kit/pull/8), [fang-ling/foundation-kit#13](https://github.com/fang-ling/foundation-kit/pull/13)

## Summary of changes

Introduces the `ObjectiveCEquatable` protocol, allowing Objective-C types to be
compared for value equality in a canonical, consistent manner.

## Motivation

Currently, Objective-C types implement equality ad hoc via `isEqual:`. This
leads to inconsistent behavior across types and complicates generic programming.
By introducing a formal `ObjectiveCEquatable` protocol, types can explicitly
declare their support for value equality, enabling safer and clearer comparisons
and facilitating protocol-oriented designs similar to Swift.

## Proposed solution

We propose introducing the `ObjectiveCEquatable` protocol. Types that adopt this
protocol define canonical equality semantics by implementing `isEqual:`.

```objective-c
[@"cat" isEqual:@"another cat"];
/* Evaluates to `no`. */
```

## Detailed design

- `ObjectiveCEquatable` is a new protocol in ObjectiveCKit.
- All adopting types implement `isEqual:` to define equality.
- Common FoundationKit types will adopt `ObjectiveCEquatable`.
- Objective-C containers can implement recursive equality checking based on
  contained elements.

```objective-c
/**
 * A type that can be compared for value equality.
 *
 * Types that conform to the ``ObjectiveCEquatable`` protocol can be compared
 * for equality or inequality using the `isEqual:` method. Most basic types
 * conform to ``ObjectiveCEquatable``.
 */
@protocol ObjectiveCEquatable

/**
 * Returns a Boolean value that indicates whether the receiver and a given
 * object are equal.
 *
 * This method defines what it means for instances to be equal. For example, a
 * container object might define two containers as equal if their corresponding
 * objects all respond `yes` to an `isEqual:` request.
 *
 * - Parameter object: The object to be compared to the receiver. May be `nil`,
 *   in which case this method returns `no`.
 *
 * - Returns: `yes` if the receiver and the `object` are equal, otherwise `no`.
 */
- (CBoolean)isEqual:(nullable ObjectiveCAnyObject)object;

@end
```

## Source compatibility

This proposal is additive — existing code will not have to change due to this
API addition.

## Implications on adoption

Adopting this protocol may require updating custom types to implement `isEqual:`
consistently. Library authors can adopt `ObjectiveCEquatable` incrementally; no
changes are required for types that already implement -isEqual:. This feature
can be freely adopted or ignored without affecting existing source
compatibility.
