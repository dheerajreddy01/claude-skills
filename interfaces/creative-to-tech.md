# Creative → Tech Interface

> Mapping creative domain requirements to technical implementation

## Domain Skills Covered

- `ui-ux-design` - UI/UX design
- `game-design` - Game design
- `storytelling` - Story creation
- `visual-media` - Visual media production
- `brainstorming` - Creative ideation

## Requirement → Technology Mapping

| Domain Requirement | Technical Implementation | Software Skills |
|-------------------|-------------------------|-----------------|
| Interface design implementation | React/Vue + CSS | `frontend` |
| Game development | Unity/Godot/Phaser | `game-development` |
| Design system | Component Library | `frontend`, `documentation` |
| Interactive prototype | Framer/Protopie → Code | `frontend` |
| Animation effects | CSS/GSAP/Lottie | `frontend` |
| Responsive design | Media Queries + Flexbox/Grid | `frontend` |
| Content publishing | CMS/Blog Platform | `content-platforms` |
| Game backend | Multiplayer Server | `backend`, `realtime-systems` |

## Common Combination Patterns

### Pattern 1: Product Designer (Who Codes)

**Focus**: Design-to-implementation, design systems, prototype development

```yaml
domain_skills:
  - ui-ux-design (deep)

software_skills:
  - frontend (required)
  - documentation (recommended)
```

**Use Case**: Design-to-Code, design system maintenance, interactive prototypes

### Pattern 2: Indie Game Developer

**Focus**: Indie game development, full-stack capability

```yaml
domain_skills:
  - game-design (deep)
  - storytelling (basic)

software_skills:
  - game-development (required)
  - frontend (recommended - for UI)
```

**Use Case**: Indie games, game jams, prototype development

### Pattern 3: Multiplayer Game Developer

**Focus**: Multiplayer games, real-time sync, server architecture

```yaml
domain_skills:
  - game-design (deep)

software_skills:
  - game-development (required)
  - backend (required)
  - realtime-systems (required)
  - database (recommended)
```

**Use Case**: MMOs, multiplayer battles, real-time sync games

### Pattern 4: Content Creator (Tech-Enabled)

**Focus**: Self-publishing, content distribution, tech-assisted creation

```yaml
domain_skills:
  - storytelling (deep)
  - visual-media (basic)

software_skills:
  - content-platforms (recommended)
  - automation-scripts (optional)
```

**Use Case**: Blogs, ebooks, automated publishing

### Pattern 5: Interactive Experience Designer

**Focus**: Interactive art, digital experiences, creative technology

```yaml
domain_skills:
  - ui-ux-design (basic)
  - brainstorming (basic)

software_skills:
  - frontend (required)
  - game-development (recommended)
```

**Use Case**: Interactive websites, digital art, immersive experiences

## Technology Stack Recommendations

### Design Implementation

| Use Case | Recommended Stack |
|----------|------------------|
| Web UI | React/Vue + Tailwind/Styled |
| Design System | Storybook + Component Library |
| Animation | Framer Motion / GSAP |
| 3D Web | Three.js / React Three Fiber |

### Game Development

| Use Case | Recommended Stack |
|----------|------------------|
| 2D games | Godot / Phaser.js |
| 3D games | Unity / Unreal |
| Web games | Phaser / PixiJS |
| Multiplayer game backend | Colyseus / Socket.io |

### Content Publishing

| Use Case | Recommended Stack |
|----------|------------------|
| Blog | Next.js + MDX / Ghost |
| CMS | Strapi / Sanity / Contentful |
| Video | YouTube API / Cloudflare Stream |

## Creative-Tech Synergies

### Design → Code Conversion Flow

```
Figma Design → Design Tokens → Component Code → Production
     ↓              ↓               ↓
 Manual design   JSON/CSS variables   React/Vue components
```

### Game Design → Implementation Flow

```
GDD Document → Prototype → Core Loop → Polish → Ship
     ↓            ↓           ↓         ↓
 Game plan     Quick validation   Core gameplay   Detail polish
```

## Anti-Patterns to Avoid

| Anti-Pattern | Problem | Better Approach |
|--------------|---------|-----------------|
| Over-engineered handoffs | Implementation drifts from design | Have the designer participate in implementation review |
| Skipping prototypes | Core gameplay never validated | Build a paper prototype first |
| Optimizing visuals too early | Wasted effort | Validate gameplay first |
| Letting tech constraints dictate design | Creativity gets limited | Imagine first, evaluate feasibility second |
