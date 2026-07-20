---
schema: "1.0"
name: r
version: "1.0.0"
description: R vectorized computing, tidyverse workflows, scoping, and statistical data practices
domain: technology
triggers:
  keywords:
    primary: [r language, rstudio, tidyverse, ggplot2]
    secondary: [dplyr, CRAN, statistical computing, data frame]
  context_boost: [statistics, data analysis, plotting, research]
  context_penalty: [python, sql, javascript]
  priority: high
dependencies:
  software-skills: []
author: claude-domain-skills
---

# R

> Vectorize or pay for it — R's performance model punishes explicit loops

## Applicable Scenarios

- Writing or reviewing R scripts/packages for data analysis or statistics
- Choosing between base R and tidyverse idioms
- Diagnosing memory blowups or slow loops on large data frames
- Managing package dependencies and reproducible environments
- Debugging factor/NA-related data surprises

## Core Knowledge

### Vectorization

- R's core performance model rewards operating on entire vectors/columns at once (`x + y`, `sapply`, `dplyr::mutate`) rather than iterating element-by-element with `for` loops
- Vectorized operations are implemented in C under the hood; an equivalent `for` loop pays R's interpreter overhead on every single iteration, which compounds dramatically at scale

### Tidyverse vs. Base R

| Approach | Style |
|------|------|
| **Base R** | `[`, `subset()`, `apply` family — no extra dependencies, sometimes more verbose |
| **Tidyverse** (dplyr, tidyr, ggplot2) | Pipe-based (`|>` or `%>%`), consistent verb-based grammar (`filter`, `mutate`, `summarize`) |

Tidyverse's consistent grammar makes code more readable and composable for data manipulation pipelines; base R remains relevant for package development (fewer dependencies) and some performance-sensitive paths.

### Scoping & Environments

- R uses **lexical scoping** — a function looks up free variables in the environment where it was *defined*, not where it's called from
- `<<-` (the "superassignment" operator) assigns in an enclosing (often global) environment rather than the local one — convenient for specific patterns (e.g., memoization closures) but a common source of unintended global state mutation when used casually

### Data Frames & Factors

- Assigning to part of a data frame or vector often triggers **copy-on-modify** — R copies the object before mutating, since R's semantics are (mostly) value-based, not reference-based; repeated in-place-looking modifications to a large data frame in a loop can copy the whole structure on every iteration
- **Factors** represent categorical data with a fixed set of levels — operations that seem to just "add a value" can silently produce `NA` if the value isn't an existing factor level, and legacy default behavior (`stringsAsFactors`) has caused significant historical confusion (changed in R 4.0+, but still relevant for older code/habits)

## Best Practices

1. **Vectorize instead of looping** wherever the operation can be expressed that way — this is usually both faster and more idiomatic
2. **Prefer tidyverse verbs for data manipulation pipelines** for readability, unless writing a package where dependency weight matters
3. **Avoid `<<-` outside of well-understood, narrow patterns** (like closures for memoization) — it creates action-at-a-distance
4. **Check factor levels explicitly** before assigning new categorical values, and be aware of `stringsAsFactors` behavior in older code/environments
5. **Use `renv` (or a similar lockfile-based tool)** for reproducible package environments across machines/CI
6. **Preallocate vectors/lists** when building up results in a loop that can't be vectorized, rather than growing them one element at a time

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| `for` loop appending to a vector/list one element at a time | Preallocate the result structure, or vectorize the operation entirely |
| Assigning a new category to a factor column without checking existing levels | Use `factor(x, levels = c(...))` explicitly, or convert to character first if levels should be dynamic |
| Using `<<-` freely across a script for "convenience" | Pass values explicitly as function arguments/return values; reserve `<<-` for specific closure patterns |
| Ignoring silent `NA` propagation through arithmetic/logical operations | Explicitly check for and handle `NA` (`is.na()`, `na.rm = TRUE` where appropriate) rather than assuming clean data |
| No dependency lockfile, relying on whatever CRAN versions happen to be installed | Use `renv::snapshot()`/`renv::restore()` for reproducible package versions |

## Sharp Edges

### SE-1: Copy-on-Modify Blowing Up Memory on Large Data Frames
- **Severity**: high
- **Situation**: A loop that repeatedly modifies a large data frame (e.g., updating one column per iteration) becomes progressively slower and consumes far more memory than the data frame's actual size would suggest
- **Cause**: R's copy-on-modify semantics mean many "in-place-looking" mutations actually create a full (or partial) copy of the object; repeating this in a loop over a large data frame multiplies both time and peak memory usage
- **Symptoms**:
  - Memory usage during a loop far exceeds the data frame's `object.size()`
  - Loop iterations get progressively slower as the loop runs
- **Solution**: Vectorize the operation across the whole column/data frame at once instead of looping, or use `data.table`'s reference-semantics modification-in-place operators when working with genuinely large datasets where copy-on-modify overhead is prohibitive
- **Details**: → [extended/checklists.md#performance-checklist]

### SE-2: Silent Factor Level Mismatches Producing NA
- **Severity**: high
- **Situation**: Assigning a new value to a factor column (or joining data with mismatched factor levels) silently produces `NA` instead of an error, because the value isn't among the factor's existing levels
- **Cause**: A factor's levels are a fixed, predefined set — assigning a value outside that set doesn't automatically extend the levels, it coerces to `NA`
- **Symptoms**:
  - Downstream analysis shows unexpected `NA` values with no corresponding warning at the point of assignment
  - A value that "should be there" (verified present in the source data) is missing after a factor-related operation
- **Solution**: Explicitly define factor levels to include all expected values before assignment, or work with character vectors during data manipulation and only convert to factors at the point where categorical semantics (e.g., for modeling or plotting) are actually needed

### SE-3: `for` Loops Instead of Vectorized Operations Causing Massive Slowdowns
- **Severity**: medium
- **Situation**: A script using explicit `for` loops for what's fundamentally a vectorizable operation (e.g., computing a value per row) runs orders of magnitude slower than the vectorized equivalent, and this only becomes a visible problem once the dataset grows
- **Cause**: R's interpreter has meaningful per-iteration overhead; vectorized operations are implemented in compiled C code operating on whole vectors at once, avoiding that per-element interpreter cost entirely
- **Symptoms**:
  - A script that runs acceptably on a small test dataset becomes impractically slow on the full production dataset
  - Profiling shows time dominated by loop overhead rather than the actual computation
- **Solution**: Rewrite explicit loops as vectorized expressions, `apply`-family calls, or `dplyr` verbs wherever the logic allows it; reserve explicit loops for genuinely sequential/stateful logic that can't be vectorized

### SE-4: Silent `NA` Propagation Through Calculations
- **Severity**: medium
- **Situation**: A single missing value (`NA`) in a dataset propagates silently through arithmetic or aggregate calculations, and the result — also `NA` or subtly wrong — isn't flagged as an error, so a downstream report or model silently reflects missing/incorrect analysis
- **Cause**: Most base R arithmetic and aggregate functions (`sum`, `mean`, etc.) propagate `NA` by default rather than raising an error, on the philosophy that missingness should be handled explicitly by the analyst, not hidden
- **Symptoms**:
  - A calculation that should have produced a number instead silently produces `NA`, and it's discovered only downstream (e.g., a chart with a missing bar)
- **Solution**: Explicitly decide and document how `NA` should be handled at each step (`na.rm = TRUE` where appropriate, or explicit imputation/filtering), and audit for unexpected `NA` counts as a standard data-quality check rather than assuming a clean dataset

### SE-5: `<<-` Causing Unintended Global State Mutation
- **Severity**: medium
- **Situation**: A function using `<<-` to "simplify" updating a variable ends up mutating global (or enclosing-scope) state that other, unrelated parts of the script also read, causing bugs that depend on call order and are hard to trace back to their source
- **Cause**: `<<-` searches up the chain of enclosing environments and assigns to the first existing binding it finds (or creates one globally if none exists) — this creates an implicit dependency between the function and whatever else touches that same variable, invisible from the function's signature
- **Symptoms**:
  - A variable's value changes unexpectedly after calling a function that doesn't appear (from its signature) to have side effects
  - Bugs that only appear depending on the order functions are called in
- **Solution**: Pass and return values explicitly through function arguments and return values rather than mutating enclosing scope; reserve `<<-` for narrow, well-understood patterns like closures implementing memoization where the side effect is the entire point and contained

## Recommended Tools

| Category | Tools |
|------|------|
| Data manipulation | dplyr, data.table, tidyr |
| Visualization | ggplot2, plotly |
| Reproducibility | renv, targets (pipeline management) |
| Testing/quality | testthat, lintr |

## Performance

**Full checklist**: → [extended/checklists.md#performance-checklist]

## Related Resources

[R Documentation](https://www.r-project.org/other-docs.html) | [R for Data Science](https://r4ds.hadley.nz/) | [Advanced R (Hadley Wickham)](https://adv-r.hadley.nz/)

## Related Domains

[[python]] | [[snowflake]] | [[clickhouse]]
