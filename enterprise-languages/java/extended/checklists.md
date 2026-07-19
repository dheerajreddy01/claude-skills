# Java — Extended Checklists

## JPA/Hibernate Performance Checklist

Before shipping an endpoint that touches JPA entities:

- [ ] Any loop that accesses a lazy-loaded association is replaced with `JOIN FETCH`, `@EntityGraph`, or a DTO projection
- [ ] SQL logging (`show-sql` or a query counter in tests) checked during development to catch N+1 patterns early
- [ ] Fetch type (`EAGER`/`LAZY`) reviewed per association — `EAGER` on collections is rarely correct
- [ ] Pagination used for any endpoint returning a potentially large collection
- [ ] Read-only queries marked `@Transactional(readOnly = true)` to avoid unnecessary dirty-checking overhead
- [ ] Batch inserts/updates use `hibernate.jdbc.batch_size` rather than one statement per row

## Production Readiness Checklist

- [ ] Heap size (`-Xmx`/`-Xms`) and GC algorithm set explicitly for production, not left to defaults
- [ ] Constructor injection used throughout; no field `@Autowired` in new code
- [ ] Global exception handling (`@ControllerAdvice` or equivalent) in place — no empty catch blocks
- [ ] `ThreadLocal` usage cleared in a `finally` block, especially in thread-pooled contexts
- [ ] Connection pool (HikariCP or equivalent) sized and timeout-configured explicitly, not left at defaults
- [ ] JVM Flight Recorder or equivalent profiling enabled for production incident diagnosis
- [ ] Structured logging (SLF4J + MDC) with request/trace IDs
- [ ] Health/readiness/liveness endpoints exposed (Spring Boot Actuator or equivalent)
- [ ] Dependency versions audited for known CVEs (OWASP Dependency-Check or Snyk) in CI
- [ ] Graceful shutdown configured (drain in-flight requests before the JVM exits)
