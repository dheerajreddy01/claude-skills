# Swift — Extended Checklists

## Memory Safety Checklist

- [ ] Closures stored as properties or held beyond the immediate call use `[weak self]`/`[unowned self]` where a retain cycle is possible
- [ ] `deinit` verified to actually fire for view controllers/objects expected to be transient (via logging or Instruments)
- [ ] Instruments' Leaks/Allocations run periodically against real usage flows, not just unit tests
- [ ] Force-unwrap (`!`) reserved for genuine programmer invariants; external/network/user-derived values use `guard let`/`if let`
- [ ] `@StateObject` used for objects a SwiftUI view creates and owns; `@ObservedObject` only for objects passed in from a parent
- [ ] UI-updating code marked `@MainActor` or explicitly dispatched to the main thread
- [ ] Xcode's Main Thread Checker and Thread Sanitizer enabled during development/testing

## Production Readiness Checklist

- [ ] Struct vs. class chosen deliberately per type based on whether value or reference semantics are actually needed
- [ ] `inout` parameters or index-based mutation used where in-place mutation of an original value type is required
- [ ] SwiftLint run in CI for style/safety rule enforcement
- [ ] Crash reporting (e.g., via a crash analytics SDK) configured and monitored, with force-unwrap sites treated as priority fixes when they surface
- [ ] Dependency versions pinned via Swift Package Manager/CocoaPods lockfiles
- [ ] Async/await (Swift concurrency) used consistently rather than mixing completion-handler and async styles within the same module without a clear boundary
