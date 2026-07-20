# Claude Skills — Enterprise Edition

> A governed library of domain-expertise packs that turns Claude into a specialist across business, finance, creative, professional services, lifestyle, methodology, programming languages, cloud platforms, and developer tooling.

[![Skills](https://img.shields.io/badge/skills-110-blue)](./README.md)
[![Domains](https://img.shields.io/badge/domains-9-green)](./README.md)
[![Version](https://img.shields.io/badge/version-2.0.0-informational)](./SKILL.md)
[![Plugin](https://img.shields.io/badge/Claude_Code-Plugin-orange)](https://code.claude.com/docs/en/discover-plugins)
[![License](https://img.shields.io/badge/license-MIT-lightgrey)](./LICENSE)

```
┌─────────────────────────────────────────────────────────────────┐
│                  Claude Skills — Enterprise Edition              │
│                                                                 │
│   ┌──────────────┐  ┌──────────────┐  ┌──────────────┐         │
│   │   Business   │  │   Finance    │  │   Creative   │         │
│   └──────────────┘  └──────────────┘  └──────────────┘         │
│                                                                 │
│   ┌──────────────┐  ┌──────────────┐  ┌──────────────┐         │
│   │ Professional │  │  Lifestyle   │  │  Methodology │         │
│   └──────────────┘  └──────────────┘  └──────────────┘         │
│                                                                 │
│   ┌──────────────┐  ┌──────────────┐  ┌──────────────┐         │
│   │  Languages   │  │    Cloud     │  │   DevTools   │         │
│   └──────────────┘  └──────────────┘  └──────────────┘         │
│                                                                 │
│         Claude auto-detects the task → loads the matching      │
│                    domain knowledge pack                        │
└─────────────────────────────────────────────────────────────────┘
```

## Table of Contents

- [Overview](#overview)
- [Why This Exists](#why-this-exists)
- [Domain Catalog](#domain-catalog)
- [Installation](#installation)
- [Automatic Domain Detection](#automatic-domain-detection)
- [Skill Architecture](#skill-architecture)
- [Creating a New Domain Skill](#creating-a-new-domain-skill)
- [Directory Structure](#directory-structure)
- [Contributing](#contributing)
- [Governance & Quality Bar](#governance--quality-bar)
- [License](#license)

## Overview

This repository is a structured library of **110 domain-expertise skills**, organized into **9 categories** spanning both non-technical domains (business, finance, creative, professional, lifestyle, methodology) and technical domains (programming languages, cloud platforms, developer tooling). Each skill packages the frameworks, formulas, checklists, and pitfalls a specialist in that field would bring to a task — so that Claude can apply them on demand instead of answering from generic knowledge alone.

Every skill is designed to:
- Be **auto-loaded** based on task keywords (via the `/evolve` skill or the `triggers` mechanism)
- Provide **verified frameworks and methodologies**, not just prose — every formula and benchmark in this library has been checked for correctness
- Stay **self-contained**, so a skill can be installed individually without pulling in the rest of the library
- Follow a **consistent schema** (`SKILL.md` + optional `extended/` reference material) so new domains are easy to author and review

## Why This Exists

Generic LLM responses to domain questions ("how do I value this stock," "how do I structure this sprint," "is this game balance fair") tend to be shallow and inconsistent from one conversation to the next. This library exists to give Claude a **repeatable, reviewed baseline** for each domain — the same DCF formula, the same RFM segmentation logic, the same WCAG contrast reference — every time, rather than reinventing it per conversation.

## Domain Catalog

| Category | Sub-domains | Description |
|------|--------|------|
| **enterprise-business/** | marketing, sales, product-management, project-management, strategy | Business operations |
| **enterprise-finance/** | quant-trading, investment-analysis, strategy-optimization | Finance expertise |
| **enterprise-creative/** | game-design, game-planner, galgame-master, deckbuilder-roguelike, ui-ux-design, brainstorming, storytelling, visual-media | Creative work |
| **enterprise-professional/** | research-analysis, knowledge-management | Professional services |
| **enterprise-lifestyle/** | personal-growth, side-income | Lifestyle |
| **enterprise-methodology/** | knowledge-acquisition-4c, tech-spec-gen, skill-optimizer, consistency-checker, director-pipeline | Development methodology |
| **enterprise-languages/** | python, javascript, go, rust, java, csharp, cpp, php, ruby, swift, kotlin, scala, r, elixir, dart-flutter, bash, haskell, lua, zig | Programming languages |
| **enterprise-cloud/** | aws, azure, gcp, cloudflare, vercel-netlify, digitalocean, oracle-cloud, firebase, supabase, heroku-flyio-render | Cloud platforms |
| **enterprise-devtools/** | jira, splunk, new-relic, datadog, grafana-prometheus, pagerduty, docker, kubernetes, git, github-actions, terraform, postgresql, redis, mongodb, postman, mysql, rabbitmq, elasticsearch, kafka, jenkins, gitlab, circleci, argocd-flux, ansible, pulumi, cloudformation, playwright-cypress, k6-jmeter, vault, okta-auth0, sonarqube, snyk, confluence, slack, linear, notion, sentry, honeycomb, sqlite, cassandra, dynamodb, clickhouse, snowflake, nats, openapi-graphql, helm, openshift, nginx, istio-linkerd, airflow, dbt, spark, opentelemetry, vector-databases, launchdarkly, chaos-engineering | Developer & ops tooling |

### Full Skill Index

| Domain | Example Triggers | Description |
|------|------------|------|
| `enterprise-finance/quant-trading` | quant, backtest, strategy | Quantitative trading strategy development |
| `enterprise-finance/investment-analysis` | financial statements, investment, valuation | Investment analysis and valuation |
| `enterprise-finance/strategy-optimization` | strategy optimization, improve win rate, parameter tuning | Trading strategy optimization methodology |
| `enterprise-business/product-management` | PRD, OKR, roadmap | Product management |
| `enterprise-business/project-management` | Scrum, sprint, Gantt chart | Project management |
| `enterprise-business/marketing` | marketing, CAC, funnel | Marketing strategy |
| `enterprise-business/sales` | sales, e-commerce, CRM | Sales and e-commerce operations |
| `enterprise-business/strategy` | strategy, blue ocean, differentiation | Business strategy and competitive advantage |
| `enterprise-creative/game-design` | game, level, balance | Game design |
| `enterprise-creative/game-planner` | GDD, game planning, design pillars | Game design documents |
| `enterprise-creative/galgame-master` | galgame, bishoujo game, visual novel, romance route | Galgame creation |
| `enterprise-creative/deckbuilder-roguelike` | deckbuilding, deckbuilder, roguelike | Card roguelike design |
| `enterprise-creative/ui-ux-design` | UI, UX, accessibility | Interface and experience design |
| `enterprise-creative/brainstorming` | inspiration, brainstorming, creativity | Creative ideation methodology |
| `enterprise-creative/storytelling` | novel, comic, screenplay, character | Story creation and narrative |
| `enterprise-creative/visual-media` | photography, video, animation, film | Visual media production |
| `enterprise-professional/research-analysis` | research, competitive analysis, market research | Research analysis methodology |
| `enterprise-professional/knowledge-management` | knowledge management, notes, PKM | Personal knowledge systems |
| `enterprise-lifestyle/personal-growth` | life planning, personal brand, time management | Personal growth and career development |
| `enterprise-lifestyle/side-income` | side hustle, passive income, investing | Side income and financial freedom |
| `enterprise-methodology/knowledge-acquisition-4c` | learning, research, entering a new field | Systematic learning methodology (4C) |
| `enterprise-methodology/tech-spec-gen` | tech-spec, technical spec, design-to-spec, PRD | Design document → technical spec conversion |
| `enterprise-methodology/skill-optimizer` | skill optimization, reduce tokens, skill trimming | Skill optimization and token efficiency |
| `enterprise-methodology/consistency-checker` | check, consistency, verify, sync | Content consistency checker |
| `enterprise-methodology/director-pipeline` | director pipeline, director dev tester, staged handoff, role pipeline | Director→Dev→Tester staged handoff workflow with acceptance-criteria-driven feedback loop |
| `enterprise-languages/python` | python, django, fastapi, pandas | Python language and production practices |
| `enterprise-languages/javascript` | javascript, typescript, node, react | JavaScript/TypeScript and Node.js production practices |
| `enterprise-languages/go` | golang, go, goroutine, channel | Go language and concurrency practices |
| `enterprise-languages/rust` | rust, cargo, ownership, borrow checker | Rust ownership model and systems practices |
| `enterprise-languages/java` | java, jvm, spring, maven | Java, JVM, and Spring ecosystem practices |
| `enterprise-cloud/aws` | aws, ec2, s3, lambda, iam | AWS architecture and security practices |
| `enterprise-cloud/azure` | azure, azure functions, app service, entra id | Azure architecture and security practices |
| `enterprise-cloud/gcp` | gcp, google cloud, cloud run, bigquery | Google Cloud architecture and security practices |
| `enterprise-devtools/jira` | jira, sprint, backlog, JQL | Jira issue tracking and workflow administration |
| `enterprise-devtools/splunk` | splunk, SPL, index, log search | Splunk architecture and search optimization |
| `enterprise-devtools/new-relic` | new relic, NRQL, APM, distributed tracing | New Relic APM and alerting practices |
| `enterprise-devtools/datadog` | datadog, monitor, unified service tagging | Datadog tagging strategy and cost management |
| `enterprise-devtools/grafana-prometheus` | grafana, prometheus, promql, alertmanager | Prometheus/Grafana metrics and dashboard practices |
| `enterprise-devtools/pagerduty` | pagerduty, on-call, escalation policy | PagerDuty on-call and incident response practices |
| `enterprise-devtools/docker` | docker, dockerfile, container, compose | Docker image builds and container security practices |
| `enterprise-devtools/kubernetes` | kubernetes, k8s, pod, deployment, helm | Kubernetes workload design and cluster reliability practices |
| `enterprise-devtools/git` | git, merge, rebase, branch | Git branching models and history hygiene |
| `enterprise-devtools/github-actions` | github actions, CI/CD, workflow, pipeline | GitHub Actions workflow design and security practices |
| `enterprise-devtools/terraform` | terraform, IaC, hcl, terraform plan | Terraform state management and safe apply practices |
| `enterprise-devtools/postgresql` | postgresql, postgres, SQL, EXPLAIN | PostgreSQL indexing and query performance practices |
| `enterprise-devtools/redis` | redis, cache, key-value, TTL | Redis data structures and cache reliability practices |
| `enterprise-devtools/mongodb` | mongodb, document database, aggregation pipeline | MongoDB schema design and aggregation practices |
| `enterprise-devtools/postman` | postman, API testing, collection, newman | Postman collection design and CI-driven API testing |
| `enterprise-devtools/mysql` | mysql, mariadb, innodb, EXPLAIN | MySQL/InnoDB indexing and replication practices |
| `enterprise-devtools/rabbitmq` | rabbitmq, message queue, AMQP, exchange | RabbitMQ topology and delivery guarantee practices |
| `enterprise-devtools/elasticsearch` | elasticsearch, opensearch, mapping, shard | Elasticsearch mapping design and query DSL practices |
| `enterprise-devtools/kafka` | kafka, topic, partition, consumer group | Kafka partition design and consumer reliability practices |
| `enterprise-languages/csharp` | csharp, c#, dotnet, .net | C#/.NET async patterns, DI lifetimes, and production practices |
| `enterprise-languages/cpp` | c++, cpp, RAII, template | C++ RAII, ownership, undefined behavior, and build practices |
| `enterprise-languages/php` | php, laravel, symfony, composer | PHP request lifecycle, type juggling, and Laravel/Symfony production practices |
| `enterprise-languages/ruby` | ruby, rails, ruby on rails, gem | Ruby/Rails ActiveRecord, metaprogramming, and production practices |
| `enterprise-languages/swift` | swift, ios, swiftui, xcode | Swift ARC, optionals, value types, and SwiftUI production practices |
| `enterprise-languages/kotlin` | kotlin, android, jetpack compose, coroutine | Kotlin null safety, coroutines, and Android/JVM production practices |
| `enterprise-languages/scala` | scala, akka, spark, sbt | Scala functional/OOP hybrid, implicit resolution, and JVM concurrency practices |
| `enterprise-languages/r` | r language, rstudio, tidyverse, ggplot2 | R vectorized computing, tidyverse workflows, and statistical data practices |
| `enterprise-languages/elixir` | elixir, phoenix, erlang, BEAM | Elixir/BEAM concurrency, OTP supervision, and Phoenix production practices |
| `enterprise-cloud/cloudflare` | cloudflare, cloudflare workers, R2, CDN | Cloudflare Workers, R2, DNS/CDN/WAF, and edge platform practices |
| `enterprise-cloud/vercel-netlify` | vercel, netlify, jamstack, edge function | Vercel/Netlify serverless functions, preview deployments, and JAMstack build practices |
| `enterprise-cloud/digitalocean` | digitalocean, droplet, DO, app platform | DigitalOcean Droplets, App Platform, and simplified cloud infrastructure practices |
| `enterprise-cloud/oracle-cloud` | oracle cloud, OCI, autonomous database, compartment | Oracle Cloud Infrastructure compartments, IAM policy language, and Autonomous Database practices |
| `enterprise-devtools/jenkins` | jenkins, jenkinsfile, pipeline, groovy | Jenkins pipeline design, plugin management, and credential security practices |
| `enterprise-devtools/gitlab` | gitlab, gitlab ci, .gitlab-ci.yml, merge request | GitLab CI/CD pipeline design, runner architecture, and integrated platform practices |
| `enterprise-devtools/circleci` | circleci, circle ci, orb, workflow | CircleCI workflow design, orb usage, and caching/parallelism practices |
| `enterprise-devtools/argocd-flux` | argocd, flux, gitops, continuous delivery | GitOps continuous delivery practices with ArgoCD and Flux |
| `enterprise-devtools/ansible` | ansible, playbook, ansible-playbook, inventory | Ansible playbook design, idempotency discipline, and secrets management practices |
| `enterprise-devtools/pulumi` | pulumi, infrastructure as code, pulumi stack | Pulumi infrastructure-as-code with general-purpose languages, state, and stack reference practices |
| `enterprise-devtools/cloudformation` | cloudformation, cfn, cloudformation stack, cloudformation template | AWS CloudFormation template design, stack update safety, and drift management practices |
| `enterprise-devtools/playwright-cypress` | playwright, cypress, e2e testing, end-to-end test | Playwright and Cypress end-to-end testing: automation model, flakiness, and CI reliability |
| `enterprise-devtools/k6-jmeter` | k6, jmeter, load testing, performance testing | k6 and JMeter load testing: realistic traffic modeling and result interpretation |
| `enterprise-devtools/vault` | vault, hashicorp vault, secrets management, dynamic secrets | HashiCorp Vault secrets engines, dynamic secrets, and unseal/HA practices |
| `enterprise-devtools/okta-auth0` | okta, auth0, SSO, identity provider | Okta and Auth0 identity/SSO: OIDC/SAML, token handling, and MFA policy practices |
| `enterprise-devtools/sonarqube` | sonarqube, static analysis, code quality, quality gate | SonarQube static analysis, quality gates, and code health practices |
| `enterprise-devtools/snyk` | snyk, dependency vulnerability, SCA, container scanning | Snyk dependency, container, and license vulnerability scanning practices |
| `enterprise-devtools/confluence` | confluence, wiki, documentation space, page tree | Confluence space design, page permissions, and documentation hygiene practices |
| `enterprise-devtools/slack` | slack, slack bot, slack app, webhook | Slack workspace structure, app/bot permission scopes, and integration reliability practices |
| `enterprise-devtools/linear` | linear, issue tracker, cycle, triage | Linear cycles, triage workflow, and opinionated issue-tracking practices |
| `enterprise-devtools/notion` | notion, notion database, workspace, block | Notion workspace architecture, database design, and permission inheritance practices |
| `enterprise-devtools/sentry` | sentry, error tracking, exception monitoring, source map | Sentry error grouping, source maps, and alerting/PII hygiene practices |
| `enterprise-devtools/honeycomb` | honeycomb, observability, trace, high cardinality | Honeycomb high-cardinality observability, instrumentation, and sampling practices |
| `enterprise-devtools/sqlite` | sqlite, embedded database, single-file database | SQLite embedded database concurrency, WAL mode, and appropriate-use practices |
| `enterprise-devtools/cassandra` | cassandra, wide column, ring, gossip protocol | Apache Cassandra query-first data modeling, consistency levels, and ring architecture practices |
| `enterprise-devtools/dynamodb` | dynamodb, single table design, partition key, GSI | DynamoDB single-table design, partition key strategy, and capacity/cost practices |
| `enterprise-devtools/clickhouse` | clickhouse, OLAP, columnar database, MergeTree | ClickHouse columnar OLAP design, MergeTree engine practices, and query performance |
| `enterprise-devtools/snowflake` | snowflake, data warehouse, virtual warehouse, snowpipe | Snowflake virtual warehouse cost management, micro-partitioning, and data recovery practices |
| `enterprise-devtools/nats` | nats, jetstream, pub sub, subject | NATS and JetStream pub/sub design, delivery guarantees, and consumer reliability practices |
| `enterprise-devtools/openapi-graphql` | openapi, swagger, graphql, api design | OpenAPI/Swagger REST contract design and GraphQL schema/resolver practices |
| `enterprise-devtools/helm` | helm, helm chart, kubernetes package manager, values.yaml | Helm chart structure, templating, and release lifecycle practices |
| `enterprise-devtools/openshift` | openshift, okd, red hat kubernetes, security context constraint | OpenShift/OKD security context constraints, Routes, and Operator-based platform practices |
| `enterprise-languages/dart-flutter` | dart, flutter, widget, statefulwidget | Dart/Flutter widget lifecycle, state management, and async production practices |
| `enterprise-languages/bash` | bash, shell script, shell scripting, sh | Bash/shell scripting quoting discipline, error handling, and portability practices |
| `enterprise-languages/haskell` | haskell, cabal, stack, ghc | Haskell laziness, type system, and functional programming production practices |
| `enterprise-languages/lua` | lua, luajit, metatable, coroutine | Lua tables, metatables, coroutines, and embedded-scripting production practices |
| `enterprise-languages/zig` | zig, comptime, allocator, error union | Zig manual memory management, comptime, and error-union production practices |
| `enterprise-cloud/firebase` | firebase, firestore, cloud functions for firebase, firebase auth | Firebase Firestore, security rules, and Cloud Functions platform expertise |
| `enterprise-cloud/supabase` | supabase, row level security, RLS, supabase auth | Supabase Postgres, Row Level Security, and realtime/auto-API platform expertise |
| `enterprise-cloud/heroku-flyio-render` | heroku, fly.io, flyio, render | Heroku, Fly.io, and Render simple-PaaS deployment practices |
| `enterprise-devtools/nginx` | nginx, reverse proxy, load balancer, location block | Nginx reverse proxy, load balancing, and web server configuration practices |
| `enterprise-devtools/istio-linkerd` | istio, linkerd, service mesh, sidecar | Istio and Linkerd service mesh traffic management, mTLS, and operational practices |
| `enterprise-devtools/airflow` | airflow, DAG, apache airflow, task idempotency | Apache Airflow DAG design, task idempotency, and scheduler operational practices |
| `enterprise-devtools/dbt` | dbt, data build tool, dbt model, dbt run | dbt (data build tool) model design, DAG dependencies, and incremental transformation practices |
| `enterprise-devtools/spark` | spark, apache spark, pyspark, spark job | Apache Spark partitioning, shuffle performance, and distributed job reliability practices |
| `enterprise-devtools/opentelemetry` | opentelemetry, otel, otel collector, instrumentation | OpenTelemetry instrumentation, context propagation, and Collector pipeline practices |
| `enterprise-devtools/vector-databases` | vector database, pinecone, weaviate, pgvector | Vector database (Pinecone/Weaviate/pgvector) embedding search, indexing, and RAG chunking practices |
| `enterprise-devtools/launchdarkly` | launchdarkly, feature flag, feature toggle, unleash | LaunchDarkly feature flag rollout strategy, targeting, and flag lifecycle hygiene practices |
| `enterprise-devtools/chaos-engineering` | chaos engineering, gremlin, chaos mesh, fault injection | Chaos engineering experiment design, blast radius control, and resilience validation practices |

## Installation

### Option A: Plugin Marketplace (recommended)

```bash
# 1. Add the marketplace (GitHub format: owner/repo)
/plugin marketplace add dheerajreddy01/claude-skills

# 2. Open the plugin management UI and browse available plugins on the Discover tab
/plugin

# 3. Install specific skills (pick what you need)
/plugin install marketing@claude-skills
/plugin install quant-trading@claude-skills
/plugin install game-design@claude-skills
/plugin install python@claude-skills
/plugin install aws@claude-skills
/plugin install datadog@claude-skills
/plugin install kubernetes@claude-skills
/plugin install terraform@claude-skills
/plugin install rabbitmq@claude-skills
/plugin install kafka@claude-skills
/plugin install cloudflare@claude-skills
/plugin install ansible@claude-skills
/plugin install dynamodb@claude-skills
/plugin install airflow@claude-skills
/plugin install vector-databases@claude-skills
/plugin install director-pipeline@claude-skills

# Or just mention a skill's subject in conversation — Claude will load it automatically
```

**Supported marketplace formats:**
```bash
# Short format (recommended)
/plugin marketplace add dheerajreddy01/claude-skills

# HTTPS URL
/plugin marketplace add https://github.com/dheerajreddy01/claude-skills.git

# Specific branch or tag
/plugin marketplace add dheerajreddy01/claude-skills#main
```

**Plugin commands:**
| Command | Description |
|------|------|
| `/plugin` | Open the interactive plugin management UI |
| `/plugin install <name>@<marketplace>` | Install a specific plugin |
| `/plugin disable <name>@<marketplace>` | Temporarily disable a plugin |
| `/plugin uninstall <name>@<marketplace>` | Completely remove a plugin |

### Option B: Clone into your Skills directory

```bash
git clone https://github.com/dheerajreddy01/claude-skills.git ~/.claude/skills/domain-skills
```

Claude will automatically discover and use the relevant skills as needed.

### Option C: claude-starter-kit

```bash
npx claude-starter-kit
# Interactively select the domains you need
```

## Automatic Domain Detection

When used together with the `/evolve` skill, Claude will automatically:

1. Analyze the task's keywords
2. Match them against each skill's `triggers`
3. Load the relevant domain knowledge before executing the task

```
User: /evolve analyze Apple's financial statements and assess investment value

Agent:
🔍 Auto Domain Detection
→ Keywords: financial statements, analysis, investment
→ Loaded: enterprise-finance/investment-analysis
→ Executing the task using the investment analysis framework
```

## Skill Architecture

Each skill is a self-contained package:

```
enterprise-finance/quant-trading/
├── .claude-plugin/
│   └── plugin.json            # Plugin manifest (name, author, version)
├── skills/quant-trading/
│   └── SKILL.md                # Core instructions, frontmatter, triggers
└── extended/                   # Optional deep-dive reference material
    ├── code-examples.md
    └── templates.md
```

### Triggers vs. Keywords

`SKILL.md` frontmatter follows the skillpkg-compatible schema:

```yaml
---
schema: "1.0"
name: quant-trading
triggers: [quant, trading, backtest, strategy, factor]
keywords: [finance, trading, quantitative]
---
```

| Field | Purpose | Example |
|------|------|------|
| `triggers` | **Task matching** — fires based on what the user says | `[financial statements, investment, stock]` |
| `keywords` | **Category search** — lookup by domain | `[finance, business]` |

**Writing good triggers:**
```yaml
triggers:
  # ✅ Include synonyms
  - quantitative
  - quant

  # ✅ Include common phrasing
  - backtest
  - strategy testing

  # ❌ Avoid overly generic terms
  # - analysis (too broad)
  # - report (too broad)
```

### Skill Enhancement Features

| Feature | Description | Purpose |
|------|------|------|
| **Sharp Edges** | Documented warnings about common domain pitfalls, with severity ratings | Proactively steers users away from known failure modes |
| **Validations** | Quality-check rules embedded in the skill | Checks output against domain standards before it's considered done |
| **Collaboration** | Cross-domain prerequisite/delegation rules | Enables smart handoff between skills (e.g. quant-trading → python) |

```yaml
# SKILL.md frontmatter example
---
schema: "1.0"
name: quant-trading
collaboration:
  prerequisites:
    - skill: python
      reason: Foundation for quant development
  delegation_triggers:
    - trigger: database design
      delegate_to: database
---

# Sharp Edges
- id: backtest-overfitting
  severity: critical
  situation: The strategy backtest shows suspiciously high returns
  solution: Use walk-forward validation and out-of-sample testing
```

## Creating a New Domain Skill

```bash
# 1. Create the directory
mkdir -p enterprise-finance/my-domain

# 2. Create SKILL.md
cat > enterprise-finance/my-domain/SKILL.md << 'EOF'
---
schema: "1.0"
name: my-domain
version: "1.0.0"
description: Domain description
triggers: [keyword1, keyword2]
keywords: [category1, category2]
---

# Domain Name

## Applicable Scenarios
- Scenario 1
- Scenario 2

## Core Knowledge
...

## Best Practices
...
EOF

# 3. Register it as an installable plugin
# Add the following entry to the "plugins" array in .claude-plugin/marketplace.json:
{
  "name": "my-domain",
  "description": "Domain description",
  "source": "./enterprise-finance/my-domain",
  "version": "1.0.0",
  "strict": true
}

# 4. Verify
# Mention the trigger keyword in a Claude Code conversation — it should load automatically
```

Before submitting a new skill, run it against the [Governance & Quality Bar](#governance--quality-bar) checklist below.

## Directory Structure

```
claude-skills/
├── .claude-plugin/              # Plugin marketplace configuration
│   └── marketplace.json         # Lists all 110 skills as standalone plugins
├── enterprise-business/         # Business operations (5 skills)
├── enterprise-finance/          # Finance (3 skills)
├── enterprise-creative/         # Creative work (8 skills)
├── enterprise-professional/     # Professional services (2 skills)
├── enterprise-lifestyle/        # Lifestyle (2 skills)
├── enterprise-methodology/      # Methodology (5 skills)
├── enterprise-languages/        # Programming languages (19 skills)
├── enterprise-cloud/            # Cloud platforms (10 skills)
├── enterprise-devtools/         # Developer & ops tooling (56 skills)
├── interfaces/                  # Cross-domain → technical handoff definitions
├── docs/                        # Quickstart and supporting documentation
├── SKILL-TEMPLATE.md            # Skill creation template
├── SKILL.md                     # Library-level manifest and usage guide
├── CONTRIBUTING.md              # Contributing guide
├── OPTIMIZATION-REPORT.md       # Token-efficiency audit of existing skills
├── README.md
└── LICENSE
```

## Contributing

Contributions of new domains, corrections, or translations are welcome — see [CONTRIBUTING.md](./CONTRIBUTING.md) for the skill schema, style conventions, and review checklist.

## Governance & Quality Bar

Every skill in this library is expected to meet the same bar before merge:

- [ ] `SKILL.md` frontmatter is complete (`schema`, `name`, `version`, `triggers`, `keywords`)
- [ ] Every formula and numeric benchmark is verified against a recognized standard or textbook definition
- [ ] Tables and checklists are internally consistent (no contradicting rows across sections)
- [ ] Triggers include realistic phrasing a user would actually type, not just the domain's formal name
- [ ] No fabricated statistics, invented frameworks, or unverifiable citations
- [ ] `extended/` reference material (if present) is linked correctly and doesn't duplicate the core `SKILL.md`

## License

Released under the [MIT License](./LICENSE).