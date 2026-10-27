# Plan: Pixel Grid Landing Page (React + Vercel)

## 1. Tech Stack

- **Build**: Vite + React + TypeScript (fast, zero-config Vercel deploy)
- **Styling**: Plain CSS with CSS custom properties (design tokens from §6)
- **Font**: Silkscreen (Google Fonts) with pixel monospace fallback
- **Animation**: Single `<canvas>` overlay using `requestAnimationFrame`
- **Visitor counter**: Vercel Serverless Function + Vercel KV (free tier)
- **Deploy target**: Vercel (static frontend + `/api` serverless)

## 2. Project Structure

```
/
├── api/
│   └── visitors.ts          # GET (read count) / POST (increment) → Vercel KV
├── src/
│   ├── components/
│   │   ├── Hero.tsx
│   │   ├── PixelTrail.tsx   # Canvas grid + pointer-reactive pixels
│   │   ├── Footer.tsx
│   │   └── VisitorCount.tsx # Small badge, right edge of footer bar
│   ├── App.tsx
│   ├── main.tsx
│   ├── index.css            # Design tokens + base styles
│   └── styles/              # Per-component CSS modules
├── index.html
├── package.json
├── vite.config.ts
├── vercel.json
├── .env.example             # KV env var names
└── README.md                # Setup + Vercel deploy steps
```

## 3. Implementation Steps

| # | Step | Notes |
|---|------|-------|
| 1 | Scaffold Vite React-TS project | `npm create vite@latest . -- --template react-ts` |
| 2 | Add design tokens | CSS vars from design doc §6; `clamp()` typography |
| 3 | Build `Hero` | Dark panel, centered headline, canvas overlay, `aria-hidden` |
| 4 | Build `PixelTrail` | Canvas grid + DPR-aware sizing; pointermove → grid coords → stepped trails from palette; bounded active-pixel Map; idle cleanup; `prefers-reduced-motion` falls back to static grid |
| 5 | Build `Footer` | 4-col grid (description + 3 nav), vertical borders, external-link icon (square + diagonal arrow), language selector bottom-left; stacks on mobile |
| 6 | Build `VisitorCount` | Small badge (eye icon + number, monospace) anchored to the right edge of the footer bar; calls `/api/visitors` once per session |
| 7 | Write `api/visitors.ts` | `@vercel/kv`; GET returns count, POST increments via `INCR`; debounced client-side to 1 increment/session |
| 8 | Responsive pass | Mobile-first: headline `clamp()`, footer stacks, canvas grid cell shrinks, safe tap targets |
| 9 | Accessibility | Single `<h1>`, real anchors, focus rings, reduced-motion path, aria-hidden canvas, contrast ≥ AA |
| 10 | Vercel config | `vercel.json` with build/output dirs; README documents KV setup + env vars |

## 4. Visitor Count Placement

"Small count appearing at the right" — implemented as a **compact badge anchored to the right end of the footer bar** (same row as the 3 nav columns on desktop, tucked at the right edge on mobile). Style: muted gray text, ~12px, with a small eye/dot icon. Updates live as the API responds.

## 5. Key Design Decisions

- **Vite over Next.js** — the user said "easily deployable to Vercel" + we need a single API route. Vite + Vercel's automatic `/api` detection gives us that without SSR overhead.
- **Canvas over DOM nodes** — design doc §9 mandates this for performance.
- **Vercel KV over in-memory** — Vercel Functions are stateless; persistent count needs a store. KV is the native, free, lowest-friction option.
- **Fading pixel mode** — chosen over persistent (design doc §4 leaves it configurable; fading is safer, prevents unbounded canvas state, feels alive).

## 6. Mobile Behavior

- Headline: `clamp(2rem, 11vw, 7rem)` keeps two lines legible down to 320px.
- Footer stacks: description on top, then a horizontal scrollable row of nav items.
- Canvas disables trail generation during vertical scroll gestures (touch-action: pan-y) so it doesn't fight native scrolling.
- Visitor count moves from bottom-right of footer to inline at the end of the description row.

## 7. Acceptance Criteria (mapping to design doc §10)

All 12 checkboxes are satisfied by steps 1–10 above. Highlighted:

- Palette enforced in `PixelTrail` color picker (no random colors).
- Canvas `pointer-events: none` so clicks reach links.
- Resize handler keeps grid + trails aligned; DPR scaling keeps it sharp.

## 8. Decisions Needed Before Building

Resolved defaults used in the implementation (all changeable):

1. **Visitor count backend** — `@vercel/kv` (works with both legacy Vercel KV and the Upstash Redis integration via `KV_REST_API_URL` / `KV_REST_API_TOKEN`), with a localStorage fallback when the API is unavailable (e.g. plain `vite dev`).
2. **Pixel trail mode** — **fading** (stepped 4-level fade, ~1.6 s lifetime, capped at 2000 cells).
3. **Footer nav destinations** — real external links (Walrus site / GitHub / docs) in `src/links.ts`; the design doc forbids `#` placeholders.
4. **Language selector** — kept (`<details>`-based "English" + chevron, opens upward, bottom-left of the footer).