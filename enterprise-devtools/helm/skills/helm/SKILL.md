---
schema: "1.0"
name: helm
version: "1.0.0"
description: Helm chart structure, Go templating, values precedence, and release lifecycle practices
domain: technology
triggers:
  keywords:
    primary: [helm, helm chart, kubernetes package manager, values.yaml]
    secondary: [chart repository, subchart, helm upgrade, tiller, Chart.yaml]
  context_boost: [kubernetes deployment, package templating]
  context_penalty: [kustomize, raw manifests]
  priority: high
dependencies:
  software-skills: []
author: claude-domain-skills
---

# Helm

> Templates that render valid YAML for the values you tested, and quietly don't for the ones you didn't

## Applicable Scenarios

- Structuring a Helm chart (templates, values, dependencies) for a Kubernetes application
- Managing values overrides across multiple environments
- Diagnosing a failed `helm upgrade` or unexpected rendered output
- Handling chart dependencies (subcharts) and shared values
- Planning release rollback strategy

## Core Knowledge

### Chart Structure

```
mychart/
├── Chart.yaml          # Metadata: name, version, appVersion, dependencies
├── values.yaml          # Default configuration values
├── templates/           # Go-templated Kubernetes manifests
│   ├── deployment.yaml
│   └── _helpers.tpl     # Reusable template snippets
└── charts/              # Vendored subchart dependencies
```

- **`Chart.yaml`**'s `version` is the chart's own version; `appVersion` is the version of the application it deploys — these are independent and often confused
- **Subcharts** let a chart depend on other charts (e.g., a web app chart depending on a Redis chart) — subchart values are namespaced under the subchart's name in the parent's `values.yaml`

### Templating

- Helm templates are Go templates rendered against the merged values, producing final Kubernetes YAML — templating happens entirely client-side (or server-side via `helm template`) before anything is sent to the cluster
- Conditional/loop constructs (`{{ if }}`, `{{ range }}`) generate different YAML depending on values — a combination of values nobody tested can render syntactically invalid or semantically wrong YAML that only fails at actual deploy time
- `helm template` (renders without installing) and `helm lint` are the primary ways to catch template bugs before they hit a real cluster

### Values Precedence

Values merge from multiple sources, later sources overriding earlier ones:

```
chart's values.yaml → -f file1.yaml → -f file2.yaml → --set key=value
```

- Multiple `-f` flags apply in the order given, each overriding overlapping keys from the previous
- `--set` always wins over any `-f` file, regardless of order — a `--set` in a CI script can silently override a carefully maintained environment values file

### Release Lifecycle

| Command | Effect |
|------|------|
| `helm install` | First deployment of a release |
| `helm upgrade` | Applies a new chart version/values to an existing release |
| `helm rollback` | Reverts to a previous release revision |
| `helm history` | Lists all revisions of a release |

Helm tracks release history as its own state (stored as Secrets in-cluster by default) — this is what `helm rollback` reverts to, and it's a record of *values and chart version*, not a guarantee that the underlying resources can always be cleanly reverted (see Sharp Edges).

## Best Practices

1. **Run `helm lint` and `helm template` in CI** for every chart change, across the actual values files used per environment — not just the defaults
2. **Keep environment-specific values in separate, version-controlled files** (`values-prod.yaml`, `values-staging.yaml`), layered onto shared defaults
3. **Avoid `--set` for anything beyond quick manual testing** — it's invisible in version control and silently overrides file-based values
4. **Set explicit resource requests/limits in chart defaults**, not just documentation — a chart's defaults are what most installs will actually run with
5. **Pin subchart versions explicitly** in `Chart.yaml`'s `dependencies`, don't float on ranges for anything beyond local development
6. **Understand that Helm doesn't manage CRD lifecycle well** — treat CRDs installed by a chart as a separate, deliberate operational concern, not something `helm upgrade` will safely evolve

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| Testing a chart only against its default `values.yaml` | Run `helm template`/`lint` against every real environment's values combination in CI |
| Using `--set` for production deploys | Use version-controlled values files layered via `-f`; reserve `--set` for local debugging |
| Assuming `Chart.yaml`'s `version` and `appVersion` are the same thing | Track them independently — `version` is the chart, `appVersion` is the app it deploys |
| Expecting `helm upgrade` to handle any field change on any resource | Understand which fields are immutable on the underlying resource type and plan around forced replacements |
| Relying on `helm upgrade` to evolve CRDs installed by the chart | Manage CRD installation/upgrade as an explicit, separate step |

## Sharp Edges

### SE-1: `helm upgrade` Failing on Immutable Field Changes
- **Severity**: high
- **Situation**: A seemingly minor values change (e.g., a Job's `selector`, a PVC's storage class) causes `helm upgrade` to fail with a cryptic Kubernetes API error about an immutable field, and the release gets stuck in a failed/pending state
- **Cause**: Some Kubernetes resource fields are immutable after creation at the API level — Helm doesn't know this ahead of time and simply submits the changed manifest, and the Kubernetes API server is what actually rejects it
- **Symptoms**:
  - `helm upgrade` fails with `field is immutable` or similar, often deep in a nested resource type
  - The release history shows a failed revision that needs manual intervention to resolve
- **Solution**: For fields known to be immutable on a given resource type, either avoid changing them via chart values in-place, or explicitly delete-and-recreate the specific resource as part of the upgrade process (some charts template this via `helm.sh/hook: pre-upgrade` delete jobs)
- **Details**: → [extended/checklists.md#chart-safety-checklist]

### SE-2: Values Override Precedence Confusion
- **Severity**: high
- **Situation**: A deploy uses multiple `-f` values files plus a `--set` flag, and the final rendered configuration doesn't match what any single file specifies — a value from an unexpected source silently won
- **Cause**: Helm's merge order (chart defaults → `-f` files in given order → `--set`) means a later source always wins on overlapping keys, and it's easy to lose track of which source actually determined a given final value across several layered files and flags
- **Symptoms**:
  - A configuration change made in what looks like "the right file" doesn't take effect
  - `helm get values` shows a different effective value than expected from reading the values files alone
- **Solution**: Use `helm template`/`helm get values -a` to inspect the actual merged/rendered output before trusting what any individual source file implies, and minimize the number of values sources layered together for any single deploy

### SE-3: Template Logic Producing Invalid YAML Only for Certain Values
- **Severity**: medium
- **Situation**: A chart's `{{ if }}`/`{{ range }}` template logic renders valid YAML for the values combinations that were tested, but a different, untested combination (e.g., an empty list, a missing optional key) produces malformed or semantically wrong YAML that only fails at actual `helm install`/`upgrade` time
- **Cause**: Go templates don't validate the *semantic* correctness of their output — a template can be syntactically valid Go template code while still producing broken YAML for edge-case input values
- **Symptoms**:
  - A chart that works fine in one environment fails to render or deploy correctly in another with different values
  - The failure is a YAML parse error or a Kubernetes API rejection that only appears with specific values combinations
- **Solution**: Run `helm template`/`lint` against every real environment's values file (not just defaults) in CI, and explicitly test edge cases like empty lists, zero replicas, and missing optional values

### SE-4: Chart Default Resource Settings Propagating an Anti-Pattern
- **Severity**: medium
- **Situation**: A chart ships with no (or unrealistic) default resource requests/limits, and every team that installs it without overriding those defaults ends up with unbounded or mis-sized workloads across the fleet
- **Cause**: Most consumers of a chart install it with mostly-default values, especially early on — whatever the chart's defaults are effectively becomes the fleet-wide baseline unless someone actively overrides them
- **Symptoms**:
  - Many unrelated deployments across the cluster share the same missing/unrealistic resource configuration, traceable back to one chart's defaults
- **Solution**: Set sane, explicit resource request/limit defaults in the chart itself (not just as commented-out examples in `values.yaml`), so the path of least resistance for consumers is already reasonably safe

### SE-5: CRD Lifecycle Not Managed by `helm upgrade`
- **Severity**: high
- **Situation**: A chart that installs Custom Resource Definitions (CRDs) ships a new chart version with updated CRDs, but `helm upgrade` doesn't apply the CRD changes — the cluster's CRDs silently stay on the old version, causing confusing behavior for anything relying on the new CRD schema
- **Cause**: Helm deliberately does not manage CRD upgrades or deletions through the normal template lifecycle (CRDs placed in a chart's special `crds/` directory are only installed on `helm install`, never updated or removed by `helm upgrade`/`uninstall`) — this is a documented but frequently-missed Helm design decision
- **Symptoms**:
  - A resource using a CRD's new field has no effect, because the cluster's installed CRD schema doesn't yet include that field
  - `kubectl get crd <name> -o yaml` shows an older schema version than what the chart's `crds/` directory currently contains
- **Solution**: Treat CRD installation/upgrade as an explicit, separate operational step (a dedicated `kubectl apply` step, or a separate chart/job specifically for CRD management) rather than assuming `helm upgrade` keeps them current

## Recommended Tools

| Category | Tools |
|------|------|
| Linting/rendering | `helm lint`, `helm template`, `helm-unittest` |
| Chart management | Helmfile, Chart Releaser |
| Repositories | ChartMuseum, OCI-based registries |
| Alternatives/complements | Kustomize (overlay-based, no templating language) |

## Chart Safety

**Full checklist**: → [extended/checklists.md#chart-safety-checklist]

## Related Resources

[Helm Docs](https://helm.sh/docs/) | [Helm Best Practices](https://helm.sh/docs/chart_best_practices/) | [Chart Template Guide](https://helm.sh/docs/chart_template_guide/)

## Related Domains

[[kubernetes]] | [[docker]] | [[argocd-flux]] | [[github-actions]]
