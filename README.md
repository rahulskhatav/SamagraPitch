# Redirection, Not New Money — Scroll-Zoom Pitch Page

Build a single-file, scroll-driven presentation webpage for a governance case competition submission (Samagra, The Governance Challenge 2026 — Government of Maharashtra, Education sector).

The page is a **pitch deck rendered as a scroll experience**. Each section is a full-screen "slide" that the viewer zooms *into* as they scroll — like PowerPoint's Zoom transition. It is not a conventional marketing page.

---

## 1. Deliverable & constraints

- **One file:** `index.html`. All CSS and JS inline. No build step, no bundler, no framework.
- **No external dependencies** except Google Fonts (`Inter`, weights 300/400/500/600).
- **Desktop-first.** Design for 1440×900. Set `min-width: 1180px` on the root wrapper and allow horizontal scroll below that rather than reflowing. This will be presented on a laptop/projector, not a phone.
- **Must work offline** once fonts are cached, and must open correctly from `file://`.
- **No localStorage / sessionStorage.**
- Total page target: under 150 KB of hand-written source.

---

## 2. Design system

Set these as CSS custom properties on `:root` and use them everywhere. Do not introduce colours outside this palette.

```
--bg:        #141c2b   /* deep navy, page background */
--ink:       #ffeedb   /* warm cream, primary text */
--gold:      #f2c14a   /* primary accent — headline emphasis, the "ask" */
--gold-soft: #f7d777   /* eyebrow labels, hairlines */
--green:     #34a853   /* chain: working links */
--green-dim: #2b8c47   /* chain: connector strokes */
--green-txt: #c9f2d3
--red:       #e0342c   /* chain: broken links, gaps */
--red-soft:  #ff8a7a
--violet:    #7c5cff
--blue:      #3b86e8
--pink:      #e8579e
--panel:     #1e2a3f   /* card / table cell surface */
```

Derived text tints: `rgba(255,238,219,.72)` for body, `.62` for captions, `.42` for meta.

**Typography** — `Inter`, `system-ui`, `sans-serif` throughout.

| Role | Spec |
|---|---|
| H1 (scene 01 only) | `clamp(64px, 7.4vw, 118px)` / line-height `.92` / weight 500 / letter-spacing `-.035em` |
| H2 (scene headline) | `clamp(38px, 3.7vw, 56px)` / line-height `1.06` / weight 500 / letter-spacing `-.025em` |
| H3 (pillar title) | `26px` / weight 500 |
| Eyebrow | `13px` / `letter-spacing: .24em` / `text-transform: uppercase` |
| Lead paragraph | `26px` / weight 300 / line-height `1.45` |
| Body | `19px` / weight 300 / line-height `1.6` |
| Caption | `14px` / line-height `1.45` |
| Big stat | `38px` / weight 500 / letter-spacing `-.025em` |

Max text measure: `46ch` for lead, `56ch` for body. Never full-bleed paragraphs.

**Layout:** each scene's inner container uses `padding: 0 6vw 0 9vw` — deliberately asymmetric, content hangs left.

**Eyebrow pattern:** a 56px gradient hairline (`transparent → --gold-soft`) followed by uppercase label, gap 14px. Use at the top of most scenes; vary the label colour per scene (gold, green, blue, violet) to give each slide its own key.

---

## 3. The scroll-zoom mechanic (most important part)

This is the defining interaction. Get it right before anything else.

**Structure per scene:**

```html
<section data-scene data-screen-label="01 Apex" style="height:200vh;position:relative">
  <div data-scene-inner style="position:sticky;top:0;height:100vh;
       display:flex;flex-direction:column;justify-content:center;
       padding:0 6vw 0 9vw;will-change:transform,opacity;overflow:hidden">
    <div data-glow></div>   <!-- radial-gradient wash, pointer-events:none -->
    <!-- scene content -->
  </div>
</section>
```

**Behaviour:** on scroll, compute each scene's progress `t` from 0→1 across its scroll span, then transform the sticky inner:

- `t < 0.30` (entering, skip for the first scene): ease-out-cubic from `scale(0.78)` → `scale(1)`, opacity `0 → 1`, blur `5px → 0`.
- `0.30 ≤ t ≤ 0.70`: fully settled — `scale(1)`, opacity `1`, no blur. This is the readable dwell window.
- `t > 0.70` (exiting, skip for the last scene): ease-in from `scale(1)` → `scale(1.34)`, opacity `1 → 0`, blur `0 → 7px`.

The result: the viewer flies *into* each slide, holds, then flies *through* it into the next. Content grows past the viewport as it leaves rather than sliding away.

**Implementation notes:**

- Single `scroll` listener, `{passive:true}`, throttled with `requestAnimationFrame` and a `pending` flag. One listener for all scenes — do not attach per-section observers.
- Also recompute on `resize`.
- Set `pointer-events: none` on any inner whose opacity ≤ 0.7 so faded slides don't swallow clicks.
- **`prefers-reduced-motion: reduce` → disable entirely:** no scale, no blur, opacity locked at 1. The page must remain fully readable as a plain vertical document.
- Scene heights: `200vh` for simple scenes, `240vh` for dense ones (04, 05, 06) to lengthen the dwell.

**Glow layer:** each scene gets an absolutely positioned `[data-glow]` with two large radial gradients at low alpha (0.14–0.26), positioned differently per scene, using that scene's key colour. This is what stops the dark background feeling flat.

---

## 4. Scene-by-scene content

Use this copy **verbatim**. The figures are sourced and must not be paraphrased, rounded, or invented.

### Scene 01 — Apex

- Eyebrow: `Maharashtra · deep tech in education` (gold)
- H1: `Redirection,` / line break / `not new money.` — second line in `--gold`
- Lead: `A regulated sandbox for testing. A signal channel for feedback. Procurement that pays only for proof.`
- Footer cue: `SCROLL` in meta style with a trailing gradient hairline.
- Glow: gold at 88% 26%, violet at 8% 86%.

### Scene 02 — The broken chain

- Eyebrow: `Six stages of deep tech development` (green)
- H2: `The chain holds at both ends. It snaps in the middle.`
- **Chain diagram**, horizontal, single row, `flex-wrap: nowrap`:
  `Conceived → Researched → Tested → Funded → Adopted → Scaled`
  - Working nodes (Conceived, Researched, Adopted, Scaled): 16px circle, 2px `--green` border, `rgba(52,168,83,.28)` fill, label in `--green-txt`.
  - Broken nodes (**Tested**, **Funded**): 22px circle, 2px `--red-soft` border, solid `--red` fill, `box-shadow: 0 0 0 8px rgba(224,52,44,.2)` halo, label in `--red-soft` weight 500.
  - Connectors are `flex:1` 3px bars: solid green between working pairs; green→red and red→green gradients on either side of the break; and **between Tested and Funded use a dashed stroke** — `repeating-linear-gradient(90deg, #e0342c 0 5px, transparent 5px 13px)` — to read as severed.
- Body: `An idea can be conceived and researched here, and it can scale here — but nothing crosses from a working prototype to a funded, procured product. Two links carry the whole failure.`

### Scene 03 — What the state already owns

- Eyebrow: `What the state already owns` (blue)
- Five-column grid, 1px gaps over a `rgba(255,238,219,.13)` background so the gaps read as hairline rules. Each cell: a 3px coloured top bar, then stat + caption.

| Stat | Caption | Top bar |
|---|---|---|
| `1.45 cr` | school students · 87 universities · 5,579 colleges | blue |
| `7.5 lakh` | teachers already inside AI Saathi training | violet |
| `₹10,000 cr` | AI Policy 2026 · 2,000 GPUs · 6 Centres of Excellence | pink |
| `ATL` | Atal Tinkering Lab network — a physical testbed in place | green |
| `₹524 cr` | a year · recurring · cabinet-approved · currently unclaimed (PRS 2026-27) | gold |

The fifth cell is the punchline — give it a `rgba(217,160,31,.16)` tinted background instead of `--panel`.

- Closing H2 below the grid: `The gap is wiring, not money.`

### Scene 04 — Five systemic gaps, scored (height 240vh)

- Eyebrow: `Five systemic gaps, scored` (red)
- Sub-line: `Impact (blocking power · lifecycle coverage · reach) × feasibility (legal · fiscal · institutional readiness · time to proof)`
- Table, columns: `# | Gap | Why it matters | Score`. Rows in descending score order. Header row in caption style, uppercase, dim.

| # | Gap | Why it matters | Score |
|---|---|---|---|
| S1 | No lawful route to process minors' learning data | Adaptive learning is behavioural monitoring under DPDP. Every AI pilot is non-compliant until the state builds a pathway. Gates all others. | 21.3 |
| S2 | The state runs blind — no signal in, nothing measured, nothing verifiable | No teacher feedback loop, no evaluation on testbeds, no credential portability. Nothing can be validated, so nothing can be procured on evidence. | 21.0 |
| S5 | Four departments own one stage each; nobody owns the pipeline | The reason the other four stay open. A failure spanning two departments is visible to somebody, actionable by nobody. | 16.3 |
| S3 | Research is neither pointed at education nor spread beyond a few institutions | No education vertical in any funding instrument; ~2.5% of colleges run PhDs. Slows what gets built; blocks nothing in flight. | 10.8 |
| S4 | Nothing crosses from a working idea to a paying customer | No demand channel to buyers, no IP/licensing framework, no capital bridging validation. The narrowest reach of the five. | 10.0 |

- **Visually elevate S1 and S2**: tinted row background, red left border, score in `--red-soft` at larger size. S3–S5 render dimmer (`opacity: .68`) so the eye lands on the top two.
- Closing callout block: heading `S1 and S2 fail together.` then `They are the permission and the machinery. Permission without machinery is exactly what South Korea had; machinery without permission is illegal.`

### Scene 05 — Gate 1: MERS (height 240vh)

- Eyebrow row: `Resolves S1` (as a small red pill/badge) + `Gate 1 · the legal gate`
- H2: `Maharashtra EdTech Regulatory Sandbox`
- Lead: `A lawful pathway to process minors' learning data — co-managed by SCERT, the School Education Department and MSInS.`
- **Three pillars side by side** (equal-width columns, 1px hairline dividers between). Each pillar: `PILLAR 0N` eyebrow → H3 → one-line summary → an expand affordance → detail items beneath.

**Pillar 01 — Selection Mechanism**
Summary: `A 3-phase lifecycle that keeps fly-by-night operators out of school access.` · Toggle label: `Three phases ↓`
- *Phase 1 · admission & design review* — DPIIT-registered, an educational MVP (XR, blockchain or adaptive systems), a DPDPA threat assessment, and SCERT pedagogical review against state curriculum standards.
- *Phase 2 · beta piloting* — Access to a designated cluster of Certified Pilot Schools, observed by SCERT officers under limited-month administrative safe harbours.
- *Phase 3 · graduation & scaling* — Third-party audit of data-erasure compliance, technical security and learning-outcome efficacy before exit.

**Pillar 02 — Operational Requirements**
Summary: `What a startup must run continuously while inside the sandbox.` · Toggle label: `Three duties ↓`
- *Comprehensive data mapping* — Every field collected, its purpose, its retention and its route out of the system, declared up front.
- *Continuous threat modelling* — Maintained against live telemetry logs, not filed once as an admission document.
- *Non-adaptive static fallback* — Every deep-tech platform keeps a fully functional non-adaptive version of its software available.

**Pillar 03 — Technical Governance**
Summary: `Consent that is provable, and telemetry that carries no identity.` · Toggle label: `Two ledgers ↓`
- *Immutable cryptographic consent* — A private, permissioned state ledger. Every grant, change or withdrawal made through Aaple Sarkar is cryptographically logged.
  - *Granular opt-in · s.6(3)* — Per-activity toggles in Marathi and English.
  - *Mitigating s.6(5)* — Withdraw consent and the student reverts to the standard digital version — no loss of basic pedagogical access.
- *Pseudonymized telemetry ledger* — VR gaze tracking, haptic latency and adaptive progression are stripped of identity, hashed, and processed as pseudonymized transactions.

### Scene 06 — Gate 2: Evidence (height 240vh)

- Eyebrow row: `Resolves S2` badge + `Gate 2 · evidence, market & procurement`
- H2: `Evidence-driven translation & scaling`
- Lead: `Signal in from classrooms, proof out through procurement — so public money buys only what demonstrably works.`
- Same three-pillar layout as scene 05.

**Pillar 01 — Demand Sourcing**
Summary: `7.5 lakh teachers become innovation sensors inside an app the state already ships.` · Toggle: `AI Saathi return channel ↓`
- *AI Saathi return channel* — A feedback mechanism inside the notified AI Saathi assistant — no new app, no new rollout.
- *Friction-to-problem mapping* — Classroom challenges captured in real time become a public database of ~15,000 verified problem statements a year, at a conservative 2% response rate.
- *Focused R&D pipeline* — MSInS-registered startups build against verified friction points — immediate classroom utility, a pre-qualified customer base.

**Pillar 02 — Evaluation Testbed**
Summary: `A double-sided quality gate on the ATL network, modelled on the UK's DfE and EEF.` · Toggle: `Two-stage efficacy gate ↓`
- *ATL network integration* — The testbed layers onto existing Atal Tinkering Lab schools as real-world testing grounds.
- *Stage 1 · safety & compliance* — Vetted inside MERS for DPDP compliance, algorithmic fairness and baseline technical safety.
- *Stage 2 · pedagogical impact* — A structured pilot measures retention and conceptual mastery against a control group.
- *VSK integration* — Verified efficacy feeds the notified Vidya Samiksha Kendra, gating state procurement on empirical proof of learning outcomes.

**Pillar 03 — Credential Portability**
Summary: `One graduation, recognised everywhere — no repeating the pilot in every district.` · Toggle: `APAAR · ABC · EU standard ↓`
- *APAAR / ABC portability* — Testbed graduations link directly to the central APAAR registry and Academic Bank of Credits.
- *EU digital credentials standard* — Tamper-proof micro-credentials issued to graduating startups, styled on the European framework.
- *Frictionless procurement* — The credential becomes a legally recognised quality stamp — a highway past repeat pilot trials in other states and municipalities.

### Scene 07 — Ownership

- Eyebrow: `Ownership`
- H2: `One council,` / line break / `already in statute.`
- Body: `The gates need a keeper. Rather than build a new body, reconstitute the ` **Rajiv Gandhi Science & Technology Commission** ` — Maharashtra Act XV of 2004 — to own the pipeline end to end.` (bold the commission name in `--gold`)
- **Re-render the six-stage chain from scene 02, now fully healed** — all six nodes green, all connectors solid green, and a bracket or spanning rule beneath all six labelled with the commission. This visual payoff is the point of the scene.
- Closing line: `One keeper, six stages, no new statute — and the chain closes.`

### Scene 08 — The core ask

- Eyebrow: `The ask`
- H2: `Two moves.` / line break / `No new budget line.`
- Two cards side by side:
  - `MOVE ONE` → **A state notification under Rule 10** → `Opens the sandbox. Executive action, no legislation.`
  - `MOVE TWO` → **A module inside a course the state already runs** → `Opens the signal channel through AI Saathi.`
- Three-up stat strip beneath:
  - `₹524 cr` — approved, recurring, unclaimed
  - `36 districts` — compute, testbeds and teachers already there
  - `S1 + S2` — need only executive action
- Final line, large, gold, centred or hanging left: `Redirection, not new money.`

---

## 5. Component patterns

**Expandable pillar detail.** Each pillar's detail list is collapsed by default behind the toggle label (`Three phases ↓` etc.). Clicking the label expands that pillar only. Implement with a single state object keyed `p1`–`p6`; toggling one must not close others. Rotate the `↓` to `↑` when open. Animate with a `max-height` + `opacity` transition, ~240ms ease.

Provide a URL flag `?expand=1` that opens all pillars at once — needed for screenshots and PDF export of the submission.

**Badges.** `Resolves S1` / `Resolves S2` render as small pills: 11px uppercase, `.18em` tracking, 1px border in the scene key colour, transparent fill, 4px radius.

**Cards / panels.** `--panel` background, 8px radius, 1px `rgba(255,238,219,.13)` border, 26px 22px padding, optional 3px coloured top bar.

**Hairline dividers.** `rgba(255,238,219,.13)`, 1px. Between pillars use vertical rules, not margins.

---

## 6. Accessibility & robustness

- Every scene keeps semantic structure (`section`, one `h1`, then `h2`/`h3`). The scroll effect is presentation only — with JS disabled the page must still read top to bottom as a legible document.
- Honour `prefers-reduced-motion: reduce` as specified in §3.
- Toggles must be real `<button>` elements with `aria-expanded`, keyboard-operable, visible focus ring in `--gold`.
- Colour is never the only carrier of meaning: broken chain nodes are larger *and* red *and* labelled in a distinct weight.
- Minimum body contrast: cream on navy at ≥ 4.5:1. Do not drop body text below `rgba(255,238,219,.62)`.

---

## 7. Build order

1. Skeleton: 8 `data-scene` sections with correct heights and sticky inners.
2. Design tokens and typography.
3. The scroll-zoom engine (§3) — verify the dwell window feels right before adding content.
4. Scenes 01, 02, 03 with real copy.
5. The chain component, reused in 02 (broken) and 07 (healed) from one function with a `state` argument.
6. Scenes 04–08.
7. Pillar expand/collapse + `?expand=1`.
8. Reduced-motion pass and keyboard pass.

---

## 8. Acceptance checklist

- [ ] Scrolling produces a distinct zoom-into / zoom-through transition per scene, not a fade or a slide.
- [ ] Each scene has a settled, fully readable dwell window mid-scroll.
- [ ] Scene 02 chain visibly breaks at **Tested** and **Funded**, with a dashed connector between them.
- [ ] Scene 07 chain is the same component, fully healed.
- [ ] S1 and S2 are visually dominant in the scene 04 table; S3–S5 recede.
- [ ] All three pillars in scenes 05 and 06 sit horizontally with details underneath.
- [ ] `?expand=1` opens every pillar.
- [ ] Every figure matches §4 exactly — `1.45 cr`, `7.5 lakh`, `₹10,000 cr`, `₹524 cr`, `15,000`, `2%`, `21.3 / 21.0 / 16.3 / 10.8 / 10.0`, `Act XV of 2004`, `36 districts`.
- [ ] No console errors; opens correctly from `file://`.
- [ ] With JS off, all content is still readable in document order.
- [ ] `prefers-reduced-motion` disables all transforms and blur.

---

## 9. Do not

- Do not add a nav bar, logo, footer, contact form, cookie banner, or CTA button. This is a pitch, not a website.
- Do not add stock imagery, illustrations, or icon sets. The visual language is type, colour, hairlines and the chain diagram.
- Do not invent statistics, sources, citations, or additional gaps.
- Do not soften the copy or make it more "marketing". The register is a government policy note.
- Do not make it responsive down to mobile at the cost of the desktop composition.
