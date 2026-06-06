# Coding Standards

This document describes coding standards that are used in the project. Although
no coding standards should be regarded as absolute requirements to be followed
in all instances, coding standards are particularly important for large-scale
code bases that follow a library-based design.

While this document may provide guidance for some mechanical formatting issues,
whitespace, or other “microscopic details”, these are not fixed standards.
Always follow the golden rule:

> If you are extending, enhancing, or bug fixing already implemented code, use
> the style that is already being used so that the source is uniform and easy to
> follow.

The ultimate goal of these guidelines is to increase the readability and
maintainability of our common source base.

## Mechanical Source Issues

### Source Code Formatting

#### Commenting

Comments are important for readability and maintainability. When writing
comments, write them as English prose, using proper capitalization, punctuation,
etc. Aim to describe what the code is trying to do and why, not how it does it
at a micro level. Here are a few important things to document:

##### File Headers

Every source file should have a header containing the file's basic information.
The standard header looks like this:

```c
/*
 *  [Filename]
 *  [package-name]
 *
 *  Created by [Your Name] on [yyyy]/[m]/[d].
 *
 *  Licensed under the Apache License, Version 2.0 (the "License");
 *  you may not use this file except in compliance with the License.
 *  You may obtain a copy of the License at
 *
 *    http://www.apache.org/licenses/LICENSE-2.0
 *
 *  Unless required by applicable law or agreed to in writing, software
 *  distributed under the License is distributed on an "AS IS" BASIS,
 *  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 *  See the License for the specific language governing permissions and
 *  limitations under the License.
 */
```

The first section of the header describes the basic information of the file,
making the file itself easier to identify and search for.

This is followed by a notice that defining the license under which the file is
released. This makes it perfectly clear under what terms the source code may be
distributed and modified.

The license text may vary depending on the license being used. For example,the
GNU Lesser Public License:

```c
/*
 *  <...>
 *
 *  This program is free software: you can redistribute it and/or modify it
 *  under the terms of the GNU Lesser General Public License version 3.0 only,
 *  as published by the Free Software Foundation.
 *
 *  This program is distributed in the hope that it will be useful, but WITHOUT
 *  ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or
 *  FITNESS FOR A PARTICULAR PURPOSE. See the GNU Lesser General Public License
 *  version 3.0 for more details.
 *
 *  You should have received a copy of the GNU Lesser General Public License
 *  version 3.0 along with this program. If not, see
 *  <https://www.gnu.org/licenses/>.
 */
```

Or a proprietary notice with no open-source license at all:

```c
/*
 *  <...>
 *
 *  Copyright (c) [yyyy]-[yyyy] [Your Name]. All rights reserved.
 *
 *  This software is the sole property of [Your Name]. Dissemination of this
 *  information or reproduction of this material is strictly forbidden
 *  unless prior written permission is obtained from the copyright holder.
 *
 *  This program is distributed in the hope that it will be useful, but WITHOUT
 *  ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or
 *  FITNESS FOR A PARTICULAR PURPOSE.
 */
```

Instead of C-style multi-line comments, `//` comments may also be used in C++ or
Swift source files.

##### Header Guard

The header file's guard should be the filename, using '\_' instead of extension
marker. For example, the header file `CString.h`'s guard is `CString_h`.

##### Class overviews

Classes are a fundamental part of an object-oriented design. As such, a class
definition should have a comment block that explains what the class is used for
and how it works. Every non-trivial class is expected to have a DocC comment
block.

##### Method information

Methods and global functions should also be documented. A quick note about what
it does and a description of the edge cases is all that is necessary here. The
reader should be able to understand how to use interfaces without reading the
code itself.

Good things to talk about here are what happens when something unexpected
happens, for instance, does the method return `null`?

#### Comment Formatting

In general, prefer C-style comments (`/* ... */` for normal comments,
`/** ... */` for DocC documentation comments) for C and Objective-C source
files, and prefer C++-style comments (`//` for normal comments, `///` for DocC
documentation comments) for C++ and Swift source files.

#### DocC Use in Documentation Comments

Include descriptive paragraphs for all public interfaces (public classes, member
and non-member functions). Avoid restating the information that can be inferred
from the API name or signature. The first sentence is used as a summary. Try to
use a single sentence, because DocC uses the first line of a documentation
comment as the summary. Put detailed discussion into separate paragraphs.

A minimal documentation comment:

```objective-c
/**
 * The receiver's superview, or `nil` if it has none.
 */
@property (nonatomic, readonly) UIView* superview;
```

Always document function parameters and return values, even when their behavior
appears obvious. Documentation should explicitly describe the intent, usage, and
behavior of APIs to improve clarity, consistency, and long-term maintainability.
Use descriptive function and argument names whenever possible, but do not rely
on naming alone as a substitute for documentation comments.

When you need to provide additional content for a symbol, add one or more
paragraphs directly below a symbol's summary to create a Discussion section. The
content you include depends on the type of symbol you're documenting:

- For a property, explain how it affects the behavior of its parent. Describe
  typical usage and any permitted or default values.
- For a method, describe its usage patterns and any side effects or additional
  behaviors. Highlight whether the method executes asynchronously or performs
  any expensive operations.
- For an enumeration case or constant, concisely describe what it represents.

For methods that take parameters, document those parameters directly below the
summary, or the Discussion section if you prefer. Describe each parameter in
isolation. Discuss its purpose and, where necessary, the range of acceptable
values.

DocC supports two approaches for documenting the parameters of a method. You can
either use a parameters "section" or one or more parameter "fields". Both use
Markdown's list syntax.

A Parameters "section" uses a single top-level unordered list item; starting
with either a hyphen (-), asterisk (\*), or plus sign (+), followed by a space,
the plural `Parameters` keyword (case insensitive), and a colon (:). Individual
parameters use nested list items; starting with two spaces of indentation,
either a hyphen (-), asterisk (\*), or plus sign (+), one space, the parameter
name, a colon, and the formatted documentation for that parameter.

Parameter "fields" use individual top-level unordered list items for each
parameter; starting with either a hyphen (-), asterisk (\*), or plus sign (+),
followed by a space, the singular `Parameter` keyword (case insensitive), one
space, the parameter name, a colon, and the formatted documentation for that
parameter.

Some parameters can benefit from more than one paragraph of documentation. For
example:

- Additional documentation for a boolean parameter can describe the effects of
  passing either a `yes`/`true` or `no`/`false` value if its not already clear
  from the parameter's name.
- Additional documentation for an enumeration parameter can describe the effects
  of passing each case if its not already clear from combination of the
  parameter's name and the case's name.
- Additional documentation for an closure parameter can describe the inputs and
  of that closure that closure if its not already clear from the parameter's
  name.

For methods that return a value, include a `Returns` section in your
documentation comment to describe the returned value. If the return value is
optional, provide information about when the method returns nil.

A `Returns` section contains a single top-level unordered list item; starting
with either a hyphen (-), asterisk (\*), or plus sign (+), followed by a space,
the `Returns` keyword (case insensitive), a colon, and the formatted
documentation that describe the returned value.

A documentation comment that includes each of the previously mentioned sections
provides much more information to developers than a single-line source comment,
as the following example shows:

```objective-c
/**
 * Returns a string created by using a given format string as a template into
 * which the remaining argument values are substituted.
 *
 * Pass a comma-separated list of trailing variadic arguments to substitute into
 * format.
 *
 * - Parameter format: A format string. This value must not be `nil`.
 *
 * - Returns A string created by using format as a template into which the
 *   remaining argument values are substituted without any localization.
 */
+ (instancetype)makeStringWithFormat:(FoundationString*)format, ...;
```

Don't duplicate the documentation comment in the header file and in the
implementation file. Put the documentation comments for public APIs into the
header file. Documentation comments for private APIs can go to the
implementation file. In any case, implementation files can include additional
comments (not necessarily in DocC markup) to explain implementation details as
needed.

Don't duplicate the function or class name at the beginning of the comment. For
humans, it is obvious which function or class is being documented; automatic
documentation processing tools are smart enough to bind the comment to the
correct declaration.

#### Error and Warning Messages

Clear diagnostic messages are important to help users identify and fix issues in
their inputs. Use succinct but correct English prose that gives the user the
context needed to understand what went wrong. Also, to match error message
styles commonly produced by other tools, start the first sentence with a
lowercase letter, and finish the last sentence without a period, if it would end
in one otherwise. Sentences which end with different punctuation, such as
"did you forget ';'?", should still do so.

For example, this is a good error message:

```
error: section header 3 is corrupt. Size is 10 when it should be 20
```

This is a bad message, since it does not provide useful information and uses the
wrong style:

```
error: Corrupt section header.
```

#### `#include` / `#import` Style

Immediately after the [header file comment](#file-headers) (and include guards
if working on a header file), the minimal list of `#include`s required by the
file should be listed. We prefer these `#includes` to be listed in this order:

1. Main Module Header
2. Local/Private Headers
3. Other project headers

and each category should be sorted lexicographically by the full path.

The `Main Module Header` file applies to implementation files which implement an
interface defined by a `.h` file. This `#include` should always be included
first regardless of where it lives on the file system. By including a header
file first in the implementation files that implement the interfaces, we ensure
that the header does not have any hidden dependencies which are not explicitly
included in the header, but should be. It is also a form of documentation in the
implementation file to indicate where the interfaces it implements are defined.

#### Source Code Width

Write your code to fit within 80 columns.

There must be some limit to the width of the code in order to allow developers
to have multiple files side-by-side in windows on a modest display. If you are
going to pick a width limit, it is somewhat arbitrary, but you might as well
pick something standard. Going with 90 columns (for example) instead of 80
columns wouldn't add any significant value and would be detrimental to printing
out code. Also many other projects have standardized on 80 columns, so some
people have already configured their editors for it (vs something else, like 90
columns).

Documentation files should also be wrapped to 80 columns, just like source code
files.

DocC generally ignores single line breaks within a paragraph, so wrapping text
at 80 columns does not affect the rendered output. Keeping lines within 80
columns improves readability in editors and terminals, while maintaining
consistency with the formatting conventions used throughout the codebase.

#### Whitespace

In all cases, prefer spaces to tabs in source files with an indentation width of
two spaces.

As always, follow the Golden Rule above: follow the style of existing code if
you are modifying and extending it.

Do not add trailing whitespace. Some common editors will automatically remove
trailing whitespace when saving a file which causes unrelated changes to appear
in diffs and commits.

### Language and Compiler Issues

#### Treat Compiler Warnings Like Errors

Compiler warnings are often useful and help improve the code. Those that are not
useful, can be often suppressed with a small code change.

#### Write Portable Code

In almost all cases, it is possible to write completely portable code. When you
need to rely on non-portable code, put it behind a well-defined and
well-documented interface.

#### Do not use Exceptions

Exceptions are resource-intensive in Objective-C. You should not use exceptions
for general flow-control, or simply to signify errors. Instead you should use
the return value of a method or function to indicate that an error has occurred,
and provide information about the problem in an error object.

#### Use Auto Type Deduction to Make Code More Readable

Use `let` (an alias for `__auto_type`) to make the code more readable or easier
to maintain.

## Style Issues

### The High-Level Issues

#### Self-contained Headers

Header files should be self-contained (compile on their own) and end in `.h`.

Users and refactoring tools should not have to adhere to special conditions to
include the header. Specifically, a header should have header guards and include
all other headers it needs.

In general, a header should be implemented by one or more implementation files.
Each of these implementation files should include the header that defines their
interface first. This ensures that all of the dependencies of the header have
been properly added to the header itself, and are not implicit.

#### `#include` What You Use

For every symbol (type, function, variable, or macro) that you use in a
implementation file either the implementation file or header file should include
a `.h` file that exports the declaration of that symbol. Obviously symbols
defined in implementation file itself are excluded from this requirement.

This puts us in a state where every file includes the headers it needs to
declare the symbols that it uses. When every file includes what it uses, then it
is possible to edit any file and remove unused headers, without fear of
accidentally breaking the upwards dependencies of that file. It also becomes
easy to automatically track and update dependencies in the source code.

#### Keep "Internal" Headers Private

Many modules have a complex implementation that causes them to use more than one
implementation file. It is often tempting to put the internal communication
interface (helper classes, extra functions, etc.) in the public module header
file. Don't do this!

If you really need to do something like this, put a private header file in the
same directory as the source files, and include it locally. This ensures that
your private interface remains private and undisturbed by outsiders.

> It's okay to put extra implementation methods in a public class itself. Just
> make them private (or protected) and all is well.

#### Use Early Exits and `continue` to Simplify Code

When reading code, keep in mind how much state and how many previous decisions
have to be remembered by the reader to understand a block of code. Aim to reduce
indentation where possible when it doesn't make it more difficult to understand
the code. One great way to do this is by making use of early exits and the
`continue` keyword in long loops. Consider this code that does not use an early
exit:

```objective-c
- (Value*)doSomethingWithInstruction:(Instruction* i) {
  if (!i.isTerminator && i.hasOneUse && [self doOtherThingWithInstruction:i]) {
    /* ... some long code ... */
  }

  return nil;
}
```

This code has several problems if the body of the `if` is large. When you're
looking at the top of the function, it isn't immediately clear that this only
does interesting things with non-terminator instructions, and only applies to
things with the other predicates. Second, it is relatively difficult to describe
(in comments) why these predicates are important because the if statement makes
it difficult to lay out the comments. Third, when you're deep within the body of
the code, it is indented an extra level. Finally, when reading the top of the
function, it isn't clear what the result is if the predicate isn't true; you
have to read to the end of the function to know that it returns `nil`.

It is much preferred to format the code like this:

```objective-c
- (Value*)doSomethingWithInstruction:(Instruction* i) {
  /* Terminators never need 'something' done to them because ... */
  if (i.isTerminator) {
    return nil;
  }

  /*
   * We conservatively avoid transforming instructions with multiple uses
   * because goats like cheese.
   */
  if (!i.hasOneUse) {
    return nil;
  }

  /* This is really just here for example. */
  if (![self doOtherThingWithInstruction:i]) {
    return nil;
  }

  /* ... some long code ... */
}
```

This fixes these problems. A similar problem frequently happens in for loops. A
silly example is something like this:

```objective-c
for (let i = 0; i < array.count; i += 1) {
  if (array[i]) {
    let lhs = array[i].lhs;
    let rhs = array[i].rhs;
    if (lhs != rhs) {
      /* ... */
    }
  }
}
```

When you have very, very small loops, this sort of structure is fine. But if it
exceeds more than 10-15 lines, it becomes difficult for people to read and
understand at a glance. The problem with this sort of code is that it gets very
nested very quickly. This means that the reader of the code has to keep a lot of
context in their brain to remember what is going immediately on in the loop,
because they don't know `if`/when the `if` conditions will have `else`s etc. It
is strongly preferred to structure the loop like this:

```objective-c
for (let i = 0; i < array.count; i += 1) {
  let item = array[i];
  if (!item) {
    continue;
  }

  let lhs = array[i].lhs;
  let rhs = array[i].rhs;
  if (lhs == rhs) {
    continue;
  }

  /* ... */
}
```

This has all the benefits of using early exits for functions: it reduces the
nesting of the loop, it makes it easier to describe why the conditions are true,
and it makes it obvious to the reader that there is no else coming up that they
have to push context into their brain for. If a loop is large, this can be a big
understandability win.

#### Don't use `else` after a `return`

For similar reasons as above (reduction of indentation and easier reading),
please do not use `else` or `else if` after something that interrupts control
flow — like `return`, `break`, `continue`, `goto`, etc.

The idea is to reduce indentation and the amount of code you have to keep track
of when reading the code.

#### Turn Predicate Loops into Predicate Functions

It is very common to write small loops that just compute a boolean value. There
are a number of ways that people commonly write these, but an example of this
sort of thing is:

```objective-c
let isFooFound = no;
for (let i = 0; i < array.count; i += 1) {
  if (array[i].isFoo) {
    isFooFound = yes;

    break;
  }
}

if (isFooFound) {
  /* ... */
}
```

Instead of this sort of loop, we prefer to use a predicate function that uses
early exits:

```objective-c
/* Returns true if the specified list has an element that is a foo. */
- (CBoolean)containsFoo {
  for (let i = 0; i < self.array.count; i += 1) {
    if (array[i].isFoo) {
      return yes;
    }
  }

  return no;
}

/* ... */

if ([self containsFoo]) {
  /* ... */
}
```

There are many reasons for doing this: it reduces indentation and factors out
code which can often be shared by other code that checks for the same predicate.
More importantly, it _forces you to pick a name_ for the function, and forces
you to write a comment for it. In this silly example, this doesn't add much
value. However, if the condition is complex, this can make it a lot easier for
the reader to understand the code that queries for this predicate. Instead of
being faced with the in-line details of how we check to see if the array
contains a foo, we can trust the function name and continue reading with better
locality.

### The Low-Level Issues

#### Name Types, Functions, Variables, and Enumerators Properly

Poorly-chosen names can mislead the reader and cause bugs. We cannot stress
enough how important it is to use descriptive names. Pick names that match the
semantics and role of the underlying entities, within reason. Avoid
abbreviations unless they are well known. After picking a good name, make sure
to use consistent capitalization for the name, as inconsistency requires clients
to either memorize the APIs or to look it up to find the exact spelling.

In general, names should be in camel case (e.g. `FoundationArray` and
`isLValue()`). Different kinds of declarations have different rules:

- Type names (including classes, structs, enums, typedefs, etc) should be nouns
  and start with the package prefix (e.g. `FoundationArray`).
- Variable names should be nouns (as they represent state). The name should be
  camel case, and start with an lower-case letter (e.g. `leader` or `boats`).
- Function names should be verb phrases (as they represent actions), and
  command-like function should be imperative. The name should be camel case, and
  start with a lowercase letter (e.g. `openFile:` or `isFoo`).
- Enum declarations (e.g. `enum CFoo { ... }`) are types, so they should follow
  the naming conventions for types.
- Enumerators (e.g. `enum { kFoo, kBar }`) should start with character `k`
  followed by an upper-case letter. Unless the enumerators are defined in their
  own small namespace, enumerators should have a prefix corresponding to the
  enum declaration name. For example, `enum CValue { ... }`; may contain
  enumerators like `kCValueArgument`, `kCValueBasicBlock`, etc.

### Microscopic Details

This section describes preferred low-level formatting guidelines along with
reasoning on why we prefer them.

#### Spaces Before Parentheses

Put a space before an open parenthesis only in control flow statements, but not
in normal function call expressions and function-like macros. For example:

```objective-c
if (yes) /* ... */
for (ley i = 0; i < 100; i += 1) /* ... */
while (yes) /* ... */

someFunction(42);
```

The reason for doing this is not completely arbitrary. This style makes control
flow operators stand out more, and makes expressions flow better.

#### Restrict Visibility

Functions and variables should have the most restricted visibility possible.

#### Prefer Dot Syntax for Objective-C Properties

Use dot syntax to access or set Objective-C properties. For example:

```objective-c
if ([object isKindOfClass:FoundationString.class]) { /* ... */

object.property = newValue;
```

This is preferred over the equivalent message send syntax, as dot syntax is more
concise and consistent.

#### Use Unix line endings for files

Use Unix line endings for all files.

## See Also

A lot of these comments and recommendations have been culled from other sources.
Two particularly important sources for our work are:

1. [LLVM Coding Standards](https://llvm.org/docs/CodingStandards.html)
2. [Programming with Objective-C - Conventions](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/Conventions/Conventions.html)
