---
name: frontend-slides
description: Create stunning, animation-rich HTML presentations from scratch or by converting PowerPoint files. Includes Chart.js data visualization, design tokens, responsive layouts, copywriting formulas, and contextual slide strategies. Use when the user wants to build a presentation, convert a PPT/PPTX to web, or create slides for a talk/pitch. Helps non-designers discover their aesthetic through visual exploration rather than abstract choices.
metadata:
  origin: ECC
  author: claudekit
  version: "2.0.0"
---

# Frontend Slides

Create zero-dependency, animation-rich HTML presentations that run entirely in the browser.

Inspired by the visual exploration approach showcased in work by zarazhangrui (credit: @zarazhangrui).

## When to Activate

- Creating a talk deck, pitch deck, workshop deck, or internal presentation
- Converting `.ppt` or `.pptx` slides into an HTML presentation
- Improving an existing HTML presentation's layout, motion, or typography
- Exploring presentation styles with a user who does not know their design preference yet

## Non-Negotiables

1. **Zero dependencies**: default to one self-contained HTML file with inline CSS and JS.
2. **Viewport fit is mandatory**: every slide must fit inside one viewport with no internal scrolling.
3. **Show, don't tell**: use visual previews instead of abstract style questionnaires.
4. **Distinctive design**: avoid generic purple-gradient, Inter-on-white, template-looking decks.
5. **Production quality**: keep code commented, accessible, responsive, and performant.

Before generating, read `STYLE_PRESETS.md` for the viewport-safe CSS base, density limits, preset catalog, and CSS gotchas.

## Workflow

### 1. Detect Mode

Choose one path:
- **New presentation**: user has a topic, notes, or full draft
- **PPT conversion**: user has `.ppt` or `.pptx`
- **Enhancement**: user already has HTML slides and wants improvements

### 2. Discover Content

Ask only the minimum needed:
- purpose: pitch, teaching, conference talk, internal update
- length: short (5-10), medium (10-20), long (20+)
- content state: finished copy, rough notes, topic only

If the user has content, ask them to paste it before styling.

### 3. Discover Style

Default to visual exploration.

If the user already knows the desired preset, skip previews and use it directly.

Otherwise:
1. Ask what feeling the deck should create: impressed, energized, focused, inspired.
2. Generate **3 single-slide preview files** in `.ecc-design/slide-previews/`.
3. Each preview must be self-contained, show typography/color/motion clearly, and stay under roughly 100 lines of slide content.
4. Ask the user which preview to keep or what elements to mix.

Use the preset guide in `STYLE_PRESETS.md` when mapping mood to style.

### 4. Build the Presentation

Output either:
- `presentation.html`
- `[presentation-name].html`

Use an `assets/` folder only when the deck contains extracted or user-supplied images.

Required structure:
- semantic slide sections
- a viewport-safe CSS base from `STYLE_PRESETS.md`
- CSS custom properties for theme values
- a presentation controller class for keyboard, wheel, and touch navigation
- Intersection Observer for reveal animations
- reduced-motion support

### 5. Enforce Viewport Fit

Treat this as a hard gate.

Rules:
- every `.slide` must use `height: 100vh; height: 100dvh; overflow: hidden;`
- all type and spacing must scale with `clamp()`
- when content does not fit, split into multiple slides
- never solve overflow by shrinking text below readable sizes
- never allow scrollbars inside a slide

Use the density limits and mandatory CSS block in `STYLE_PRESETS.md`.

### 6. Validate

Check the finished deck at these sizes:
- 1920x1080
- 1280x720
- 768x1024
- 375x667
- 667x375

If browser automation is available, use it to verify no slide overflows and that keyboard navigation works.

### 7. Deliver

At handoff:
- delete temporary preview files unless the user wants to keep them
- open the deck with the platform-appropriate opener when useful
- summarize file path, preset used, slide count, and easy theme customization points

Use the correct opener for the current OS:
- macOS: `open file.html`
- Linux: `xdg-open file.html`
- Windows: `start "" file.html`

## PPT / PPTX Conversion

For PowerPoint conversion:
1. Prefer `python3` with `python-pptx` to extract text, images, and notes.
2. If `python-pptx` is unavailable, ask whether to install it or fall back to a manual/export-based workflow.
3. Preserve slide order, speaker notes, and extracted assets.
4. After extraction, run the same style-selection workflow as a new presentation.

Keep conversion cross-platform. Do not rely on macOS-only tools when Python can do the job.

## Implementation Requirements

### HTML / CSS

- Use inline CSS and JS unless the user explicitly wants a multi-file project.
- Fonts may come from Google Fonts or Fontshare.
- Prefer atmospheric backgrounds, strong type hierarchy, and a clear visual direction.
- Use abstract shapes, gradients, grids, noise, and geometry rather than illustrations.

### JavaScript

Include:
- keyboard navigation
- touch / swipe navigation
- mouse wheel navigation
- progress indicator or slide index
- reveal-on-enter animation triggers

### Accessibility

- use semantic structure (`main`, `section`, `nav`)
- keep contrast readable
- support keyboard-only navigation
- respect `prefers-reduced-motion`

## Content Density Limits

Use these maxima unless the user explicitly asks for denser slides and readability still holds:

| Slide type | Limit |
|------------|-------|
| Title | 1 heading + 1 subtitle + optional tagline |
| Content | 1 heading + 4-6 bullets or 2 short paragraphs |
| Feature grid | 6 cards max |
| Code | 8-10 lines max |
| Quote | 1 quote + attribution |
| Image | 1 image constrained by viewport |

## Anti-Patterns

- generic startup gradients with no visual identity
- system-font decks unless intentionally editorial
- long bullet walls
- code blocks that need scrolling
- fixed-height content boxes that break on short screens
- invalid negated CSS functions like `-clamp(...)`

## Chart.js Data Visualization

Integrate live, interactive charts into HTML presentations using Chart.js.

### When to Use Chart.js in Presentations

- **Data-heavy decks** (QBRs, board meetings, investor pitches) where static numbers land flat
- **Before/after metrics** to show growth, conversion, or impact
- **Comparison slides** where multiple datasets need side-by-side visualization
- **Trend storytelling** — line charts for revenue, bar charts for category breakdowns
- **Market sizing** — doughnut or polar area charts for TAM/SAM/SOM

### Available Chart Types

| Type | Chart.js `type` | Best For |
|------|-----------------|----------|
| Bar | `'bar'` | Comparisons, categories, discrete data |
| Line | `'line'` | Trends over time, continuous data |
| Pie | `'pie'` | Proportional parts of a whole (few segments) |
| Doughnut | `'doughnut'` | Same as pie, with center whitespace for labels |
| Radar | `'radar'` | Multi-dimensional comparisons (e.g., skill assessments) |
| Polar Area | `'polarArea'` | Segments with varying magnitudes |
| Bubble | `'bubble'` | Three-dimensional data (x, y, radius) |
| Scatter | `'scatter'` | Correlation and distribution |

### How to Integrate Chart.js in HTML Slides

Include Chart.js from CDN in the `<head>`:

```html
<script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.1/dist/chart.umd.min.js"></script>
```

Add a canvas element inside the slide:

```html
<div class="slide">
  <div class="slide-content">
    <h2>Revenue Growth</h2>
    <div class="chart-container" style="width: min(80%, 600px); height: clamp(200px, 40vh, 350px);">
      <canvas id="revenueChart"></canvas>
    </div>
  </div>
</div>
```

Initialize the chart in a script block (after slide navigation code, or inside a `DOMContentLoaded` listener):

```html
<script>
new Chart(document.getElementById('revenueChart'), {
    type: 'line',
    data: {
        labels: ['Sep', 'Oct', 'Nov', 'Dec'],
        datasets: [{
            label: 'MRR ($K)',
            data: [5, 12, 28, 45],
            borderColor: '#FF6B6B',
            backgroundColor: 'rgba(255, 107, 107, 0.1)',
            borderWidth: 3,
            fill: true,
            tension: 0.4
        }]
    },
    options: {
        responsive: true,
        maintainAspectRatio: false,
        plugins: { legend: { display: false } },
        scales: {
            x: { grid: { color: 'rgba(255,255,255,0.05)' }, ticks: { color: '#B8B8D0' } },
            y: { grid: { color: 'rgba(255,255,255,0.05)' }, ticks: { color: '#B8B8D0' } }
        }
    }
});
</script>
```

### Responsive Charts

- Set `responsive: true` and **`maintainAspectRatio: false`** so the chart fills its container height.
- Wrap the canvas in a `chart-container` div with height constrained via `clamp()`.
- On mobile, charts remain readable because the container scales down — test at 375px wide.
- For pie/doughnut charts, keep segment count low (≤6) to maintain legibility on small screens.
- Always use dark-themed grid colors (`rgba(255,255,255,0.05)`) and light tick colors (`#B8B8D0`) for dark presentation backgrounds.

## Strategic Presentation Design

Beyond visual polish — structure slides for persuasion, clarity, and audience engagement.

### Layout Patterns

Choose a layout that matches the slide's purpose:

| Layout | Use Case | CSS Grid |
|--------|----------|----------|
| Title Slide | Opening / first impression | `flex; center` |
| Two Column Split | Compare / contrast | `grid-template-columns: 1fr 1fr` |
| Feature Grid | Show capabilities (3-6 cards) | `grid-template-columns: repeat(3, 1fr)` |
| Metrics Dashboard | Display KPIs (3-4 metrics) | `grid-template-columns: repeat(4, 1fr)` |
| Comparison Table | Compare options | Use semantic `<table>` |
| Timeline Flow | Show progression | Vertical or horizontal flex |
| Team Grid | Introduce people | `grid-template-columns: repeat(auto-fill, minmax(180px, 1fr))` |
| Quote Testimonial | Customer endorsement | Centered blockquote |
| Big Number Hero | Single powerful metric | Centered, oversized number |
| Product Screenshot | Show product UI | Image constrained + overlay |
| Pricing Cards | Present tiers | `grid-template-columns: repeat(3, 1fr)` |
| CTA Closing | Drive action | Centered, bold call-to-action |

All grid layouts must include mobile breakpoints — collapse to single column below 768px.

Visual treatments per slide type:

| Treatment | When to Use |
|-----------|-------------|
| `gradient-glow` | Title slides, CTAs |
| `subtle-border` | Problem statements |
| `icon-top` | Feature grids |
| `screenshot-shadow` | Product screenshots |
| `popular-highlight` | Pricing (scale 1.05) |
| `bg-overlay` | Background images |
| `contrast-pair` | Before/after |
| `logo-grayscale` | Client logos |

### Copywriting Formulas

Apply these persuasion frameworks to slide copy:

| Formula | Use On | Structure |
|---------|--------|-----------|
| **PAS** (Problem-Agitate-Solution) | Problem slides | Pain point → Agitate → Solution |
| **AIDA** (Attention-Interest-Desire-Action) | CTAs, closing | Bold statement → Benefit → Social proof → CTA |
| **FAB** (Features-Advantages-Benefits) | Feature slides | Feature → Advantage → Benefit |
| **Before-After-Bridge** | Transformations, case studies | Before pain → After desire → Bridge |
| **Cost of Inaction** | Urgency, risk slides | Status quo → Loss → Time decay |

Formula-to-slide mapping:

| Slide Type | Primary Formula | Emotion |
|------------|-----------------|---------|
| Title / Hook | AIDA, Hook | curiosity |
| Problem | PAS, Agitate | frustration |
| Cost / Risk | Cost of Inaction | fear |
| Solution | FAB, BAB | hope |
| Features | FAB | confidence |
| Traction | Proof Stack | trust |
| Social Proof | Testimonial | trust |
| Pricing | Value Stack | confidence |
| CTA | AIDA, Urgency | urgency |

Headline patterns that work:
- "Stop [bad thing]" — problem framing
- "Get [desired result] in [timeframe]" — benefit promise
- "The [adjective] way to [action]" — differentiation
- "[Number] ways to [achieve goal]" — listicle
- "[Old way] is dead. Meet [new way]." — contrast

### Slide Strategies

Match the deck structure to the audience and goal:

| Strategy | Slides | Best For |
|----------|--------|----------|
| YC Seed Deck | 10-12 | Raising seed funding |
| Guy Kawasaki (10/20/30) | 10 | Pitching in 20 minutes |
| Series A | 12-15 | Raising Series A |
| Product Demo | 5-8 | Demonstrating product value |
| Sales Pitch | 7-10 | Closing qualified leads |
| Nancy Duarte Sparkline | Varies | Transforming perspective |
| Problem-Solution-Benefit | 3-5 | Quick persuasion |
| QBR | 10-15 | Updating stakeholders |
| Conference Talk | 15-25 | Thought leadership |
| Workshop | 20-40 | Teaching skills |

**YC Seed Deck structure (10 slides):**
1. Title / Hook
2. Problem
3. Solution
4. Traction
5. Market
6. Product
7. Business Model
8. Team
9. Financials
10. The Ask

**Emotion arc:** curiosity → frustration → hope → confidence → trust → urgency

**Nancy Duarte Sparkline Pattern:** Alternate between "What Is" (current pain) and "What Could Be" (better future), with pattern breaks at 1/3 and 2/3 to create engagement peaks.

**Strategy-to-context mapping:**

| Context | Recommended Strategy |
|---------|---------------------|
| Raising money | YC Seed, Series A, Guy Kawasaki |
| Selling product | Sales Pitch, Product Demo |
| Internal update | QBR, All-Hands, Board Meeting |
| Public speaking | Conference Talk, Workshop |
| Proving value | Case Study, Competitive Analysis |

## References (Knowledge Base)

The following reference files from the `slides` skill provide deeper dives into layout, copywriting, strategies, and templates. Refer to them when building data-driven or strategic decks:

| Topic | File |
|-------|------|
| Layout Patterns (25 layouts) | `skills/slides/references/layout-patterns.md` |
| HTML Template + Chart.js | `skills/slides/references/html-template.md` |
| Copywriting Formulas (25 formulas) | `skills/slides/references/copywriting-formulas.md` |
| Slide Strategies (15 strategies) | `skills/slides/references/slide-strategies.md` |

## Related Skills

- `frontend-patterns` for component and interaction patterns around the deck
- `liquid-glass-design` when a presentation intentionally borrows Apple glass aesthetics
- `e2e-testing` if you need automated browser verification for the final deck
- `design-system` for token architecture and component specifications used in strategic slides
- `ui-styling` for Tailwind/shadcn component styling in slide-adjacent UIs

## Deliverable Checklist

- presentation runs from a local file in a browser
- every slide fits the viewport without scrolling
- style is distinctive and intentional
- animation is meaningful, not noisy
- reduced motion is respected
- file paths and customization points are explained at handoff
- (if using Chart.js) charts are responsive, use dark-themed colors, and load without errors
- (if strategic) deck structure follows a strategy matching the audience and goal
