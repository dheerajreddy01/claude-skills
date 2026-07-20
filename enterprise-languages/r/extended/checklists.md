# R — Extended Checklists

## Performance Checklist

- [ ] No `for` loop appending to a vector/list one element at a time where vectorization or preallocation would work
- [ ] Large data frame modifications reviewed for copy-on-modify overhead; `data.table` considered for genuinely large datasets with heavy in-place updates
- [ ] Vectorized operations (or `apply`-family/`dplyr` verbs) used in place of explicit element-wise loops wherever possible
- [ ] Profiling (`Rprof`/`profvis`) run on any script suspected of being slow, before guessing at a fix
- [ ] Memory usage (`object.size()`, `pryr::mem_used()`) checked for scripts working with large datasets

## Data Quality & Reproducibility Checklist

- [ ] Factor levels explicitly defined before assignment; character vectors used during manipulation, converted to factors only when needed
- [ ] `NA` handling decided explicitly per calculation (`na.rm`, imputation, or filtering), not left to default propagation
- [ ] Unexpected `NA` counts checked as a standard data-quality step, not discovered downstream
- [ ] `<<-` usage limited to narrow, well-understood patterns (e.g., closures); no casual global state mutation
- [ ] `renv::snapshot()` used to lock package versions for reproducible environments across machines/CI
- [ ] `testthat` tests cover key data transformation functions, not just final outputs
