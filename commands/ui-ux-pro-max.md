---
description: Comprehensive UI/UX design workflow — styles, color palettes, typography, UX guidelines, and stack-specific patterns.
argument-hint: <what to design>
---

# /ui-ux-pro-max — AI-Powered Design Intelligence

$ARGUMENTS

---

## Purpose

Comprehensive design guide for web and mobile applications. Synthesizes styles, color palettes, font pairings, UX guidelines, and chart types across multiple stacks.

> **Note:** The original antigravity-kit shipped a search script with an indexed design database. Here we rely on the `frontend-design`, `web-design-guidelines`, and `tailwind-patterns` skills plus the `frontend-specialist` agent.

---

## Workflow

### Step 1: Analyze Requirements

Extract:

- **Product type**: SaaS, e-commerce, portfolio, dashboard, landing, etc.
- **Style keywords**: minimal, playful, professional, elegant, dark mode
- **Industry**: healthcare, fintech, gaming, education
- **Stack**: React, Vue, Next.js, html-tailwind (default), Flutter, SwiftUI, Jetpack Compose, shadcn/ui

### Step 2: Invoke `frontend-specialist`

Pass the requirements above. The agent will:

1. Load the `frontend-design` skill (design tokens, component patterns)
2. Load the `web-design-guidelines` skill (accessibility, contrast rules)
3. Load `tailwind-patterns` if the stack is Tailwind-based
4. Propose a **complete design system**: pattern, style, colors, typography, effects
5. List **anti-patterns** to avoid

### Step 3: (Optional) Persist the Design System

If you want the system reusable across pages:

```
design-system/
├── MASTER.md          — global source of truth
└── pages/
    ├── dashboard.md   — page-specific overrides
    ├── checkout.md
    └── …
```

When building a specific page: first check `design-system/pages/<page>.md`. If absent, fall back to `MASTER.md`.

### Step 4: Implement

The `frontend-specialist` generates components following the design system.

---

## Common Rules for Professional UI

### Icons & Visuals

| Rule | Do | Don't |
|------|----|----- |
| No emoji icons | SVG icons (Heroicons, Lucide, Simple Icons) | Emojis like 🎨 🚀 ⚙️ |
| Stable hover states | Color/opacity transitions | Scale transforms that shift layout |
| Correct brand logos | Official SVG from Simple Icons | Guessed paths |
| Consistent icon sizing | Fixed `24x24` viewBox with `w-6 h-6` | Random mixed sizes |

### Interaction

| Rule | Do | Don't |
|------|----|----- |
| `cursor-pointer` on clickables | Always | Default cursor on interactive elements |
| Hover feedback | Color/shadow/border change | No indication |
| Transitions | `transition-colors duration-200` | Instant or >500ms |

### Light/Dark Mode

| Rule | Do | Don't |
|------|----|----- |
| Glass card light mode | `bg-white/80`+ | `bg-white/10` (too transparent) |
| Light mode body text | `#0F172A` (slate-900) | `#94A3B8` (slate-400) |
| Muted light text | `#475569` (slate-600) min | gray-400 or lighter |
| Border visibility | `border-gray-200` | `border-white/10` (invisible) |

### Layout

| Rule | Do | Don't |
|------|----|----- |
| Floating navbar | `top-4 left-4 right-4` spacing | `top-0 left-0 right-0` |
| Content padding | Offset for fixed navbar height | Content hidden behind fixed elements |
| Max-width consistency | One of `max-w-6xl` / `max-w-7xl` | Random mixed widths |

---

## Pre-Delivery Checklist

### Visual Quality
- [ ] No emojis used as icons
- [ ] All icons from one set (Heroicons / Lucide)
- [ ] Brand logos verified from Simple Icons
- [ ] Hover states don't cause layout shift

### Interaction
- [ ] All clickables have `cursor-pointer`
- [ ] Hover states give clear visual feedback
- [ ] Transitions smooth (150–300ms)
- [ ] Focus states visible for keyboard nav

### Light/Dark Mode
- [ ] Light mode text contrast ≥ 4.5:1
- [ ] Glass / transparent elements visible in light
- [ ] Borders visible in both modes
- [ ] Both modes tested

### Layout
- [ ] Floating elements have edge spacing
- [ ] No content hidden behind fixed navbars
- [ ] Responsive at 375 / 768 / 1024 / 1440px
- [ ] No horizontal scroll on mobile

### Accessibility
- [ ] All images have alt text
- [ ] Form inputs have labels
- [ ] Color is never the sole indicator
- [ ] `prefers-reduced-motion` respected

---

## Examples

```
/ui-ux-pro-max landing page for a beauty spa
/ui-ux-pro-max SaaS dashboard for fintech analytics
/ui-ux-pro-max mobile onboarding for a fitness app
/ui-ux-pro-max admin panel for a CRM with a minimal aesthetic
```

---

## Related

| Need | Skill / Agent |
|------|---------------|
| Design tokens + components | `frontend-design` |
| Accessibility + contrast | `web-design-guidelines` |
| Tailwind v4 patterns | `tailwind-patterns` |
| Next.js / React perf | `nextjs-react-expert` |
| Implementation | `frontend-specialist` agent |
