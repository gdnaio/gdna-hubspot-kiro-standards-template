---
title: g/d/n/a Brand Standards
inclusion: always
---

# g/d/n/a Brand Standards

## Color Palette (gdna.io Primary Brand)

### Primary Colors
| Name | Hex | Usage |
|------|-----|-------|
| White | `#FFFFFF` | Backgrounds, clean space, text on dark |
| Offwhite | `#F9F9F9` | Subtle backgrounds, cards, containers |
| Squid Ink | `#002D43` | Primary brand color, headers, emphasis |
| Black | `#000000` | Body text, strong contrast |

### Accent Colors (IOEM Model)
| Name | Hex | Pastel | Phase | Usage |
|------|-----|--------|-------|-------|
| Orange | `#FF9900` | `#FFD699` | Execution | CTAs, execution emphasis, AWS alignment |
| Blue | `#0096FF` | `#99D5FF` | Innovation | Innovation projects, ideation, discovery |
| Green | `#B6FFBB` | `#E2FFE4` | Managed | Stability, operations, managed services |
| Pink | `#FF66B2` | `#FFC2E0` | Optimization | Optimization, efficiency, improvement |

### Supporting Neutrals
| Name | Hex | Pastel | Usage |
|------|-----|--------|-------|
| Gray | `#ACB7C5` | `#DEE2E8` | Borders, dividers, subtle text |
| Darker Gray | `#6E7A84` | `#C5CACE` | Secondary text, icons, disabled states |

### Color Rules
- White or Offwhite backgrounds with Squid Ink headers
- Orange for CTAs and key actions (AWS-aligned)
- Blue for innovation-focused content
- Green for stability/reliability messaging
- Pink for optimization themes
- Never mix too many accents in one composition
- Pastels for backgrounds/highlights only, never as primary
- Avoid Orange/Blue clashes (separate visually)

## Typography

### Primary Typeface: Amazon Ember

| Weight | File Ref | Usage |
|--------|----------|-------|
| Amazon Ember Display Light | `Ember_lt` | Headlines, hero text, page titles |
| Amazon Ember Bold | `Ember_md` | Subheads, section headers, CTAs |
| Amazon Ember Light | `Ember_lt` | Body copy, descriptions |
| Amazon Ember Condensed | `Ember_con` | Compact layouts (use sparingly) |
| Amazon Ember Condensed Bold | `Ember_c_bold` | Compact emphasis |

### Web Typography Hierarchy
| Element | Font | Notes |
|---------|------|-------|
| H1 | Amazon Ember Display Light | Largest, sentence case, punctuation |
| H2 | Amazon Ember Bold | Major sections |
| H3 | Amazon Ember Bold | Subsections |
| Body | Amazon Ember Light | Left-aligned, max 15 words/line |
| Captions | Amazon Ember Light Italic | Smaller than body |

### Typography Rules
- Always sentence case for headlines (never ALL CAPS for body)
- Left-aligned only (never justified or centered for body)
- Line height: 1.5-1.75x font size
- Max 15 words per line
- Bold for key terms, italic for notes/captions
- Underline reserved for links only

## Button Styles

### Primary CTA
- Background: Orange `#FF9900`
- Text: White `#FFFFFF`
- Font: Amazon Ember Bold

### Secondary CTA
- Background: White `#FFFFFF`
- Border: Squid Ink `#002D43`
- Text: Squid Ink `#002D43`
- Font: Amazon Ember Bold

## Web Colors
- Background: White `#FFFFFF` or Offwhite `#F9F9F9`
- Primary text: Squid Ink `#002D43` or Black `#000000`
- Links: Blue `#0096FF`
- CTAs: Orange `#FF9900`

## CSS Custom Properties

Use these in all theme CSS:
```css
:root {
  --color-white: #FFFFFF;
  --color-offwhite: #F9F9F9;
  --color-squid-ink: #002D43;
  --color-black: #000000;
  --color-orange: #FF9900;
  --color-blue: #0096FF;
  --color-green: #B6FFBB;
  --color-pink: #FF66B2;
  --color-gray: #ACB7C5;
  --color-darker-gray: #6E7A84;
  --color-orange-pastel: #FFD699;
  --color-blue-pastel: #99D5FF;
  --color-green-pastel: #E2FFE4;
  --color-pink-pastel: #FFC2E0;
  --color-gray-pastel: #DEE2E8;
  --font-display: 'Amazon Ember Display', 'Amazon Ember', sans-serif;
  --font-heading: 'Amazon Ember', sans-serif;
  --font-body: 'Amazon Ember Light', 'Amazon Ember', sans-serif;
  --font-weight-light: 300;
  --font-weight-regular: 400;
  --font-weight-bold: 700;
  --line-height-body: 1.65;
  --line-height-heading: 1.2;
  --max-content-width: 1200px;
  --max-text-width: 720px;
}
```

## Brand Voice in Visuals
- "Precision and velocity" — Orange accent
- "Built for AWS" — Orange accent (AWS alignment)
- "Listening is creation" — Blue accent
- Professional, disciplined, AWS-native aesthetic
- Generous white space, clear visual hierarchy
