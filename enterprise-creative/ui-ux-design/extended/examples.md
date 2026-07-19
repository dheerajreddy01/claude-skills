# UI/UX Design Examples and Detailed Guides

## Responsive Design Details

### Common Breakpoints (Mobile First)

| Device | Width | CSS |
|------|------|-----|
| Phone | < 640px | Default |
| Tablet | >= 768px | @media (min-width: 768px) |
| Laptop | >= 1024px | @media (min-width: 1024px) |
| Desktop | >= 1280px | @media (min-width: 1280px) |
| Large screen | >= 1536px | @media (min-width: 1536px) |

### Responsive Strategy

**Content hierarchy:**
- Phone: stacked layout, single column
- Tablet: two columns or adjusted layout
- Desktop: full multi-column layout

**Interaction adjustments:**
- Phone: larger touch targets (44px+)
- Tablet: accommodate both touch and pointer
- Desktop: hover states, fine-grained interactions

### Responsive Component Patterns

1. **Hamburger Menu**
   - Phone: collapses into a hamburger menu
   - Desktop: expands into horizontal navigation

2. **Card Reflow**
   - Phone: 1 column
   - Tablet: 2 columns
   - Desktop: 3-4 columns

3. **Table Transformation**
   - Phone: stacked cards or horizontal scroll
   - Desktop: standard table

4. **Sidebar**
   - Phone: collapsed or bottom sheet
   - Desktop: persistent sidebar

---

## Motion Design Details

### Motion Principles

| Principle | Description | Implementation |
|------|------|------|
| **Purposeful** | Every animation should have meaning | Guide attention, express relationships |
| **Natural** | Match physical intuition | Use easing functions |
| **Fast** | Don't make users wait | Mostly 200-300ms |
| **Consistent** | Same action, same motion | Establish motion guidelines |

### Easing Function Reference

**ease-out (deceleration)**
- Use for: elements entering the screen, expanding actions
- Function: `cubic-bezier(0.0, 0.0, 0.2, 1)`

**ease-in (acceleration)**
- Use for: elements leaving the screen, collapsing actions
- Function: `cubic-bezier(0.4, 0.0, 1, 1)`

**ease-in-out (accelerate then decelerate)**
- Use for: position changes, state transitions
- Function: `cubic-bezier(0.4, 0.0, 0.2, 1)`

### Timing Recommendations

| Type | Duration | Example |
|------|------|------|
| Quick feedback | 100-150ms | Button hover |
| Standard transition | 200-300ms | Modal opening |
| Complex animation | 300-500ms | Page transition |

---

## Dark Mode Design Guide

### Color Adjustment Principles

**Not a simple inversion**
- Wrong: black background + white text + inverted colors
- Right: redesign the color scale while maintaining hierarchy

### Background Levels

| Level | Color | Purpose |
|------|------|------|
| Base layer | #121212 | Base |
| Surface layer 1 | #1E1E1E | Cards, panels |
| Surface layer 2 | #2D2D2D | Popovers |
| Surface layer 3 | #383838 | Floating elements |

### Text Colors

| Type | Opacity | Value |
|------|--------|------|
| Primary text | 87% white | rgba(255,255,255,0.87) |
| Secondary text | 60% white | rgba(255,255,255,0.6) |
| Disabled text | 38% white | rgba(255,255,255,0.38) |

### Accent Color Adjustments
- Light-mode accent colors may be too intense
- Use a lower-saturation version
- Ensure contrast still meets the AA standard
