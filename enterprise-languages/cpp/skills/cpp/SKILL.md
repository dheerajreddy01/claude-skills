---
schema: "1.0"
name: cpp
version: "1.0.0"
description: C++ RAII, ownership, undefined behavior, move semantics, and build practices
domain: technology
triggers:
  keywords:
    primary: [c++, cpp, RAII, template, undefined behavior]
    secondary: [smart pointer, CMake, move semantics, STL, ODR]
  context_boost: [systems programming, performance-critical, embedded, game engine]
  context_penalty: [java, python, javascript]
  priority: high
dependencies:
  software-skills: []
author: claude-domain-skills
---

# C++

> No garbage collector, no safety net — ownership and lifetime are your job, always

## Applicable Scenarios

- Writing or reviewing C++ code for correctness and safety
- Choosing ownership models (raw pointers vs smart pointers vs references)
- Diagnosing undefined behavior, memory corruption, or crashes
- Structuring multi-file builds with CMake and avoiding ODR violations
- Reviewing move semantics and copy/move constructor correctness

## Core Knowledge

### RAII (Resource Acquisition Is Initialization)

- A resource (memory, file handle, lock, socket) is acquired in a constructor and released in the destructor — this ties resource lifetime to object scope, and is C++'s primary answer to "how do I not leak things" without a garbage collector
- Destructors run deterministically at scope exit (including during stack unwinding from an exception), which is what makes RAII reliable — unlike `finally`-style cleanup, it can't be forgotten because it's automatic

### Ownership Model

| Type | Ownership Semantics |
|------|------|
| **Raw pointer (`T*`)** | No ownership implied — use only for non-owning references (observing, not managing lifetime) |
| **`unique_ptr<T>`** | Sole ownership, moves (not copies) — the default choice for owned heap allocations |
| **`shared_ptr<T>`** | Shared ownership via reference counting — use when lifetime genuinely needs to be shared, not as a default |
| **References (`T&`)** | Non-owning, non-null (by convention) alias to an existing object |

Modern C++ style avoids raw `new`/`delete` almost entirely — ownership is expressed through `unique_ptr`/`shared_ptr` and containers, with raw pointers/references reserved for non-owning access.

### Undefined Behavior (UB)

- UB isn't "an error the compiler catches" — it's a category of program behavior the standard places zero requirements on; the compiler is free to assume it never happens, which can produce results ranging from "seems to work" to "crashes" to "optimized in a way that deletes the check that would have caught it"
- Common UB sources: signed integer overflow, dereferencing a null/dangling pointer, reading uninitialized memory, out-of-bounds array access, data races on non-atomic variables

### Move Semantics

- A **move** transfers ownership of a resource from one object to another without copying the underlying data — critical for performance with heap-allocated types (strings, vectors) and required for move-only types like `unique_ptr`
- After being moved-from, an object is left in a "valid but unspecified" state — it can be destroyed or reassigned, but its contents shouldn't be assumed to still be there

### One Definition Rule (ODR) & Builds

- The ODR requires every entity (function, class, template) to have exactly one definition across the entire program — violating it (e.g., differing inline function definitions across translation units due to mismatched compiler flags or header versions) is UB, not a guaranteed compile/link error
- CMake manages multi-target builds and dependency graphs; mismatched compiler flags or ABI-incompatible library versions linked into the same binary are a common source of ODR violations and subtle runtime corruption

## Best Practices

1. **Prefer `unique_ptr` by default**, `shared_ptr` only when shared ownership is a genuine requirement, and raw pointers/references only for non-owning access
2. **Follow the Rule of Zero** where possible — let the compiler generate special member functions by relying on RAII-managing members, rather than hand-writing copy/move constructors
3. **Enable and heed compiler warnings** (`-Wall -Wextra` at minimum) and run sanitizers (ASan, UBSan) in CI, not just in ad hoc local debugging
4. **Never assume UB "just happens to work"** — code that's technically UB can break under a different compiler, optimization level, or platform with no warning
5. **Pass by const reference for read-only access to large objects**, by value when you intend to take ownership (and can then move from the parameter)
6. **Keep translation units consistent** (compiler flags, header versions) across a build to avoid ODR violations

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| Using `new`/`delete` directly for owned heap allocations | Use `make_unique`/`make_shared` and let RAII manage lifetime |
| Returning a reference/pointer to a local (stack) variable | Return by value (relying on move semantics/RVO) instead |
| Assuming signed integer overflow wraps predictably | Treat it as UB; use unsigned types or explicit overflow-checked arithmetic where wraparound matters |
| Iterating a container while modifying it (e.g., erasing in a range-for) | Use the iterator returned by `erase()`, or a remove-erase idiom |
| Copying large objects by value in hot paths | Pass by `const&` for read access, or move when transferring ownership |

## Sharp Edges

### SE-1: Dangling References and Use-After-Free
- **Severity**: critical
- **Situation**: A function returns a reference or pointer to a local stack variable, or an object is used after the `unique_ptr`/`shared_ptr` (or scope) that owned it has released it — the program reads or writes memory that's no longer valid
- **Cause**: C++ doesn't track object lifetime at the language level beyond scope rules; nothing prevents holding onto a reference/pointer past the lifetime of what it points to
- **Symptoms**:
  - Intermittent, seemingly random crashes or corrupted values that vary by build/optimization level
  - AddressSanitizer flags a use-after-free or use-after-scope error that a normal debug build didn't catch
- **Solution**: Never return a reference/pointer to a local variable, use RAII/smart pointers to make ownership explicit so lifetime is obvious at the call site, and run AddressSanitizer in CI/testing to catch these before they reach production
- **Details**: → [extended/checklists.md#safety-checklist]

### SE-2: Undefined Behavior From Signed Integer Overflow
- **Severity**: high
- **Situation**: Code relies on signed integer overflow "wrapping around" (as it does in some other languages), and under optimization, the compiler exploits the fact that overflow is UB to eliminate a check the programmer expected to run, producing a security-relevant bug
- **Cause**: Signed integer overflow is undefined behavior in the C++ standard; compilers are permitted to assume it never happens, and optimizing compilers do act on that assumption (e.g., eliminating an overflow check that would only matter if overflow occurred)
- **Symptoms**:
  - A bounds/overflow check that "looks correct" in source is silently removed at a higher optimization level
  - UBSan flags signed overflow that a normal build's behavior masked
- **Solution**: Never rely on signed overflow behavior; use unsigned types where wraparound is intentional (which IS well-defined for unsigned), or explicit overflow-checked arithmetic (`__builtin_add_overflow` or equivalent) where overflow must be detected

### SE-3: Object Slicing on Copy Through a Base Class
- **Severity**: medium
- **Situation**: A derived-class object is copied (or passed by value) through a base-class type, and the derived-specific data is silently dropped — the object is "sliced" down to just its base-class portion
- **Cause**: Copying a derived object into a base-class-typed variable (by value, not by reference/pointer) only copies the base subobject, since the compiler only knows about the base class's layout at that point
- **Symptoms**:
  - Polymorphic behavior "disappears" after a value copy — a virtual function call resolves to the base class implementation instead of the derived override
  - Data specific to the derived class is silently lost after passing through a function taking the base type by value
- **Solution**: Pass and store polymorphic types by reference or pointer (raw for non-owning, smart pointer for owning), never by value when polymorphic behavior needs to be preserved

### SE-4: Iterator Invalidation
- **Severity**: high
- **Situation**: A container is modified (element inserted/erased, or a `vector` reallocates) while an iterator, pointer, or reference into it is still held, and continuing to use that now-invalidated iterator causes undefined behavior
- **Cause**: Different container operations invalidate iterators differently and non-obviously (e.g., `vector::push_back` invalidates all iterators if it triggers a reallocation, `vector::erase` invalidates iterators at or after the erased position) — the rules vary per container type and per operation
- **Symptoms**:
  - Crashes or corrupted data specifically when a container modification happens inside a loop that's also iterating it
  - Bug only reproduces when a `vector` happens to need to grow (capacity-dependent, so intermittent across runs/inputs)
- **Solution**: Know the specific invalidation rules for whatever container is in use, use the iterator returned by mutating operations (`erase` returns the next valid iterator) rather than a stale one, and prefer index-based access with explicit bounds re-checking when modifying a `vector` during iteration

### SE-5: ODR Violations From Mismatched Translation Units
- **Severity**: medium
- **Situation**: The same inline function, template, or class has subtly different definitions across translation units (e.g., compiled with different preprocessor macros or against different header versions), and the program links successfully but exhibits corrupted or inconsistent behavior at runtime
- **Cause**: The One Definition Rule requires every entity to have exactly one definition across the whole program; violating it is undefined behavior, and unlike many other UB categories, the linker often has no way to detect or reject it
- **Symptoms**:
  - Behavior differs depending on which translation unit's version of a function "wins" at link time, in a way that has no source-level explanation
  - The bug appears/disappears based on build configuration (debug vs release, or which files get recompiled) rather than any code change
- **Solution**: Keep compiler flags and header/library versions consistent across the entire build, be especially careful with preprocessor macros that alter class layout or function bodies conditionally across different compilation units, and use consistent build systems (CMake with a single source of truth for flags) rather than ad hoc per-file compilation settings

## Recommended Tools

| Category | Tools |
|------|------|
| Build | CMake, Ninja |
| Sanitizers | AddressSanitizer, UndefinedBehaviorSanitizer, ThreadSanitizer |
| Static analysis | clang-tidy, Cppcheck |
| Testing | GoogleTest, Catch2 |

## Memory & Build Safety

**Full checklist**: → [extended/checklists.md#safety-checklist]

## Related Resources

[cppreference.com](https://en.cppreference.com/) | [C++ Core Guidelines](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines) | [Compiler Explorer](https://godbolt.org/)

## Related Domains

[[rust]] | [[go]] | [[docker]]
