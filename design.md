# design.md — Aspire Studios Design System

> **Purpose:** Everything about how the site *looks and moves*. Read this instead of scanning `index.html` CSS.
> **Source of truth:** `index.html` (CSS lines 12–3043). Where this file and the HTML disagree, the HTML wins, except items marked **(fix)**, which are deliberate corrections that keep the look identical.
> **Read order:** `rules.md` → `architecture.md` → `design.md` → `tracker.md`.

---

## 1. Brand snapshot

| Item | Value |
|---|---|
| Site | Aspire Studios — dark/light one-page digital agency site |
| Personality | Dark, premium, tech-agency; soft glass, blueprint guide-lines, playful motion |
| Primary accent | `#4F46FF` (indigo-violet) |
| Typeface | Inter 300–700 (Google Fonts, `display=swap`) |
| Default theme | Dark (admin can change to Light / Follow system) |
| Content width | `.wrap` = max-width **1240px**, side padding **28px** |
| Section rhythm | `section { padding: 120px 0 }`, `position: relative` |
| Page order | Nav → Hero → Marquee → Services → Why → Stats → Founder → Team → Testimonials → FAQ → Contact+Map → Footer (+ floating contact button) |

---

## 2. Design tokens

### 2.1 Dark theme (default, `:root`)

| Token | Value | Use |
|---|---|---|
| `--bg` | `#0B0B0C` | Base background, input bg |
| `--page-bg` | `#0E0E0F` | `body` background |
| `--surface` | `#111113` | Panels |
| `--surface-2` | `#17171A` | Cards; Team + Testimonials section bg |
| `--text` | `#F5F5F5` | Primary text |
| `--muted` | `#A1A1AA` | Secondary text |
| `--subtle` | `#71717A` | Tertiary text |
| `--accent` | `#4F46FF` | Brand accent |
| `--border` | `rgba(255,255,255,.10)` | Hairlines |
| `--font` | `"Inter", system-ui, sans-serif` | All text |
| `--ease` | `cubic-bezier(.2,.7,.2,1)` | All custom easing |

### 2.2 Light theme (`:root[data-theme="light"]`, `color-scheme: light`)

| Token | Value |
|---|---|
| `--page-bg`, `--bg` | `#F4F6FA` |
| `--surface` | `#FFFFFF` |
| `--surface-2` | `#E9EDF4` |
| `--text` | `#191C26` |
| `--muted` | `#555D6D` |
| `--subtle` | `#737B8A` |
| `--border` | `rgba(25,28,38,.14)` |

The original has **73** `data-theme="light"` override rules (light-theme fix blocks start ~line 2650). All must be carried over, scoped as `:root[data-theme="light"] .as-…`.

### 2.3 Hard-coded colours that must become tokens **(fix)**

| Colour | Where | Light variant |
|---|---|---|
| `#17171A` | Card backgrounds | → `--surface-2` |
| `#0B0B0C` | Input backgrounds | → `--bg` |
| `#08080A` | Marquee band | `#E9EDF4` |
| `#262626` | Service floating description card | — |
| `#8B85FF` | Active service number | Derive from accent |
| `rgba(79,70,255,…)` | FAB glow/ping, `.ok` box, testimonial/FAQ badges, FAQ icon hover, button wipe | Replace with `color-mix(in srgb, var(--accent) N%, transparent)` |

### 2.4 Pill colour map (`.as-pill`)

| Variant | Background / square | Used for |
|---|---|---|
| default / `--blue` | `#BBE0FF` / `#1E9BFF` | Services ("Our services") |
| `--green` | `#C9F5D4` / `#22C55E` | Why |
| `--purple` | `#E4C9FF` / `#B66DFF` | Stats, Founder tag |
| `--pink` | `#FFC4F2` / `#FF4FD8` | Team |
| `--accent` | tinted accent | Testimonials, FAQs |
| (default) | blue | Contact ("Get in touch") |

Pill anatomy: inline-flex, gap 10, padding `8px 16px 8px 10px`, radius 12, text `#14141a` 14px/500, `margin-bottom: 24px`, soft shadow, small coloured square before the text.

### 2.5 Button tile colours (`.as-btn` icon tile)

| Class | Tile bg | Icon | Button |
|---|---|---|---|
| `as-tile-blue` | `#202B3D` | `#5B9BFF` | "Let's talk" (nav) |
| `as-tile-purple` | `#302345` | `#C084FC` | "Start a project" (hero) |
| `as-tile-green` | `#203A32` | `#55D6A3` | "View all services" |
| `as-tile-amber` | `#3D2D20` | `#FFB454` | "Send message" |

Social brand hover colours (via `data-network`): YouTube `#FF4040`, Instagram `#F56EA8`, LinkedIn `#3B9BFF`. Stat-card brand fills: YouTube `#FF0033`, Instagram `linear-gradient(45deg,#F9A03F,#E1306C,#833AB4)`, LinkedIn `#0A66C2`, X `#000`. Stars `#F4B400`. Error `#F87171`.

---

## 3. Typography

- Headings (h1–h3): weight **600**, letter-spacing **-.035em**, line-height **1.04**.
- Body: `400 16px/1.6`, `-webkit-font-smoothing: antialiased`.
- Fluid sizes (must stay in plugin CSS; Elementor Free cannot author `clamp()`):

| Element | Size |
|---|---|
| Hero h1 | `clamp(34px, 4.5vw, 68px)` |
| Section h2 | `clamp(34px, 4.6vw, 68px)` |
| Service list h3 | `clamp(22px, 2.5vw, 36px)`, weight 500 |
| Team quote | `clamp(28px, 3.4vw, 50px)` |
| Marquee words | `clamp(22px, 2.6vw, 34px)`, weight 500, `#D4D4D8` |
| Hero eyebrow | 12.5px, uppercase, `.14em` tracking |
| Mobile menu links | 44px / 600 |
| Footer wordmark | JS-fitted to container width (set 100px → measure → scale) |

- Admin may swap Heading/Body fonts; never force a font on Elementor widgets with high-specificity selectors.

---

## 4. Layout, spacing, breakpoints

**Breakpoints used (plugin CSS owns all layout; NOT Elementor's 1024/767):**
`1400` (back-to-top clears FAB) · `1100` (frame offset 40→20px) · `1000` (stats 4→2 cols) · `900` (main mobile switch: stacks, nav collapses, service card hidden) · `800` (FAQ stacks) · `600` (testimonials 2→1) · `560` (stats → 1 col). QA widths: **1440, 1100, 1000, 900, 800, 600, 560, 390** (acceptance also checks 1024, 768).

**Section spacing:**
- Original is *coupled*: Why `padding: 0 0 120px`, Stats `padding: 0`, Founder/Contact (`.fds`) `140px 0 120px`, Footer `70px 0 30px`, FAQ `120px 0` with top+bottom borders.
- **(fix)** New rule: every section gets symmetric `padding-block` = **half** the original gap each side. Single variable `--as-gap` (default **120px**). Removing a section must never collapse or double spacing.

**Section backgrounds:** Team and Testimonials both use `var(--surface-2)` with `border-block: 1px solid var(--border)`. FAQ has border top/bottom. Marquee has its own band colour.

**Z-index scale:** bgfx `-1` · section internals `1–5` · progress bar `100` (and nav `90`) · service floating card `300` · FAB `10000` (items 10001/10002).

**Two-column ratios:** Hero `1.1fr/.9fr` (gap 56) · Services 2 cols (gap 80) · Why `.9fr/1.1fr` (gap 80) · Founder `.85fr/1.15fr` (gap 80) · FAQ `.82fr/1.18fr` (gap `clamp(48px,9vw,132px)`) · Contact centred max-width 820.

---

## 5. Component library

### 5.1 Button `.as-btn` (built on Elementor Button widget)
- Height **52px** (46 nav, 48 form), radius 14, bg `#F5F5F5`, text `#0B0B0C`, 15px/600.
- Square **icon tile** at right (height − 12px, radius 10).
- Hover: accent colour wipes in from right (`clip-path: inset(0 0 0 100%) → inset(0)`, .6s); label slides right; tile slides left and spins 360°.
- `as-btn--alt`: transparent with inset 1px border.
- LinkedIn `.as-lin`: blue with moving shine sweep (3.6s) and blue glow on hover.
- Text link `.as-link` ("Why Aspire →" etc.): underline grows from left on hover, arrow shifts 6px.
- Light theme: dark fill / light text variant (original fix block at ~2652).

### 5.2 Blueprint frame `.as-frame`
Dashed guide lines extending **40px** beyond a rounded box (**20px** under 1100px), 14px corner arcs, layered gradients on `::before` (keep **both** `mask-composite` syntaxes). Colour `rgba(255,255,255,.34)` + light variant. Used on: Why image, Founder image, Stats grid, Map.

### 5.3 Image placeholder `.as-ph`
Radius 18 (cards 10–12), `overflow:hidden`, subtle radial overlay (`mix-blend-mode:overlay`), gradient fallback `linear-gradient(135deg,#1c1c22,#0f0f12)`. **(fix)** Use real `<img object-fit:cover>`, not `--img:url()`.

### 5.4 Decorative grid `.as-grid-bg`
26px grid + scattered filled squares, masked. Opt-in class on Founder, Contact, Footer; each self-contained. (Original `.fds`.)

### 5.5 Reveal `.as-rv`
`opacity:0; translateY(34px)` → `.in`: `transition: opacity .9s, transform .9s var(--ease)`. Stagger classes `as-d1…as-d6` (.08s or .1s steps). Visible when JS off, in Elementor editor, or after 3s failsafe.

### 5.6 Focus & misc
`:focus-visible` 2px accent outline, 3px offset. `scroll-behavior:smooth`, `scroll-margin-top` on anchors. `body{overflow-x:hidden}`.

---

## 6. Global effects

| Effect | Spec |
|---|---|
| **Ambient bg** `.as-bgfx` | Fixed, z -1, no pointer events. 3 blurred (36px) rotated gradient ellipses, `drift` 28s alternate, delays `0 / -9s / -17s`; SVG feTurbulence noise at `.07` (file, not data-URI). Light theme blobs `.25` opacity. |
| **Progress bar** | Fixed 2px accent bar, width = scroll %, z 100. Offset by admin bar. |
| **Cursor ring** | Hover-capable + no reduced-motion only. 34px white ring, `mix-blend-mode:difference`, lerp `.18`. Grows to 64px + fills white over links, buttons, `.sv li`, team cards, service cards, hero cards. **Excludes** `.as-btn`, FAB items, stat cards, FAB main, back-to-top. Native cursor stays. |
| **Magnetic buttons** | `.as-mag` translate `0.25×` x / `0.35×` y of pointer offset from centre; reset on leave. |
| **Floating "Figma" cursors** | 3 labelled arrow+pill cursors in hero. Originals: Meezan `#4F46FF`, Ikram `#F24E8B`, Sadiq `#0FA958`. New random target every 1.8–4.0s, lerp `.025`; **cursor #2 follows real mouse** (+46,+34 offset, lerp `.09`). |
| **Reduced motion** | All animation/transition off; `.as-rv` and hero text visible; `scroll-behavior:auto`; autoplay and JS loops disabled. |

### Keyframes in source
`bob` (scroll-down arrow) · `drift` (blobs) · `fping` (FAB ping 2.6s) · `pop` · `rise` (hero deck entrance, fade+60px) · `shine` (LinkedIn btn) · `slide`/`slide2` (`slide2` belongs to dead `.cred` strip, do not port) · `up`.

---

## 7. Section-by-section visual spec

### Nav (global, plugin-rendered)
Fixed floating pill: top 16, max-width 1120, height 64, radius 18, glass `rgba(20,20,23,.6)` + `backdrop-filter: blur(18px)`, border + shadow; `.stuck` (darker) after 30px scroll; hides on scroll-down after 400px (`translateY(-140%)`), reappears on scroll-up. Logo 23×20 + name (hover: rotate 10°, scale .95). Centre links absolutely centred, underline scales in from left on hover/active. Right: "Let's talk" CTA (blue tile, `fa-comment-dots`), theme toggle (inline SVG sun/moon 20px, stroke 1.8, lifts 2px and turns accent on hover), burger ("Menu"/"Close").
**Mobile ≤900px:** CTA + centre links hidden; full-screen overlay (`100dvh`, `var(--bg)`, links 44px/600, padding 36px).

### Hero `#top`
`min-height:100svh` (fallback 100vh), padding `130px 0 80px`. Left: eyebrow, **H1 with line-mask reveal** (each line `translateY(105%)→0`, 1s, second line +.12s), lead, 2 CTAs (purple-tile button w/ rocket + "Why Aspire →"), "Scroll down" circle with bobbing arrow, social icons, contact strip (email / phone / "Location ↗", top-border separators). Right: **card deck** (`aspect-ratio 500/560`, max 500; cards at left 8%/top 8%/width 72%/height 84%, radius 10).
Deck poses `[x px, y px, rot°]` for depth 0–5: `[0,0,-2] [44,-16,6] [-30,18,-7] [22,34,3] [-48,-14,-11] [60,22,10]`; `z = N−depth`; `brightness(1 − depth×.1)`; fly-out `translate(115%,-6%) rotate(14deg)`, 550ms swap, autoplay 3800ms, click shuffles.

### Marquee `#marquee`
Band: bg `#08080A` (light `#E9EDF4`), border-block, padding 26px 0, 28px side padding per item, accent "✦" separators, `aria-hidden`. Loop leftward, pauses on hover. Original 38s fixed → **(fix)** constant px/sec speed.

### Services `#services`
Left: blue pill, H2 "We build your / digital business.", lead, list (h3 muted; active = white, 2px growing underline, number `#8B85FF`), green-tile "View all services" (`--alt`). Right: stacked image cards (`min-height:540`, 360 mobile). Card poses `[0,0,-2] [34,-10,5] [-30,14,-6] [54,-26,9]`, `brightness(1 − d×.16)`, transition .8s. Desktop (>900): floating description card (`#262626`, 340px wide, radius 6) slides to hovered row, hides 180ms after leave. ≤900: inline paragraph on active item. Secondary image (SEO only in original): 36% width, 4px white border, bottom-right.

### Why `#why`
Left: `.as-frame` + image (4:5; 4:4.2 mobile) + glass chip bottom-left ("1:1" / "Work directly with the founder", blur 14). Right: green pill, H2, lead, 4-item icon list (46px circular accent icon, bold title, muted text, dashed top borders, dashed bottom on last, staggered reveal), "Start your project →".

### Stats `#stats`
Purple pill, H2 "A community that / grows with us.", lead right. Grid of 4 in `.as-frame` (→2 cols ≤1000, 1 col ≤560). Card: `#17171A`, 1px border, **46px clipped top-right corner** + folded triangle, faint masked blueprint grid top-right, 54px white icon tile (`box-shadow 0 0 0 6px`). Hover: lift 4px, lighter bg, tile takes brand colour (`--b`) and tilts −8°, arrow fades in bottom-right rotated −45°. Counter 1600ms ease-out-quartic.

### Founder `#about`
`.as-grid-bg`; `.85fr/1.15fr`; framed square image (gradient fallback); lilac "Founder" tag; H2 "The Founder's Desk"; two paragraphs (name **Mohammed Rehan** bolder/brighter); LinkedIn button + social icons. Padding via gap rule.

### Team `#team`
`--surface-2` band. Left: pink pill, quote-icon circle (fa-quote-right), big quote (heading-styled `<p>`/blockquote), short rule, bold name, small role. Right: stacked portrait cards (72%×86%, radius 12, shadow), round 48px prev/next arrows overlapping edges (±24px; ±6px ≤900). Poses `[0,0,-2] [38,-12,5] [-30,14,-6]`, `brightness(1 − d×.2)`, front card `.front`. Quote swap: fade out, 260ms swap, `translateY(12px)`. Hover front card → social pill slides up. Autoplay 7000ms.

### Testimonials `#testimonials`
`--surface-2` band. Accent pill, H2 "People love us, you know.", lead; round 48px arrows (scale 1.08 hover) in header. Cards: min-height 272, radius 8, padding 24/22; gold stars, grey Google icon top-right, quote with CSS curly quotes, initials circle + name + role. 3 visible ≥901, 2 ≤900, 1 ≤600 (gap 12). FLIP slide 520ms.

### FAQ `#faq`
Left sticky at 120px: accent pill, H2 "Good to know before we begin." (max 9ch), lead, "Still have a question? Get in touch →". Right: `<details>` items; 34px circular +/− icon (vertical bar rotates away when open; border/bg go accent on hover/open).

### Contact + Map `#contact`
`.as-grid-bg`; centred, max-width 820; pill "Get in touch", H2 "Let's start something / worth building."; form with **floating labels**, amber "Send message" button + social icons right; success box `.ok` (accent border, tinted bg, radius 14). Map: `.as-frame`, height `clamp(260px,34vw,380px)`, radius 12, dark-mode filter `invert(92%) hue-rotate(180deg) saturate(.75) contrast(.95)` (none in light), glass "Open in Google Maps" chip bottom-left, click-to-interact.
Error state: border `#F87171` + red `<small>` message.

### Footer
`.as-grid-bg`, padding `70px 0 30px`, border-top. 4 columns: Brand (logo, tagline, social) / Navigate / Services / Contact. Giant auto-fit wordmark. Bottom bar: © {year}, "Made with care in India", round back-to-top (70px right margin <1400px to clear FAB).

### Floating contact button (FAB)
Fixed bottom-right 22px, z 10000. Main 60px, radius 14, accent + glow + ping ring (`fping` 2.6s). Opens a vertical fan: Location, Phone, Email (glass label pill + 52px round icon, staggered fade/scale). Main turns white when open.

---

## 8. Source content (seed copy — reproduce exactly)

Counts: 6 marquee words · 4 services · 6 hero cards · 4 why points · 4 stats · 3 team · 8 testimonials · 5 FAQs · 6 nav links · 3 FAB items · 4 form fields.

**Contact details:** `marketing@aspiredigitalsolutions.in` · `+91 79753 27335` (`tel:+917975327335`) · 3/D, 7th Cross Rd, Subhasnagara, Rajendra Nagar, Mysuru, Karnataka 570007.

**Nav:** Home `#top`, Services `#services`, About `#about`, Testimonials `#testimonials`, FAQs `#faq`, Contact `#contact`. CTA "Let's talk" → `#contact`.

**Hero:** eyebrow "Aspire creative studio"; H1 "Grow your business / with a new website."; lead "Aspire is a full-service creative studio creating beautiful digital experiences and products."

**Marquee:** Web development · SEO · Branding · Digital marketing · UI/UX · Content.

**Services** (title / blurb): 01 Web design & development — "Fast, responsive websites on WordPress, React and Next.js, built to convert visitors into customers." · 02 SEO & digital marketing — "Search, ads and analytics that bring the right customers to your door and show what is working." · 03 Branding & identity — "Logos, type and visual systems that make your business consistent and easy to remember." · 04 Social media — "Content and campaigns that build a loyal community around your brand."

**Why points:** Founder-led (`fa-user-tie`) · Built to convert (`fa-bullseye`) · Fast and search-ready (`fa-gauge-high`) · Clear communication (`fa-comments`). H2 "Built around your business, not a template."

**Stats:** YouTube 20K+ · Instagram 30K+ · LinkedIn 5K+ · X 3K+ (descriptions in `index.html` ~3264–3308).

**Team:** Mohammed Sadiq (UI/UX Designer) "We are a strong team who bring innovative ideas into production." · Mohammed Ikram (WordPress Developer) "They understood our brief on day one and delivered a site we are proud of." · Mohammed Meezan (SEO Expert) "Our enquiries doubled within two months of launching the new website."

**Testimonials (8):** Mohammed Imran (CEO, Construction Co.) · Priya Sharma (Owner, Fashion Boutique) · Rajesh Kumar (Director, Tech Solutions) · Anita Reddy (Founder, Organic Foods) · Deepa Hegde (Director, EduAcademy) · Kavitha Murthy (Manager, Restaurant Chain) · Suresh Rao (Owner, Fitness Studio) · Anand Patil (Realtor, Mysore). Full quote text: `index.html` lines 3741–3752 (`const T`).

**FAQs (5):** What can Aspire Studios help me with? · How does a project get started? · How long does a website take to build? · How much does a project cost? · Can you help after the website launches? (answers: `index.html` 3406–3450).

**Founder:** "Hi, I'm **Mohammed Rehan**. I started Aspire Studios to help ambitious businesses look as good online as they do in person." / "I work directly on every project, from the first sketch on paper to the last line of code, and share what I learn on YouTube and Instagram."

**Success message:** "Thanks! We've got your message and will get back to you soon."

**Footer tagline:** "We build digital experiences for ambitious businesses." Bottom: "© 2026 Aspire Studios. All rights reserved." / "Made with care in India".

---

## 9. Asset inventory

| File | Used for | Note |
|---|---|---|
| `logo.svg` | Nav + footer logo | WP blocks SVG → Safe SVG plugin or PNG/WebP fallback |
| `hero-one … hero-six.jpg` | 6 hero deck cards | First card `fetchpriority="high"` |
| `why-aspire.jpg` | Why image | |
| `founder.jpg` | Founder image | |
| `sadiq.ico` | Team portrait | **Convert to JPG/WebP** (`.ico` can't be resized) |
| `ikram.png`, `meezan.png` | Team portraits | |
| `google-ads.jpg` | Secondary service image (SEO) | |
| `web-development.jpg`, `seo.jpg`, `brand-identity.jpg`, `media-marketing.jpg` | Service cards | **Referenced only from JS** — easy to miss |

⚠ The `assets/` folder was **not** included in the analysis upload. Existence of these files is unverified.

---

## 10. Do-not-port list (dead code in source)
`.cred` strip + `slide2` keyframes · `.fd .row` · `.contact-fab.away/.in` (never toggled) · `.fa-moon/.fa-sun` rules (icons are inline SVG `.ti-*`) · `data-depth` on hero cards (never read) · nav `.tb+.theme-toggle` rules declared 3× (consolidate once).
