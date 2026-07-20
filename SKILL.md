---
schema: "1.0"
name: claude-skills
version: "2.0.0"
description: A collection of domain expertise covering business, finance, creative, professional services, lifestyle, methodology, programming languages, cloud platforms, and developer tooling - 86 domains in total
triggers: [domain, professional, knowledge, business, finance, creative, lifestyle, language, cloud, devtools]
keywords: [domain, skills, knowledge, business, finance, creative, professional, lifestyle, languages, cloud, devtools]
author: dheerajreddy01
---

# Claude Skills — Enterprise Edition

> A collection of skills that give AI professional knowledge across many domains

## Overview

This is a library of domain expertise — spanning both non-technical fields and technical domains like programming languages and cloud platforms — designed to:
- Work together with the `/evolve` skill to automatically identify the domain a task needs
- Load specific domain knowledge on demand via skillpkg
- Provide frameworks, methodologies, and best practices for each domain

## Available Domains (86)

### 💰 Finance
| Domain | Triggers | Description |
|------|--------|------|
| `enterprise-finance/quant-trading` | quant, backtest, strategy | Quantitative trading strategy development |
| `enterprise-finance/investment-analysis` | financial statements, investment, valuation, ROE | Investment analysis and valuation |
| `enterprise-finance/strategy-optimization` | strategy optimization, win rate, returns, backtest | Trading strategy optimization methodology |

### 💼 Business
| Domain | Triggers | Description |
|------|--------|------|
| `enterprise-business/marketing` | marketing, SEO, funnel, CAC | Digital marketing strategy |
| `enterprise-business/sales` | sales, e-commerce, CRM | Sales and e-commerce operations |
| `enterprise-business/product-management` | PRD, OKR, roadmap | Product management |
| `enterprise-business/project-management` | Scrum, Sprint, Gantt chart | Project management |
| `enterprise-business/strategy` | strategy, blue ocean, differentiation, business model | Business strategy |

### 🎨 Creative
| Domain | Triggers | Description |
|------|--------|------|
| `enterprise-creative/game-design` | game, level, balance, GDD | Game design |
| `enterprise-creative/ui-ux-design` | UI, UX, accessibility, WCAG | Interface and experience design |
| `enterprise-creative/brainstorming` | inspiration, brainstorming, creativity, SCAMPER | Creative ideation |
| `enterprise-creative/storytelling` | novel, screenplay, character, three-act structure | Story creation |
| `enterprise-creative/visual-media` | photography, video, animation, storyboard | Visual media production |

### 🔬 Professional
| Domain | Triggers | Description |
|------|--------|------|
| `enterprise-professional/research-analysis` | research, competitive analysis, market research, analysis | Research analysis |
| `enterprise-professional/knowledge-management` | notes, PKM, second brain, Zettelkasten | Knowledge management |

### 🌱 Lifestyle
| Domain | Triggers | Description |
|------|--------|------|
| `enterprise-lifestyle/personal-growth` | life planning, personal brand, time management | Personal growth |
| `enterprise-lifestyle/side-income` | side hustle, passive income, financial freedom | Side income and investing |

### 🧠 Methodology
| Domain | Triggers | Description |
|------|--------|------|
| `enterprise-methodology/knowledge-acquisition-4c` | learning, research, new domain, knowledge acquisition | Systematic learning from novice to expert |

### 🔧 Languages
| Domain | Triggers | Description |
|------|--------|------|
| `enterprise-languages/python` | python, django, fastapi, pandas | Python language and production practices |
| `enterprise-languages/javascript` | javascript, typescript, node, react | JavaScript/TypeScript and Node.js production practices |
| `enterprise-languages/go` | golang, go, goroutine, channel | Go language and concurrency practices |
| `enterprise-languages/rust` | rust, cargo, ownership, borrow checker | Rust ownership model and systems practices |
| `enterprise-languages/java` | java, jvm, spring, maven | Java, JVM, and Spring ecosystem practices |
| `enterprise-languages/csharp` | csharp, c#, dotnet, .net | C#/.NET async patterns, DI lifetimes, and production practices |
| `enterprise-languages/cpp` | c++, cpp, RAII, template | C++ RAII, ownership, undefined behavior, and build practices |
| `enterprise-languages/php` | php, laravel, symfony, composer | PHP request lifecycle, type juggling, and Laravel/Symfony production practices |
| `enterprise-languages/ruby` | ruby, rails, ruby on rails, gem | Ruby/Rails ActiveRecord, metaprogramming, and production practices |
| `enterprise-languages/swift` | swift, ios, swiftui, xcode | Swift ARC, optionals, value types, and SwiftUI production practices |
| `enterprise-languages/kotlin` | kotlin, android, jetpack compose, coroutine | Kotlin null safety, coroutines, and Android/JVM production practices |
| `enterprise-languages/scala` | scala, akka, spark, sbt | Scala functional/OOP hybrid, implicit resolution, and JVM concurrency practices |
| `enterprise-languages/r` | r language, rstudio, tidyverse, ggplot2 | R vectorized computing, tidyverse workflows, and statistical data practices |
| `enterprise-languages/elixir` | elixir, phoenix, erlang, BEAM | Elixir/BEAM concurrency, OTP supervision, and Phoenix production practices |

### ☁️ Cloud
| Domain | Triggers | Description |
|------|--------|------|
| `enterprise-cloud/aws` | aws, ec2, s3, lambda, iam | AWS architecture and security practices |
| `enterprise-cloud/azure` | azure, azure functions, app service, entra id | Azure architecture and security practices |
| `enterprise-cloud/gcp` | gcp, google cloud, cloud run, bigquery | Google Cloud architecture and security practices |
| `enterprise-cloud/cloudflare` | cloudflare, cloudflare workers, R2, CDN | Cloudflare Workers, R2, DNS/CDN/WAF, and edge platform practices |
| `enterprise-cloud/vercel-netlify` | vercel, netlify, jamstack, edge function | Vercel/Netlify serverless functions, preview deployments, and JAMstack build practices |
| `enterprise-cloud/digitalocean` | digitalocean, droplet, DO, app platform | DigitalOcean Droplets, App Platform, and simplified cloud infrastructure practices |
| `enterprise-cloud/oracle-cloud` | oracle cloud, OCI, autonomous database, compartment | Oracle Cloud Infrastructure compartments, IAM policy language, and Autonomous Database practices |

### 🛠️ DevTools
| Domain | Triggers | Description |
|------|--------|------|
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

## Usage

### Method 1: Automatic Detection (recommended)

When used together with the `/evolve` skill, the matching domain is automatically loaded based on the task's keywords:

```
User: /evolve analyze Apple's financial statements and assess investment value

Agent:
🔍 Auto Domain Detection
→ Keywords: financial statements, analysis, investment
→ Loaded: enterprise-finance/investment-analysis
→ Executing the task using the investment analysis framework
```

### Method 2: Manual Installation

```python
# Install a specific domain
mcp__skillpkg__install_skill({
    "source": "github:dheerajreddy01/claude-skills#enterprise-finance/quant-trading"
})

# Load it for use
mcp__skillpkg__load_skill({ "id": "quant-trading" })
```

### Method 3: Using claude-starter-kit

```bash
npx claude-starter-kit
# Interactively select the domains you need
```

## Domain Skill Structure

Each domain skill contains:

```markdown
## Applicable Scenarios
When to use this domain knowledge

## Core Framework
The domain's standard methodologies and mental models

## Best Practices
Proven approaches

## Common Mistakes
Guidance for avoiding common pitfalls

## Recommended Tools
Practical tools relevant to the domain

## Reference Resources
Resources for further learning
```

## Contributing

Contributions of new domains or translations are welcome! Please see [CONTRIBUTING.md](./CONTRIBUTING.md)

