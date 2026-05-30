# Add Environment Detection Macros in the CKit

* Proposal: [00000001](00000001-environment-detection-macros-in-c-kit.md)
* Implementation: [fang-ling/c-kit#5](https://github.com/fang-ling/c-kit/pull/5)

## Summary of changes

Adds standardized environment detection macros in the CKit, enabling
straightforward and consistent way in checking environments for cross-platform C
and Objective-C codes.

## Motivation

Cross-platform C and Objective-C development requires reliable detection of the
target operating system and CPU architecture. Currently, developers must
navigate a fragmented landscape of compiler-specific macros:

- Apple platforms use `TARGET_OS_*` macros from `TargetConditionals.h`
- Linux uses `__linux__` or `__linux`
- FreeBSD uses `__FreeBSD__`
- WASI uses `__wasi__`
- Online judges use `ONLINE_JUDGE`
- Architecture detection varies between `__arm64__`, `__x86_64__`,
  `__wasm32__`, and others

This fragmentation leads to several problems: developers must remember different
macro naming conventions for each platform and keep track of which compiler
defines which macro; and complex nested conditionals are needed to cover all
platforms, reducing readability and maintainability.

For example, detecting Apple platforms with ARM64 architecture currently
requires understanding multiple layers of macros:

```c
#if defined(__APPLE__) && defined(__arm64__)
  /* Apple Silicon specific code */
#endif
```

A unified, predictable macro system would simplify cross-platform code and
reduce the likelihood of conditional compilation errors.

## Proposed solution

We propose introducing two categories of environment detection macros in CKit:

### `C_TARGET_OS_*` — Operating System Detection

| Macro | Platform | Value |
|-------|----------|-------|
| `C_TARGET_OS_APPLE` | Apple platforms | `1` on match, `0` otherwise |
| `C_TARGET_OS_LINUX` | Linux | `1` on match, `0` otherwise |
| `C_TARGET_OS_FREEBSD` | FreeBSD | `1` on match, `0` otherwise |
| `C_TARGET_OS_WASI` | WASI | `1` on match, `0` otherwise |
| `C_TARGET_OS_ONLINE_JUDGE` | Online Judge | `1` on match, `0` otherwise |

### `C_TARGET_ARCHITECTURE_*` — Target Architecture Detection

| Macro | Architecture | Value |
|-------|--------------|-------|
| `C_TARGET_ARCHITECTURE_ARM64` | ARM64 | `1` on match, `0` otherwise |
| `C_TARGET_ARCHITECTURE_X86_64` | x86_64 | `1` on match, `0` otherwise |
| `C_TARGET_ARCHITECTURE_WASM32` | Wasm 32-bit | `1` on match, `0` otherwise |

### Always-defined semantics

All macros are **always defined** with integer values (`0` or `1`), enabling
straightforward `#if` usage:

```c
/* Clean, predictable conditional compilation. */
#if C_TARGET_OS_APPLE
#  include <Foundation/Foundation.h>
#elif C_TARGET_OS_LINUX
#  include <pthread.h>
#endif

#if C_TARGET_ARCHITECTURE_ARM64
  void optimize_for_neon(void);
#elif C_TARGET_ARCHITECTURE_X86_64
  void optimize_for_avx(void);
#endif

/* Combined conditions are natural. */
#if C_TARGET_OS_APPLE && C_TARGET_ARCHITECTURE_ARM64
  /* Apple Silicon specific code. */
#endif
```

This approach eliminates the confusion between `#ifdef` (macro existence check)
and `#if` (value check), providing a single consistent pattern across all
platform and architecture detection.

## Detailed design

All new macros are added to the existing `CBase.h`:

```c
/* Apple platforms. */
#if defined(__APPLE__)
#  define C_TARGET_OS_APPLE 1
#else
#  define C_TARGET_OS_APPLE 0
#endif

/* Linux. */
#if defined(__linux__)
#  define C_TARGET_OS_LINUX 1
#else
#  define C_TARGET_OS_LINUX 0
#endif

/* FreeBSD. */
#if defined(__FreeBSD__)
#  define C_TARGET_OS_FREEBSD 1
#else
#  define C_TARGET_OS_FREEBSD 0
#endif

/* WASI. */
#if defined(__wasi__)
#  define C_TARGET_OS_WASI 1
#else
#  define C_TARGET_OS_WASI 0
#endif

/* Online Judge platforms. */
#if defined(ONLINE_JUDGE)
#  define C_TARGET_OS_ONLINE_JUDGE 1
#else
#  define C_TARGET_OS_ONLINE_JUDGE 0
#endif

/* ARM64 architecture. */
#if defined(__arm64__)
#  define C_TARGET_ARCHITECTURE_ARM64 1
#else
#  define C_TARGET_ARCHITECTURE_ARM64 0
#endif

/* x86_64 architecture. */
#if defined(__x86_64__)
#  define C_TARGET_ARCHITECTURE_X86_64 1
#else
#  define C_TARGET_ARCHITECTURE_X86_64 0
#endif

/* WebAssembly 32-bit architecture. */
#if defined(__wasm32__)
#  define C_TARGET_ARCHITECTURE_WASM32 1
#else
#  define C_TARGET_ARCHITECTURE_WASM32 0
#endif
```

## Source compatibility

This proposal is purely additive. It introduces new macros that do not conflict
with existing code.

## Implications on adoption

This feature can be freely adopted in source code with no deployment
constraints.

## Future directions

### Additional operating systems and architectures

Future proposals may extend the detection macros to cover additional platforms
and architectures. These were not included in this proposal as the current focus
is on the primary target platforms for the framework.

### Feature detection macros

Beyond platform and architecture, future work could include feature detection
macros.

Feature detection is inherently more complex, as features may vary even within
the same architecture.

## Alternatives considered

### Use `#ifdef` with undefined macros

The traditional approach leaves macros undefined when the condition is false:

```c
#ifdef C_TARGET_OS_APPLE
  // Apple-specific code
#endif
```

This creates inconsistency with macros that use `0`/`1` values, leading to
confusion about whether to use `#ifdef` or `#if`. The always-defined approach
provides a single, consistent pattern that works everywhere. Furthermore,
`#ifdef` prevents use in expressions:

```c
/* Not possible with #ifdef-style macros. */
#define IS_APPLE_SILICON (C_TARGET_OS_APPLE && C_TARGET_ARCHITECTURE_ARM64)
```

### Enum-like integer constants

Define a single macro that expands to an integer constant:

```c
#define C_TARGET_OS 1  /* 1=Apple, 2=Linux, 3=FreeBSD, 4=WASI */

#if C_TARGET_OS == 1
  /* Apple */
#elif C_TARGET_OS == 2
  /* Linux */
#endif
```

This makes combined platform/architecture checks awkward and reduces
readability:

```c
/* Compare to the cleaner boolean approach */
#if C_TARGET_OS == 1 && C_TARGET_ARCHITECTURE == 1
  /* Apple ARM64 */
#endif

/* vs. */

#if C_TARGET_OS_APPLE && C_TARGET_ARCHITECTURE_ARM64
  /* Apple ARM64 */
#endif
```

Separate boolean macros are clearer and more composable.
