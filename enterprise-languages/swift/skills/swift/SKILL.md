---
schema: "1.0"
name: swift
version: "1.0.0"
description: Swift ARC/retain cycles, optionals, value vs reference types, and SwiftUI state management practices
domain: technology
triggers:
  keywords:
    primary: [swift, ios, swiftui, xcode]
    secondary: [optional, ARC, cocoapods, spm, retain cycle]
  context_boost: [ios app, macos app, apple platform]
  context_penalty: [kotlin, java, android]
  priority: high
dependencies:
  software-skills: []
author: claude-domain-skills
---

# Swift

> ARC removes manual memory management but not the retain-cycle bug class — closures still need `[weak self]`

## Applicable Scenarios

- Writing or reviewing Swift application code for iOS/macOS
- Diagnosing retain cycles and memory leaks under ARC
- Handling optionals safely without force-unwrap crashes
- Choosing between struct (value type) and class (reference type)
- Managing SwiftUI state (`@State`, `@Binding`, `@ObservedObject`) correctly

## Core Knowledge

### Value Types vs. Reference Types

| Type | Semantics |
|------|------|
| **`struct`/`enum`** | Value type — copied on assignment/pass; mutations to a copy don't affect the original |
| **`class`** | Reference type — assignment/pass shares the same instance; mutations are visible everywhere that reference points |

Swift's standard library leans heavily on value types (`Array`, `String`, `Dictionary` are all structs) with copy-on-write optimization under the hood — they behave like they're copied, but the actual data buffer is only duplicated when a mutation would otherwise affect a shared copy.

### ARC (Automatic Reference Counting)

- Class instances are reference-counted; when the count hits zero, the instance is deallocated — this happens at compile-determined points, not via a garbage collector pass
- A **retain cycle** (two objects holding strong references to each other, directly or via a closure) means the count never reaches zero, and neither object is ever deallocated — a memory leak ARC can't detect or break on its own
- Closures capture referenced variables **strongly by default** — a closure stored as a property that captures `self` creates a cycle if `self` also (directly or indirectly) holds that closure

### Optionals

- `Optional<T>` (`T?`) makes the absence of a value part of the type system — a genuine improvement over implicit null, but only if it's actually respected
- Force-unwrapping (`!`) tells the compiler "trust me, this has a value" — if it doesn't, the app crashes immediately with a runtime trap, not a recoverable error
- `guard let`/`if let` (and newer shorthand `if let value` binding to the same name) safely unwrap; `??` provides a default

### SwiftUI State

| Property Wrapper | Ownership |
|------|------|
| **`@State`** | View-owned, local, value-type source of truth |
| **`@Binding`** | A reference to state owned by a parent view |
| **`@ObservedObject`/`@StateObject`** | Reference-type (class, `ObservableObject`) state — `@StateObject` for state the view creates and owns, `@ObservedObject` for state passed in from outside |

Using `@ObservedObject` for state a view itself creates (instead of `@StateObject`) can cause the object to be recreated on every parent re-render, silently losing state.

## Best Practices

1. **Use `[weak self]` (or `[unowned self]` when lifetime is truly guaranteed) in closures stored as properties or held longer than the immediate call**, to avoid retain cycles
2. **Avoid force-unwrapping (`!`) outside of contexts where a nil value is truly a programmer error**, not an expected runtime condition
3. **Prefer `struct` for data models** unless reference semantics (shared mutable identity) are genuinely needed
4. **Use `@StateObject` for objects a view creates and owns**, `@ObservedObject` only for objects passed in from a parent
5. **Perform UI updates on the main thread** — `DispatchQueue.main.async` (or `@MainActor` in Swift concurrency) for anything touching UI from a background context
6. **Use Instruments (Leaks/Allocations)** periodically to catch retain cycles that unit tests won't surface

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| Closure stored as a property capturing `self` strongly | Capture `[weak self]` and safely unwrap inside the closure body |
| Force-unwrapping a value from a network response or user input | Use `guard let`/`if let` and handle the nil case explicitly |
| Using `@ObservedObject` for a view-owned object | Use `@StateObject` so the object survives parent re-renders |
| Updating `@Published`/UI state from a background thread | Dispatch to the main thread (`@MainActor` or `DispatchQueue.main`) |
| Treating a `class` as if it had value semantics | Use `struct` if independent copies are actually the intended behavior |

## Sharp Edges

### SE-1: Retain Cycles From Strong Closure Capture
- **Severity**: high
- **Situation**: A view controller or object stores a closure (e.g., a completion handler or callback) as a property, the closure captures `self` to call a method later, and `self` also (directly or transitively) holds a reference to that closure — neither ever gets deallocated
- **Cause**: Swift closures capture referenced variables strongly by default; a bidirectional strong reference (object → closure → object) is exactly what ARC can't break on its own
- **Symptoms**:
  - A view controller/object that should be deallocated (e.g., after being dismissed) never triggers `deinit`
  - Instruments' Leaks/Allocations tool shows growing instance counts for a type that should be transient
- **Solution**: Capture `[weak self]` in closures stored as properties or held beyond the immediate scope, safely unwrap inside (`guard let self else { return }`), and periodically profile with Instruments to catch cycles unit tests won't reveal
- **Details**: → [extended/checklists.md#memory-safety-checklist]

### SE-2: Force-Unwrap Crashes in Production
- **Severity**: critical
- **Situation**: Code force-unwraps (`!`) a value that's nil in a real-world scenario the developer didn't anticipate (a network response missing a field, a user-provided input, a dictionary lookup miss), and the app crashes immediately with no graceful degradation
- **Cause**: Force-unwrap tells the compiler to skip the safety check and trust the value is present — if it isn't, Swift traps immediately (equivalent to a fatal error), there's no exception to catch
- **Symptoms**:
  - Crash reports show a fatal error at a force-unwrap site, often correlated with a specific edge-case input or network condition
- **Solution**: Reserve force-unwrap for cases that are genuinely programmer invariants (verified elsewhere in the same scope), and use `guard let`/`if let`/`??` for anything derived from external input (network, user, file I/O)

### SE-3: `@ObservedObject` Losing State on Re-Render
- **Severity**: medium
- **Situation**: A SwiftUI view creates its own `ObservableObject` instance and marks it `@ObservedObject`, and whenever the parent view re-renders, the child view's body re-executes, recreating the object and silently resetting its state
- **Cause**: `@ObservedObject` doesn't take ownership — it just observes whatever instance it's given, and if the view's `body` recreates that instance on every evaluation (common when it's initialized inline in the view struct), SwiftUI has no way to know to preserve the old instance
- **Symptoms**:
  - Form input, scroll position, or other view-local state managed by the object mysteriously resets during normal navigation/re-render
- **Solution**: Use `@StateObject` for any `ObservableObject` the view itself creates and is meant to own for its lifetime; reserve `@ObservedObject` strictly for objects passed in from a parent that already owns them via `@StateObject` or another persistent source

### SE-4: Struct Mutation Semantics Surprises
- **Severity**: medium
- **Situation**: A `struct` instance stored in an array or passed to a function is mutated, and the change doesn't appear where the developer expected — because a copy, not the original, was mutated
- **Cause**: Value types copy on assignment/pass; mutating a local copy (e.g., a `for` loop variable, or a struct captured by value in a closure) never affects the original — this is by design, but it's a common source of confusion for anyone expecting reference semantics
- **Symptoms**:
  - A mutation inside a loop or function "doesn't take effect" on the caller's original data
  - Debugging reveals the code was operating on a copy the whole time
- **Solution**: Understand where a value type is being copied (function parameters, loop iteration variables, closure captures) versus where `inout`/index-based mutation is needed to affect the original, and use `inout` parameters or direct index mutation (`array[i].property = ...`) when in-place mutation of the original is actually required

### SE-5: Background-Thread UI Updates Causing Undefined Behavior
- **Severity**: high
- **Situation**: UI state (`@Published` properties, direct UIKit/AppKit view updates) is mutated from a background thread/task, and the app exhibits intermittent glitches, crashes, or UI corruption that don't reproduce consistently
- **Cause**: UIKit/AppKit and SwiftUI's rendering are not thread-safe — updates must happen on the main thread, and violating this is undefined/unpredictable behavior rather than a guaranteed crash, making it easy to miss in testing
- **Symptoms**:
  - Intermittent visual glitches or crashes that correlate with background work (network callbacks, async tasks) updating UI-bound state
  - Xcode's Main Thread Checker flags a UI API call from a background thread
- **Solution**: Mark UI-updating classes/properties `@MainActor` (Swift concurrency) or explicitly dispatch updates via `DispatchQueue.main.async`, and enable the Main Thread Checker in Xcode to catch violations during development rather than relying on production crash reports

## Recommended Tools

| Category | Tools |
|------|------|
| Dependency management | Swift Package Manager, CocoaPods |
| Debugging/profiling | Instruments (Leaks, Allocations), Main Thread Checker |
| Testing | XCTest, Swift Testing |
| Static analysis | SwiftLint |

## Memory Safety

**Full checklist**: → [extended/checklists.md#memory-safety-checklist]

## Related Resources

[Swift Docs](https://www.swift.org/documentation/) | [Swift Language Guide](https://docs.swift.org/swift-book/) | [Human Interface Guidelines](https://developer.apple.com/design/human-interface-guidelines)

## Related Domains

[[kotlin]] | [[javascript]]
