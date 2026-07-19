---
schema: "1.0"
name: visual-media
version: "1.0.0"
description: Visual media creation: photography, video production, animation, film, short-form video, post-production editing
domain: creative
triggers:
  keywords:
    primary: [photography, video, animation, film, editing]
    secondary: [post-production, reels, shooting, storyboard, camera movement, color grading]
  context_boost: [visual, creative, media]
  context_penalty: [code, api, backend]
  priority: high
dependencies:
  software-skills: []
author: claude-domain-skills
---

# Visual Media

> Tell stories with visual language, create unforgettable images

## Use Cases

- Photography composition and technique
- Video planning and production
- Short-form video content creation
- Animation production pipeline
- Post-production editing and color grading

## Photography Fundamentals

### The Exposure Triangle

```
┌─────────────────────────────────────────────────────────────────┐
│  Exposure Triangle                                              │
│                                                                 │
│              Aperture                                           │
│                 /\                                              │
│                /  \                                             │
│               /    \                                            │
│              /      \                                           │
│             /        \                                          │
│            /          \                                         │
│           /            \                                        │
│          /______________\                                       │
│    Shutter Speed    ISO Sensitivity                              │
│                                                                 │
│  Aperture (f/1.4 - f/22)                                        │
│  ├─ Wide aperture (small number): shallow depth of field, blurred background │
│  └─ Narrow aperture (large number): deep depth of field, whole frame sharp │
│                                                                 │
│  Shutter Speed (1/4000s - 30s)                                  │
│  ├─ Fast shutter: freezes motion                                │
│  └─ Slow shutter: motion blur, light trails                     │
│                                                                 │
│  ISO (100 - 12800+)                                              │
│  ├─ Low ISO: fine image quality                                 │
│  └─ High ISO: usable in low light, but with noise                │
└─────────────────────────────────────────────────────────────────┘
```

### Composition Rules

```markdown
## Basic Composition

### Rule of Thirds
- Divide the frame into a 3×3 grid
- Place the subject at an intersection point
- Place the horizon on the upper/lower third line

### Leading Lines
- Use lines to guide the eye
- Roads, rivers, railings
- Point toward the subject

### Frame Composition
- Frame the subject with foreground elements
- Doorways, windows, tree branches
- Adds a sense of depth

### Symmetry and Balance
- Perfect symmetry = stable, solemn
- Asymmetry = dynamic, tension
- Balance of visual weight

### Negative Space
- Empty space draws focus
- Gives the subject "breathing room"
- Minimalist style
```

### Working with Light

| Light Type | Characteristics | Best For |
|----------|------|------|
| **Front Light** | Flat, few shadows | Documentation, ID photos |
| **Side Light** | Dimensional, has shadows | Portraits, architecture |
| **Backlight** | Silhouette, glow | Mood, artistic shots |
| **Top Light** | Harsh shadows | Avoid at midday |
| **Golden Hour** | Soft, warm tones | Landscape, portraits |
| **Blue Hour** | Cool tones, mysterious | City, night scenes |

## Video Production

### Production Pipeline

```
┌─────────────────────────────────────────────────────────────────┐
│  Three Stages of Video Production                               │
│                                                                 │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │  Pre-        │  │  Production  │  │  Post-       │          │
│  │  production  │→ │              │→ │  production  │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
│                                                                 │
│  Pre-production (30%)                                           │
│  ├─ Script/planning                                             │
│  ├─ Storyboard                                                  │
│  ├─ Casting/location scouting                                   │
│  └─ Equipment/crew                                              │
│                                                                 │
│  Production (20%)                                                │
│  ├─ On-set coordination                                         │
│  ├─ Lighting setup                                               │
│  ├─ Audio recording                                              │
│  └─ Multi-angle shooting                                         │
│                                                                 │
│  Post-production (50%)                                           │
│  ├─ Editing                                                     │
│  ├─ Color grading                                                │
│  ├─ Sound effects/score                                          │
│  └─ VFX/subtitles                                                │
└─────────────────────────────────────────────────────────────────┘
```

### Camera Movement Techniques

| Movement | Effect | Use Case |
|------|------|----------|
| **Push in** | Emphasis, closing in | Reveal key points |
| **Pull out** | Reveals the full picture | Endings, transitions |
| **Pan** | Horizontal movement | Following, establishing environment |
| **Tilt** | Vertical movement | Showing height |
| **Track** | Follows the subject | Walking, chasing |
| **Orbit** | 360-degree circling | Dramatic moments |
| **Handheld** | Sense of presence | Documentary, tension |

## Shot Types

```markdown
## Shot Types

### Wide Shot
- Establishes the scene
- Shows the environment
- Makes the character appear small

### Full Shot
- Character's full body
- Action is clear
- Shows costume

### Medium Shot
- From the waist up
- Dialogue scenes
- Hand gestures

### Close-up
- From the shoulders up
- Emotional expression
- Important dialogue

### Extreme Close-up
- Eye, hand detail
- Intense emotion
- Object detail
```

## Short-Form Video Production

### Platform Characteristics

| Platform | Optimal Length | Aspect Ratio | Characteristics |
|------|----------|------|------|
| TikTok | 15-60s | 9:16 | Entertainment, challenges |
| Reels | 15-90s | 9:16 | Polished, aesthetic |
| Shorts | 15-60s | 9:16 | Educational, comedic |
| X (Twitter) | 15-140s | 16:9 or 9:16 | News, commentary, threads |

### Short-Form Video Structure

```
┌─────────────────────────────────────────────────────────────────┐
│  The Golden Formula for Short-Form Video                        │
│                                                                 │
│  0-3 seconds: Hook                                               │
│  ├─ Question/suspense                                           │
│  ├─ Visual impact                                                │
│  └─ "Wait, watch this..."                                        │
│                                                                 │
│  3-50 seconds: Content                                           │
│  ├─ Fast pacing                                                  │
│  ├─ Segmented presentation                                       │
│  └─ Keep it fresh                                                │
│                                                                 │
│  Last 5 seconds: CTA                                              │
│  ├─ Like/follow                                                  │
│  ├─ Comment engagement                                           │
│  └─ Watch more                                                   │
│                                                                 │
│  Important: the first 3 seconds are make or break!               │
└─────────────────────────────────────────────────────────────────┘
```

## Animation Fundamentals

### The 12 Principles of Animation

```markdown
## Disney 12 Principles of Animation

1. **Squash & Stretch**
   - Preserve volume
   - Convey weight and elasticity

2. **Anticipation**
   - Crouch before a jump
   - Wind up before a punch

3. **Staging**
   - Clearly convey intent
   - Guide the viewer's eye

4. **Straight Ahead & Pose to Pose**
   - Straight ahead: fluid, organic
   - Pose to pose: precise, controlled

5. **Follow Through & Overlapping Action**
   - Delay in hair, clothing
   - Adds a sense of naturalness

6. **Slow In & Slow Out**
   - Actions start and end slower
   - Faster in the middle

7. **Arcs**
   - Natural motion follows curves
   - Avoids a mechanical feel

8. **Secondary Action**
   - Small actions beyond the main action
   - Enriches the character

9. **Timing**
   - Frame count determines speed
   - Affects sense of weight

10. **Exaggeration**
    - More intense than reality
    - Highlights features

11. **Solid Drawing**
    - Understanding of three-dimensional form
    - Weight and balance

12. **Appeal**
    - Character charisma
    - Makes people want to watch
```

### Animation Production Pipeline

```
Planning → Script → Storyboard → Design → Animation → Post-production → Output
                    ↓
              Character design
              Set design
              Color design
```

## Post-Production Editing

### Editing Rhythm

```markdown
## Editing Principles

### Cutting on Action
- Cut mid-action
- Maintains fluidity

### Cutting on Stillness
- Cut during a pause
- Common in dialogue scenes

### The 180-Degree Rule
- Maintain characters' relative positions
- Avoid crossing the line

### J-Cut / L-Cut
- J-Cut: sound before picture
- L-Cut: picture before sound
- Adds fluidity
```

### Color Grading Fundamentals

| Adjustment | Effect | Example |
|----------|------|------|
| Exposure | Overall brightness | Brighten underexposed shots |
| Contrast | Difference between light and dark | Adds dimensionality |
| Color Temperature | Warm/cool tone | Warm tones feel cozy |
| Tint | Green/magenta | Skin tone correction |
| Saturation | Color intensity | Desaturate for a cinematic feel |
| Curves | Fine-tuned adjustment | S-curve increases contrast |

## Recommended Tools

### Shooting Equipment
- **Cameras**: Sony A7, Canon R, Fujifilm
- **Phones**: iPhone Pro, Pixel
- **Stabilizers**: DJI RS, Zhiyun

### Editing Software
- **Professional**: Premiere Pro, DaVinci Resolve, Final Cut
- **Lightweight**: CapCut, iMovie, Filmora
- **VFX**: After Effects, Motion

### Animation Tools
- **2D**: Toon Boom, Clip Studio, Procreate
- **3D**: Blender, Maya, Cinema 4D
- **Motion**: After Effects, Lottie

## Related Resources

- [StudioBinder](https://www.studiobinder.com/) - Video production tutorials
- [Peter McKinnon](https://www.youtube.com/@PeterMcKinnon) - Photography/video
- [Corridor Crew](https://www.youtube.com/@CorridorCrew) - VFX tutorials
- [The Animator's Survival Kit](https://www.amazon.com/Animators-Survival-Kit-Richard-Williams/dp/0571238343)
