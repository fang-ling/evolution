# API Design Guidelines

Delivering a clear, consistent developer experience when writing code is largely
defined by the names and idioms that appear in APIs. These design guidelines
explain how to make sure that your code feels like a part of the ecosystem.

## Fundamentals

- **Clarity at the point of use** is your most important goal. Entities such as
  methods and properties are declared only once but used repeatedly. Design APIs
  to make those uses clear and concise. When evaluating a design, reading a
  declaration is seldom sufficient; always examine a use case to make sure it
  looks clear in context.
- **Clarity is more important than brevity.** Although code can be compact, it
  is a _non-goal_ to enable the smallest possible code with the fewest
  characters. Brevity in code, where it occurs, is a side-effect of the type
  system and features that naturally reduce boilerplate.
- **Write a documentation comment** for every declaration. Insights gained by
  writing documentation can have a profound impact on your design, so don't put
  it off.

## Naming

### Promote Clear Usage

- **Include all the words needed to avoid ambiguity** for a person reading code
  where the name is used.

  For example, consider a method that removes the element at a given position
  within a collection.

  ```objective-c
  /* ✅ */
  @implementation FoundationMutableArray

  - (void)removeObjectAtIndex:(CInteger)index;

  @end

  [employees removeObjectAtIndex:x];
  ```

  If we were to omit the phrase `AtIndex` from the method signature, it could
  imply to the reader that the method searches for and removes an element equal to
  `x`, rather than using `x` to indicate the position of the element to remove.

  ```objective-c
  /* ⛔️ */
  [employees remove:x]; /* unclear: are we removing x? */
  ```

### Strive for Fluent Usage

- **Prefer method and function names that make use sites form grammatical
  English phrases.**

  ```objective-c
  /* ✅ */
  [x insertObject:y atIndex:z]; /* "x, insert y at z" */
  ```

  ```objective-c
  /* ⛔️ */
  [x insert:y at:z];
  ```

- **Begin names of factory methods with "**make**",** e.g. `[x makeIterator];`.

- **Methods that describe the state** of their receiver should be named as
  predicates beginning with "is", "has", "can", or similar auxiliaries, e.g.
  `isEnabled`, `hasChildren`, `canBecomeFirstResponder`.

- **Protocols that describe what something is should read as nouns** (e.g.
  `Collection`).

- **Protocols that describe a capability should be named using the suffixes**
  -able, -ible, or -ing (e.g. `Equatable`, `ProgressReporting`).

- **Protocols that describe a delegate relationship** should be named using the
  -Delegate suffix (e.g. `UIScrollViewDelegate`).

  - **Pass the delegating object as the first argument**. This allows a single
    delegate to service multiple instances of the same type without ambiguity.

- The names of other **types, properties, variables, and constants should read
  as nouns**.

### Use Terminology Well

- **Avoid obscure terms** if a more common word conveys meaning just as well.
  Don't say "epidermis" if "skin" will serve your purpose. Terms of art are an
  essential communication tool, but should only be used to capture crucial
  meaning that would otherwise be lost.

- **Stick to the established meaning** if you do use a term of art.

  The only reason to use a technical term rather than a more common word is that
  it _precisely_ expresses something that would otherwise be ambiguous or
  unclear. Therefore, an API should use the term strictly in accordance with its
  accepted meaning.

    - **Don't surprise an expert**: anyone already familiar with the term will
      be surprised and probably angered if we appear to have invented a new
      meaning for it.
    - **Don't confuse a beginner**: anyone trying to learn the term is likely to
      do a web search and find its traditional meaning.

- **Avoid abbreviations**. Abbreviations, especially non-standard ones, are
  effectively terms-of-art, because understanding depends on correctly
  translating them into their non-abbreviated forms.

  > The intended meaning for any abbreviation you use should be easily found by
  > a web search.

- **Embrace precedent**. Don't optimize terms for the total beginner at the
  expense of conformance to existing culture.

  It is better to name a contiguous data structure `Array` than to use a
  simplified term such as `List`, even though a beginner might grasp the
  meaning of `List` more easily. `Array`s are fundamental in modern computing,
  so every programmer knows—or will soon learn—what an array is. Use a term that
  most programmers are familiar with, and their web searches and questions will
  be rewarded.

## Conventions

### General Conventions

- **Document the complexity of any computed property that is not O(1)**. People
  often assume that property access involves no significant computation, because
  they have stored properties as a mental model. Be sure to alert them when that
  assumption may be violated.

- **Prefer methods and properties to free functions.**

- **Follow case conventions.**

  Acronyms and initialisms that commonly appear as all upper case in American
  English should be uniformly up- or down-cased according to case conventions:

  ```objective-c
  let utf8Bytes = "...";
  let isRepresentableAsASCII = no;
  let userSMTPServer = [[SecureSMTPServer alloc] init];
  ```

  Other acronyms should be treated as ordinary words:

  ```objective-c
  let radarDetector = [[RadarScanner alloc] init];
  let enjoysScubaDiving = yes;
  ```

- **Methods can share a base name** when they share the same basic meaning or
  when they operate in distinct domains.

## See Also

A lot of these comments and recommendations have been culled from other sources.

1. [Swift API Design Guidelines](https://www.swift.org/documentation/api-design-guidelines/)
