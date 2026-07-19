---
schema: "1.0"
name: ui-ux-design
version: "1.1.0"
description: UI/UX design: interface design principles, user experience, usability, accessibility design
triggers:
  keywords:
    primary: [UI, UX, interface design, user experience, wireframe]
    secondary: [usability, accessibility, prototype, design system]
  context_boost: [figma, design, component, layout, mobile, responsive]
  context_penalty: [backend, api, database, trading]
  priority: high
keywords: [creative, design, ui, ux, interface]
dependencies:
  software-skills: [frontend]
author: claude-domain-skills
---

# UI/UX Design

> User-centered design thinking

## Use Cases

- Designing user interfaces and interaction flows
- Building design systems and component libraries
- Conducting usability evaluations
- Ensuring accessibility compliance
- Creating wireframes and prototypes

## Core Design Principles

| Principle | Description |
|------|------|
| **Consistency** | Use the same visual and interaction patterns for the same functionality |
| **Visibility** | Important information and actions should be easy to discover |
| **Feedback** | Every action should have immediate visual or auditory feedback |
| **Error Tolerance** | Allow users to undo actions to avoid catastrophic errors |
| **Minimalism** | Show only necessary information to reduce cognitive load |

## Visual Hierarchy

| Level | Purpose | Design Technique |
|------|------|----------|
| **Primary** | Main action | Large buttons, strong contrast colors |
| **Secondary** | Secondary action | Smaller, outlined buttons |
| **Tertiary** | Supporting information | Text links, light colors |

Visual weight priority: Size > Color contrast > Weight > Position > Spacing

## Spacing System (8pt Grid)

| Token | Value | Purpose |
|-------|-----|------|
| xs | 4px | Compact spacing |
| sm | 8px | Related elements |
| md | 16px | Block spacing |
| lg | 24px | Block spacing |
| xl | 32px | Section spacing |
| xxl | 48px | Large block separation |

## Color Usage

### Semantic Colors

| Color | Purpose | Hex Example |
|------|------|----------|
| Primary | Brand primary color, main CTA | #0066FF |
| Success | Success, confirmation, completion | #00C853 |
| Warning | Warning, needs attention | #FFB300 |
| Error | Error, danger, delete | #FF3D00 |
| Info | Information, hint | #00B0FF |

### Contrast Requirements (WCAG 2.1)

| Level | Normal Text | Large Text |
|------|----------|----------|
| AA | 4.5:1 | 3:1 |
| AAA | 7:1 | 4.5:1 |

## Accessibility Design (WCAG POUR)

| Principle | Core Requirement |
|------|----------|
| **Perceivable** | Images have alt text, color is not the only means of conveying information, AA contrast |
| **Operable** | Keyboard operable, clear focus, no flashing |
| **Understandable** | Language specified, clear form labels, consistent navigation |
| **Robust** | Semantic HTML, correct ARIA usage, cross-device compatibility |

### Common Fixes

| Issue | Solution |
|------|----------|
| Image missing alt | `<img alt="description">` |
| Low contrast | Use a contrast checking tool |
| No keyboard navigation | Ensure `tabindex` is correct |
| No focus indicator | Add `:focus` styles |

> See `extended/checklists.md` for the detailed WCAG checklist

> **U.S. legal context**: The Americans with Disabilities Act (ADA) has been applied by U.S. courts to websites and apps, and DOJ guidance points to WCAG 2.1 AA as the practical compliance benchmark. Section 508 imposes WCAG-aligned requirements on federal agencies and contractors. Treat AA conformance as a legal floor, not just a UX nicety, for any U.S.-facing product.

## Interaction Design Patterns

### Navigation Patterns
- **Tab Bar**: 3-5 main functions (mobile devices)
- **Side Nav**: Multi-level navigation (desktop)
- **Breadcrumb**: Backtracking through deep structures

### Input Patterns
- **Inline Validation**: Real-time validation
- **Progressive Disclosure**: Gradual reveal
- **Default Values**: Pre-fill reasonable defaults

### Feedback Patterns
- **Toast**: Non-blocking notification
- **Modal**: Requires user decision
- **Skeleton**: Loading placeholder

## Design System Components

### Basic Components
Button, Input, Select, Checkbox/Radio, Toggle, Icon

### Composite Components
Card, Modal, Toast, Table, Pagination, Navigation

### Layout Components
Container, Grid, Stack, Divider

## Nielsen's 10 Usability Heuristics

| # | Heuristic | Check Question |
|---|--------|----------|
| 1 | Visibility of system status | Does the user know what's currently happening? |
| 2 | Match between system and the real world | Does it use language familiar to the user? |
| 3 | User control and freedom | Can the user undo/go back? |
| 4 | Consistency and standards | Does it follow platform conventions? |
| 5 | Error prevention | Does it prevent errors from occurring? |
| 6 | Recognition rather than recall | Are options visible? |
| 7 | Flexibility and efficiency of use | Are there shortcuts? |
| 8 | Aesthetic and minimalist design | Does it show only necessary information? |
| 9 | Help users recognize, diagnose, and recover from errors | Are error messages helpful? |
| 10 | Help and documentation | Can help be found when needed? |

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| Distinguishing function by button color | Distinguish by size/position/text |
| Using placeholder as a label | Use a separate label |
| No hover state | All interactive elements should have hover |
| Autoplaying videos | Require the user to initiate playback |

## Types of User Testing

| Type | Description | Applicable Stage |
|------|------|------|
| Exploratory | Understand how users naturally use the product | Early prototype |
| Task-oriented | Complete specific tasks | Mid-design |
| Comparative testing | A vs B options | Decision stage |
| Benchmark testing | Quantify performance metrics | Before/after launch |

> See `extended/checklists.md` for the test preparation checklist

## Responsive Design

### Breakpoints (Mobile First)

| Device | Width |
|------|------|
| Phone | < 640px (default) |
| Tablet | >= 768px |
| Laptop | >= 1024px |
| Desktop | >= 1280px |

### Key Strategies
- Phone: 44px+ touch targets, single-column layout
- Desktop: hover states, multi-column layout

> See `extended/examples.md` for a detailed responsive design guide

## Motion Design Principles

| Principle | Implementation |
|------|------|
| Purposeful | Guide attention, express relationships |
| Natural | Use easing functions |
| Fast | Mostly 200-300ms |
| Consistent | Establish motion guidelines |

> See `extended/examples.md` for detailed easing function references

## Dark Mode Key Points

- Not a simple inversion — color scales need to be redesigned
- Background levels: #121212 → #1E1E1E → #2D2D2D
- Text: 87% / 60% / 38% white opacity
- Reduce saturation of accent colors to ensure contrast

> See `extended/examples.md` for the complete dark mode guide

## Building a Design System

Audit → Define → Build → Maintain

> See `extended/templates.md` for detailed steps and Design Token examples

## Recommended Tools

- **Design**: Figma, Sketch
- **Prototyping**: Figma, ProtoPie
- **Accessibility Testing**: axe, WAVE
- **Contrast**: WebAIM Contrast Checker

## Related Resources

- [Material Design](https://m3.material.io/)
- [Apple HIG](https://developer.apple.com/design/human-interface-guidelines/)
- [Nielsen Norman Group](https://www.nngroup.com/articles/)
- [WCAG 2.1](https://www.w3.org/WAI/WCAG21/quickref/)

## Further Reading

- `extended/templates.md` - Design handoff checklist, test report template, Design Tokens
- `extended/examples.md` - Detailed guides for responsive design, motion, and dark mode
- `extended/checklists.md` - WCAG, usability testing, and pre-launch checklists
