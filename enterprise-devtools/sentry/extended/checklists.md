# Sentry — Extended Checklists

## Error Hygiene Checklist

- [ ] Source maps uploaded as a required CI/CD step for every frontend deploy, tied to the correct release identifier
- [ ] Fingerprinting rules reviewed for high-volume or dynamic-message error types, not left at default grouping
- [ ] High-volume issues periodically reviewed for whether they're actually one coherent bug or several conflated ones
- [ ] Release tagging wired consistently into the deploy process across all environments
- [ ] Alert rules use thresholds/rate limiting or new-vs-regression distinctions, not one alert per raw occurrence
- [ ] Data scrubbing (`beforeSend` hooks or built-in PII rules) configured before sensitive fields reach Sentry
- [ ] SDK default data capture settings reviewed at setup time, not only after a compliance concern is raised
- [ ] Sample rates for error and performance monitoring set deliberately and revisited as traffic grows

## Alerting & Ownership Checklist

- [ ] Issue ownership/routing rules configured so the right team is notified for the right error type
- [ ] Critical error types (payment, auth, data-loss-adjacent) have always-capture rules regardless of sample rate
- [ ] Alert channel noise reviewed periodically against whether the team is still reading/acting on it
- [ ] Regression detection (error reappearing after being marked resolved) monitored, not just first-occurrence alerts
- [ ] Release health (crash-free session/user rate) tracked as a standard deploy-quality signal
