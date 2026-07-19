# Professional → Tech Interface

> Mapping professional-skills domain requirements to technical implementation

## Domain Skills Covered

- `research-analysis` - Research analysis
- `knowledge-management` - Knowledge management

## Requirement → Technology Mapping

| Domain Requirement | Technical Implementation | Software Skills |
|-------------------|-------------------------|-----------------|
| Research reports | Markdown/LaTeX | `documentation` |
| Data analysis | Python/R | `data-analysis`, `python` |
| Reference management | Zotero/Mendeley | `documentation` |
| Knowledge base system | Obsidian/Notion | `documentation` |
| Visualized reports | Charts/Dashboards | `data-analysis`, `frontend` |
| Survey systems | Survey Tools | `automation-scripts` |

## Common Combination Patterns

### Pattern 1: Data-Driven Researcher

**Focus**: Quantitative research, data analysis, statistical validation

```yaml
domain_skills:
  - research-analysis (deep)

software_skills:
  - documentation (required)
  - data-analysis (required)
  - python (recommended)
```

**Use Case**: Market research, competitive analysis, academic research

### Pattern 2: Knowledge Worker

**Focus**: Knowledge accumulation, personal knowledge systems, output ability

```yaml
domain_skills:
  - knowledge-management (deep)
  - research-analysis (basic)

software_skills:
  - documentation (required)
```

**Use Case**: Building a second brain, knowledge extraction, content creation

### Pattern 3: Consultant / Analyst

**Focus**: Professional analysis, report writing, client communication

```yaml
domain_skills:
  - research-analysis (deep)
  - knowledge-management (basic)

software_skills:
  - documentation (required)
  - data-analysis (required)
```

**Use Case**: Management consulting, industry analysis, strategy research

## Technology Stack Recommendations

### Research & Analysis

| Use Case | Recommended Stack |
|----------|------------------|
| Data analysis | Python + Pandas + Jupyter |
| Statistical analysis | R / Python statsmodels |
| Visualization | Matplotlib / Seaborn / Plotly |
| Reports | Markdown + Pandoc / LaTeX |

### Knowledge Management

| Use Case | Recommended Stack |
|----------|------------------|
| Personal notes | Obsidian / Logseq |
| Team knowledge base | Notion / Confluence |
| Reference management | Zotero / Mendeley |
| Writing & publishing | Markdown + Static Site |

### Survey & Data Collection

| Use Case | Recommended Stack |
|----------|------------------|
| Surveys | Typeform / Google Forms |
| Qualitative analysis | NVivo / Atlas.ti |
| Interview transcription | Otter.ai + Notion |

## Anti-Patterns to Avoid

| Anti-Pattern | Problem | Better Approach |
|--------------|---------|-----------------|
| Collecting without producing | Knowledge never gets monetized | Set output goals |
| Too many tools | Attention gets fragmented | Streamline to core tools |
| No structure | Hard to retrieve | Build a tagging/linking system |
| Isolated notes | No insights emerge | Build links between notes |
