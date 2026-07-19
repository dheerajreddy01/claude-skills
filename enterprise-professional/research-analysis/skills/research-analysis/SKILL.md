---
schema: "1.0"
name: research-analysis
version: "1.0.0"
description: Research analysis: literature review, competitive analysis, market research, data analysis methodology
domain: professional
triggers:
  keywords:
    primary: [research, analysis, market research, literature review, competitor analysis, market survey]
    secondary: [report, survey, insight, questionnaire, interview, data analysis]
  context_boost: [methodology, quantitative, qualitative, framework]
  context_penalty: [code, api, frontend, backend]
  priority: high
dependencies:
  software-skills: [documentation]
author: claude-domain-skills
---

# Research Analysis

> A systematic research methodology and analysis framework

## Applicable Scenarios

- Market research and competitive analysis
- Literature review and research synthesis
- User research and needs analysis
- Data analysis and insight reports

## Research Process Framework

```
1. Define the problem → 2. Collect data → 3. Analyze and organize → 4. Extract insights → 5. Recommend actions
```

| Stage | Key activities |
|------|----------|
| **Define** | Research purpose, key questions, success criteria |
| **Collect** | Primary data (interviews, surveys), secondary data (literature, databases) |
| **Analyze** | Qualitative (coding, thematic analysis), quantitative (statistics, visualization) |
| **Extract** | Key findings, pattern recognition, causal inference |
| **Recommend** | Strategic recommendations, action plans, risk assessment |

## Competitive Analysis Framework

### Analysis Dimensions

| Dimension | Analysis points |
|------|----------|
| **Product features** | Core features, differentiators, feature completeness |
| **User experience** | Interface design, usage flow, learning curve |
| **Business model** | Pricing strategy, revenue sources, customer acquisition |
| **Market positioning** | Target audience, value proposition, brand image |
| **Technical architecture** | Tech stack, scalability, security |

### Competitive Matrix Template

```markdown
| Dimension | Us | Competitor A | Competitor B | Competitor C |
|------|------|-------|-------|-------|
| Core features | ★★★ | ★★★★ | ★★ | ★★★ |
| Pricing | $99 | $149 | $49 | $79 |
| Target audience | SMB | Enterprise | Startup | SMB |
| Differentiation | Easy to use | Feature-complete | Low price | Broad integrations |
```

## Market Research Methods

### Qualitative Research

| Method | Applicable scenario | Sample size |
|------|----------|--------|
| **In-depth interviews** | Explore motivations, understand context | 8-15 people |
| **Focus groups** | Gather diverse perspectives, spark discussion | 6-10 people/group |
| **Observation** | Understand actual behavior, discover pain points | Varies |
| **Diary studies** | Track long-term behavior changes | 10-20 people |

### Quantitative Research

| Method | Applicable scenario | Sample size |
|------|----------|--------|
| **Surveys** | Validate hypotheses, quantify trends | 100+ |
| **A/B testing** | Compare the effectiveness of options | Depends on traffic |
| **Data analysis** | Discover patterns, predict trends | Big data |

## Common Analysis Frameworks

| Framework | Purpose | Applicable scenario |
|------|------|----------|
| **SWOT** | Strengths, weaknesses, opportunities, threats | Strategic planning |
| **PEST** | Political, economic, social, technological environment | Macro analysis |
| **Porter's 5 Forces** | New entrants, supplier power, buyer power, substitutes, competitive rivalry | Industry analysis |
| **Jobs-to-be-Done** | User task motivation | Product innovation |
| **Affinity Diagram** | Organize and synthesize information | Qualitative analysis |

## Hypothesis-Driven Analysis

### Hypothesis Formula

```
IF [take action X] → THEN [produces result Y] → BECAUSE [based on reason Z]
```

### Validation Process

1. **Form a hypothesis** - Propose a testable hypothesis based on observation
2. **Design the validation** - Determine data needs and collection method
3. **Collect evidence** - Gather data that supports or refutes it
4. **Analyze results** - Determine whether the evidence supports the hypothesis
5. **Refine and confirm** - If confirmed, proceed to recommendations; otherwise refine and retest

## Triangulation

Use multiple sources to validate the same conclusion, increasing credibility:

| Type | Description | Example |
|------|------|------|
| **Data triangulation** | Multiple data sources | Interviews + surveys + behavioral data |
| **Method triangulation** | Multiple research methods | Qualitative + quantitative |
| **Researcher triangulation** | Multiple independent analysts | Two analysts coding independently |

## Questionnaire Design Points

**Structure**: Simple questions at the start → core questions in the middle → demographics at the end

**Principles**:
- One question per topic
- Avoid leading questions
- Provide a "not applicable" option
- 5-10 minutes is ideal (about 15-25 questions)

**Question types**: single choice, multiple choice, scale, matrix, open-ended, ranking

## Choosing Data Visualizations

| Purpose | Recommended chart |
|------|----------|
| **Comparison** | Bar chart, line chart, stacked bar chart |
| **Distribution** | Histogram, box plot, scatter plot |
| **Composition** | Pie chart (< 6 categories), treemap |
| **Trend** | Line chart, area chart |

**Principles**: title should convey the message, Y-axis starts at 0, consistent color coding, cite sources

## Research Quality Standards

### Qualitative Research

| Standard | How to achieve it |
|------|----------|
| **Credibility** | Triangulation, member checking |
| **Transferability** | Thick description, clear context |
| **Dependability** | Detailed records, audit trail |
| **Confirmability** | Reflexive journal, peer review |

### Quantitative Research

| Standard | Evaluation method |
|------|----------|
| **Internal validity** | Control variables, random assignment |
| **External validity** | Sample representativeness |
| **Reliability** | Cronbach's alpha |
| **Construct validity** | Factor analysis |

## Research Ethics

**Required**: informed consent, privacy protection, honest reporting, source citation, stating limitations

**Avoid**: manipulating data, selective reporting, unauthorized use of personal data, overstating generalizability

## Recommended Tools

| Purpose | Tools |
|------|------|
| **Reference management** | Zotero, Mendeley |
| **Qualitative analysis** | NVivo, ATLAS.ti, Dovetail |
| **Surveys** | Typeform, SurveyMonkey |
| **Data visualization** | Tableau, Power BI |
| **AI assistance** | Elicit, Consensus, Claude |

## Extended Resources

See the `extended/` directory for detailed templates and guides:
- `templates.md` - Research plan, literature review, report, and interview guide templates
- `examples.md` - Hypothesis analysis, triangulation, and questionnaire design examples
- `checklists.md` - Research ethics, quality assessment, and AI usage checklists

## References

- [Research Methods Knowledge Base](https://conjointly.com/kb/)
- [Nielsen Norman Group - UX Research](https://www.nngroup.com/topic/user-research/)
- [IDEO Design Kit](https://www.designkit.org/methods)
