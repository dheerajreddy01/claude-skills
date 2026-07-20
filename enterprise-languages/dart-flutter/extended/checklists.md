# Dart / Flutter — Extended Checklists

## Widget Lifecycle Checklist

- [ ] Every async callback that touches `setState` checks `mounted` first
- [ ] Every `AnimationController`, `StreamSubscription`, and `TextEditingController` created in a State is disposed/cancelled in `dispose()`
- [ ] `const` constructors used only where every dependency is a genuine constructor argument, not an ambient/inherited read
- [ ] Widget rebuild scope verified with DevTools' "Track Widget Rebuilds" for any screen with visible jank
- [ ] Platform channel calls wrapped in try/catch with explicit `PlatformException` handling
- [ ] Long-running async operations have a cancellation path tied to widget disposal
- [ ] State management choice (setState/Provider/Riverpod/Bloc) matches the actual complexity of the screen/app, not over- or under-engineered

## Production Readiness Checklist

- [ ] Native crash logs (Xcode/Android Studio) checked alongside Dart logs when platform channel behavior is unexpected
- [ ] Memory profiler run on a screen that's pushed/popped repeatedly to catch controller/subscription leaks before release
- [ ] Null safety fully adopted (no lingering legacy non-null-safe dependencies) for new code
- [ ] Build flavors/environment config (dev/staging/prod) set up explicitly, not hardcoded per build
- [ ] Crash reporting (Sentry/Firebase Crashlytics) wired in before first production release
- [ ] App size and startup time benchmarked, not just functional correctness
