---
schema: "1.0"
name: dart-flutter
version: "1.0.0"
description: Dart/Flutter widget tree model, state management, async patterns, and platform interop production practices
domain: technology
triggers:
  keywords:
    primary: [dart, flutter, widget, statefulwidget]
    secondary: [riverpod, bloc, provider, platform channel, hot reload]
  context_boost: [mobile app, cross-platform, ios android]
  context_penalty: [swift, kotlin, react native]
  priority: high
dependencies:
  software-skills: []
author: claude-domain-skills
---

# Dart / Flutter

> The widget tree is the whole mental model — rebuild scope and disposal discipline decide whether it stays fast

## Applicable Scenarios

- Designing or reviewing Flutter widget trees and state management approach
- Choosing between Provider/Riverpod/Bloc for a given app's complexity
- Diagnosing unnecessary rebuilds, jank, or memory leaks
- Wiring platform channels for native iOS/Android interop
- Debugging async/Future/Stream lifecycle issues tied to widget disposal

## Core Knowledge

### The Widget Tree

- Flutter's entire UI is a tree of immutable widget *descriptions*; the framework diffs this tree against the previous one (via Elements) and only touches the RenderObjects that actually changed
- **StatelessWidget**: no mutable state, rebuilds only when its inputs (constructor parameters) change or a parent rebuilds it
- **StatefulWidget**: pairs with a `State` object that survives rebuilds; `setState()` marks that State's subtree dirty for the next frame — it does not immediately re-render
- Rebuild scope is determined by *where* `setState` is called and how widgets are composed — calling it high in the tree rebuilds everything below unless children are `const` or otherwise cheaply diffed

### Async & Disposal

- `Future`/`async`/`await` work as expected, but a widget can be disposed *while* an async operation it started is still in flight — calling `setState` after disposal throws
- `StreamSubscription`s, `AnimationController`s, and `TextEditingController`s are **not** garbage collected just because the widget using them is — they must be explicitly cancelled/disposed in `State.dispose()`

### State Management Landscape

| Approach | Best For |
|------|------|
| **`setState` (built-in)** | Small, localized state, simple widgets |
| **Provider** | Straightforward DI + reactive rebuilds, moderate app size |
| **Riverpod** | Compile-time-safe DI, testable, scales to large apps |
| **Bloc** | Strict unidirectional event→state flow, large teams wanting enforced structure |

None of these replace understanding the widget rebuild model — they're ways of scoping *where* state lives and *what* rebuilds when it changes.

### Platform Channels

- `MethodChannel`/`EventChannel` bridge Dart and native (Swift/Kotlin) code asynchronously over a binary message protocol
- Native-side exceptions don't automatically surface as clean Dart exceptions — errors can silently fail to deliver a result or crash the native side with a stack trace that never reaches Dart-side logs

## Best Practices

1. **Scope `setState` as low in the tree as possible** — extract stateful pieces into small widgets rather than calling `setState` in a large parent
2. **Always dispose controllers and cancel subscriptions** in `State.dispose()` — treat this as mandatory, not optional cleanup
3. **Guard async callbacks with a `mounted` check** before calling `setState` after an `await`
4. **Use `const` constructors** wherever a widget's output genuinely doesn't depend on changing data — this lets Flutter skip rebuilding it entirely
5. **Pick a state management approach that matches app size** — don't reach for Bloc's ceremony for a small app, don't rely on raw `setState` sprawl in a large one
6. **Test platform channel error paths explicitly** — don't assume native failures will surface cleanly on the Dart side

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| Calling `setState` after `await` without checking `mounted` | Check `if (!mounted) return;` before any post-await `setState` |
| Not disposing `AnimationController`/`StreamSubscription` | Always cancel/dispose in `State.dispose()` |
| Putting one giant `StatefulWidget` around the whole screen | Extract small, focused stateful widgets scoped to what actually changes |
| Rebuilding expensive child widgets that never change | Mark them `const` or extract to a separate widget that isn't rebuilt |
| Assuming platform channel calls always succeed | Wrap platform calls in try/catch and handle `PlatformException` explicitly |

## Sharp Edges

### SE-1: `setState` Called After Widget Disposal
- **Severity**: high
- **Situation**: An async operation (network call, timer) started by a widget completes after the user has navigated away and the widget was disposed, and the completion handler calls `setState`, throwing an exception
- **Cause**: Dart's async operations don't know or care about the Flutter widget lifecycle — a `Future` keeps running regardless of whether the widget that started it still exists
- **Symptoms**:
  - `setState() called after dispose()` exceptions in logs, often intermittent and hard to reproduce on demand
  - Crashes correlate with fast navigation (user leaves a screen before a network call finishes)
- **Solution**: Check `if (!mounted) return;` immediately after any `await` before touching `setState` or widget-scoped state, and consider cancellable operations (e.g., a cancel token) for long-running work
- **Details**: → [extended/checklists.md#widget-lifecycle-checklist]

### SE-2: Undisposed Controllers Leaking Memory
- **Severity**: high
- **Situation**: A screen using `AnimationController`, `StreamSubscription`, or `TextEditingController` is pushed and popped repeatedly (e.g., a tab or modal reused often), and memory usage climbs because none of these were disposed
- **Cause**: These objects hold native resources or active listeners that persist independently of Dart's garbage collector — the widget being garbage collected doesn't automatically clean them up
- **Symptoms**:
  - Memory profiler shows accumulating controller/subscription instances that never decrease
  - App gradually slows down or crashes with out-of-memory after extended use
- **Solution**: Always override `dispose()` and explicitly dispose every controller and cancel every subscription created in that State, treating it as a mandatory pairing with creation

### SE-3: Unnecessary Full-Subtree Rebuilds
- **Severity**: medium
- **Situation**: A single `setState` call at a high level in the widget tree causes an entire complex screen to rebuild every frame during an animation or frequent state update, causing visible jank
- **Cause**: `setState` marks the entire subtree of that State object dirty; without `const` widgets or extracted child widgets that don't depend on the changing state, everything below rebuilds even if most of it is unaffected
- **Symptoms**:
  - Flutter DevTools' widget rebuild profiler shows widgets rebuilding that have no visual dependency on the changed state
  - Frame times spike during interactions that should be cheap
- **Solution**: Extract the actually-changing piece into its own small widget, mark unchanging child widgets `const`, and use DevTools' "Track Widget Rebuilds" to verify the fix actually narrowed rebuild scope

### SE-4: Platform Channel Failures With No Dart-Side Visibility
- **Severity**: high
- **Situation**: A native (iOS/Android) implementation behind a `MethodChannel` throws or crashes, and the Dart side either hangs waiting for a response or receives a generic `PlatformException` with no useful detail about what actually went wrong natively
- **Cause**: Platform channels serialize a limited set of error information across the Dart/native boundary — a native crash or an unhandled native exception doesn't automatically produce a rich, catchable Dart-side stack trace
- **Symptoms**:
  - `PlatformException` caught with a vague message, while the real error is only visible in native (Xcode/Android Studio) logs
  - Native-side crashes that don't surface as a normal Dart exception at all
- **Solution**: Wrap all native implementations to catch and translate exceptions into well-defined `PlatformException` codes/messages before returning across the channel, and always check native-side logs (not just Dart logs) when a platform channel call behaves unexpectedly

### SE-5: `const` Constructor Misuse Preventing or Breaking Expected Rebuilds
- **Severity**: medium
- **Situation**: A widget marked `const` (or wrapped in one) doesn't update visually even though the data it displays actually changed, because Flutter correctly determined the `const` widget instance is identical and skipped rebuilding it
- **Cause**: `const` constructors tell Flutter "this widget's output is fully determined at compile time and will never change for this instance" — using `const` on a widget whose apparent data actually comes from something outside its constructor parameters (e.g., an ambient inherited value it reads internally) breaks the framework's rebuild assumption
- **Symptoms**:
  - A widget's displayed content is stale despite the underlying data changing
  - Removing `const` "fixes" the bug, confirming the widget wasn't actually eligible for it
- **Solution**: Only mark widgets `const` when every value they depend on is genuinely part of their constructor arguments and immutable for that instance; when in doubt, don't use `const` and rely on the normal rebuild-and-diff mechanism, then optimize deliberately once correctness is confirmed

## Recommended Tools

| Category | Tools |
|------|------|
| DevTools | Flutter DevTools (widget rebuild profiler, memory view) |
| State management | Riverpod, Bloc, Provider |
| Testing | flutter_test, integration_test, mockito |
| CI/CD | Codemagic, Fastlane, GitHub Actions |

## Production Readiness

**Full checklist**: → [extended/checklists.md#widget-lifecycle-checklist]

## Related Resources

[Flutter Docs](https://docs.flutter.dev/) | [Dart Language Tour](https://dart.dev/language) | [Flutter Widget of the Week](https://www.youtube.com/playlist?list=PLjxrf2q8roU23XGwz3Km7sQZFTdB996iG)

## Related Domains

[[swift]] | [[kotlin]] | [[javascript]] | [[github-actions]]
