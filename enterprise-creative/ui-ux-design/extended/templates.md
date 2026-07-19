# UI/UX Design Templates

## Design Handoff Checklist

```markdown
## Design Spec Document Should Include

### Visual Specs
- [ ] Color variables (including dark mode)
- [ ] Typography specs (font, size, line height, weight)
- [ ] Spacing system
- [ ] Border radius specs
- [ ] Shadow specs
- [ ] Animation/transition durations

### Component Specs
- [ ] Styles for each state (default, hover, active, disabled, focus, error)
- [ ] Size variants (sm, md, lg)
- [ ] Responsive behavior

### Interaction Specs
- [ ] Animation descriptions (duration, easing function)
- [ ] Gesture support (swipe, pinch, etc.)
- [ ] Loading states

### Asset Export
- [ ] Icons (SVG, multiple sizes)
- [ ] Images (1x, 2x, 3x)
- [ ] Logo (all formats, all background variants)
```

## Usability Test Report Template

```markdown
## Usability Test Report

### Test Overview
- Test date:
- Number of participants:
- Prototype version:

### Task Success Rate

| Task | Success Rate | Average Time | Difficulty Rating |
|------|--------|----------|----------|
| Task 1 | | | |
| Task 2 | | | |

### Issues Found

#### Critical Issues
- [ ] Issue description → Suggested solution

#### Major Issues
- [ ] Issue description → Suggested solution

#### Minor Issues
- [ ] Issue description → Suggested solution

### User Feedback Highlights
- Positive feedback
- Suggestions for improvement
```

## Design Token Structure Example

```javascript
// Design Token structure example
{
  "color": {
    "primary": {
      "50": "#E3F2FD",
      "500": "#2196F3",  // Primary color
      "900": "#0D47A1"
    },
    "semantic": {
      "success": "#4CAF50",
      "warning": "#FF9800",
      "error": "#F44336"
    }
  },
  "spacing": {
    "xs": "4px",
    "sm": "8px",
    "md": "16px",
    "lg": "24px",
    "xl": "32px"
  },
  "typography": {
    "heading-1": {
      "fontSize": "32px",
      "lineHeight": "40px",
      "fontWeight": "700"
    }
  },
  "shadow": {
    "sm": "0 1px 2px rgba(0,0,0,0.05)",
    "md": "0 4px 6px rgba(0,0,0,0.1)"
  }
}
```

## Steps for Building a Design System

### Phase 1: Audit
- Collect screenshots of the existing UI
- Identify inconsistencies
- List all component variants

### Phase 2: Define
- Establish Design Tokens (color, typography, spacing)
- Define core component specs
- Write usage guidelines

### Phase 3: Build
- Figma component library
- Code component library
- Documentation site

### Phase 4: Maintain
- Version control
- Contribution process
- Periodic review and updates
