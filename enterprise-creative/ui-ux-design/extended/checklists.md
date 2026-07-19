# UI/UX Design Checklists

## WCAG POUR Accessibility Checklist

### Perceivable
- [ ] All images have alt text
- [ ] Color is not the only means of conveying information
- [ ] Contrast meets AA standard (normal text 4.5:1, large text 3:1)
- [ ] Videos have captions
- [ ] Audio has a text alternative

### Operable
- [ ] All functionality is operable via keyboard
- [ ] Focus state is clearly visible
- [ ] No content that could cause seizures (flashing < 3 times per second)
- [ ] A way to skip repeated content is provided
- [ ] Page titles are meaningful

### Understandable
- [ ] Language is explicitly specified (lang attribute)
- [ ] Forms have clear labels and error messages
- [ ] Navigation is consistent
- [ ] Error messages clearly explain the problem and the solution
- [ ] Important actions can be confirmed or undone

### Robust
- [ ] Semantic HTML
- [ ] ARIA labels used correctly
- [ ] Compatible across browsers/assistive technologies
- [ ] Code passes W3C validation

---

## Usability Testing Preparation Checklist

### 1. Define Goals
- [ ] Clearly state the hypothesis to validate
- [ ] Define success metrics (task completion rate, time, satisfaction)

### 2. Design Tasks
- [ ] Prepare 3-5 core tasks
- [ ] Tasks simulate real usage scenarios
- [ ] Task descriptions do not hint at the answer

### 3. Recruit Participants
- [ ] Recruit 5-8 people (can uncover 80% of issues)
- [ ] Participants match target user characteristics
- [ ] Prepare appropriate compensation

### 4. Prepare the Environment
- [ ] Prototype/test version ready
- [ ] Recording/audio equipment tested
- [ ] Observation notes sheet prepared

### 5. Test Facilitation Script
- Opening: "We're testing the product, not you. There are no right or wrong answers."
- During the task: "Please think out loud as you go."
- If stuck: "What did you expect to happen here?"

---

## Pre-Launch Checklist

### Navigation
- [ ] Users know where they are
- [ ] Users know where they can go
- [ ] Users can easily go back
- [ ] Breadcrumbs/navigation display correctly

### Content
- [ ] Important information is above the fold
- [ ] Copy is clear and unambiguous
- [ ] Error messages are helpful
- [ ] No typos or awkward phrasing

### Interaction
- [ ] Clickable elements look clickable
- [ ] Actions provide feedback (visual/auditory)
- [ ] Form validation is real-time
- [ ] Hover/focus/active states are complete

### Performance
- [ ] Initial load < 3 seconds
- [ ] Interaction response < 100ms
- [ ] Animations are smooth (60fps)
- [ ] Images are optimized

### Cross-Device
- [ ] Mobile version tested and passing
- [ ] Tablet version tested and passing
- [ ] Major browsers tested and passing
- [ ] Touch interactions work correctly

### Accessibility
- [ ] No critical issues found by axe/WAVE
- [ ] Keyboard navigation works smoothly
- [ ] Screen reader testing passed
