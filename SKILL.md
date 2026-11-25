---
name: human-craft-ui
description: Audit, refactor, and build web interfaces that strip away the synthetic "AI-generated aesthetic" (uniform cardification, icon bloat, badge fatigue, generic copy, cookie-cutter empty states) and replace it with refined, human-designed, enterprise-grade craft in the spirit of Stripe, Linear, GitHub, and Apple, while strictly preserving the project's authoritative Design System (docs/prolance_design_system.md, Cairo typography, Deep Teal palette, semantic surface/border/radius tokens, WCAG 2.2 AA). Use this skill whenever the user asks to review, audit, polish, refactor, or critique a frontend page, layout, dashboard, table, form, or component; whenever you are about to create new UI components or pages; and whenever the output looks like v0 / Lovable / Bolt / default shadcn-ui, even if the user never says "AI look" or "make it feel designed". Trigger on words like "clean up", "looks generic", "too busy", "too many cards", "make it feel professional", "UX review", "polish".
---

# human-craft-ui

Make interfaces look like a person with taste designed them, not like a model filled a template.

**The one-sentence thesis:** the Design System decides *what things look like*; this skill decides *how much of it to use, and where*. Every fix here comes from restraint, structure, spacing, and intent. Never from new tokens.

---

## 0. Design System Authority (NON-NEGOTIABLE)

Read this section before every task. It overrides everything below.

### 0.1 Source of truth
1. Before touching any UI, **read `docs/prolance_design_system.md`** and the project's Tailwind config / CSS variables. Treat them as the only legal vocabulary for color, type, radius, and surface.
2. If the doc and the code disagree, the doc wins. Report the disagreement; do not "fix" it silently.

### 0.2 What you MUST preserve
- Typography: **Cairo** and the defined type scale (title, heading, body, caption).
- Palette: **Deep Teal** brand and all semantic color tokens (`text-foreground`, `text-muted`, `bg-surface`, `border-border`, `text-primary`, and the status tokens defined in the doc).
- Surface, border, and radius tokens as defined.
- **WCAG 2.2 AA** contrast. Never lower contrast to achieve "subtlety".

### 0.3 What you MUST NOT do
- Do not invent, rename, or override tokens, brand colors, or font families.
- Do not use arbitrary values (`text-[#0f766e]`, `rounded-[14px]`, `p-[13px]`, `font-['Inter']`) or ad-hoc palette classes (`slate-*`, `gray-*`, `teal-600`) outside the token set.
- Do not edit `tailwind.config.*`, global CSS variables, or the design system doc as part of a UI refactor.
- Do not add a new visual style "because it looks better". If you feel the system is missing something, **stop and flag it** as a note to the user; do not patch it.

### 0.4 The only levers you may pull
| Lever | Examples |
|---|---|
| Layout restraint | Remove wrappers, flatten nesting, drop redundant containers |
| Structural composition | Table vs. list vs. grid, grouping, section order |
| Spatial hierarchy | `py-*`, `gap-*`, `space-y-*`, asymmetric column spans |
| Typographic hierarchy | Choosing among *existing* scale steps and weights |
| Dividers | `border-b border-border`, `divide-y divide-border` |
| Intentionality | Removing elements with no job; rewriting copy |

> **Rule of thumb:** if your diff adds a new color, font, radius, or shadow, you have broken the contract. If your diff mostly *deletes* classes and elements, you are probably on the right track.

---

## 1. The De-AI-ification Rules

### A. Break the "Cardification of Everything" (Container Intentionality)

**The tell:** every section, metric, and bullet sits in an identical `border border-border rounded-xl bg-surface` box. The page reads as a wall of equal-weight rectangles with no focal point.

**Rules**
1. **No uniform card walls.** Never wrap every section, KPI, and list item in the same bordered surface.
2. **Default to no container.** Group with whitespace first (`py-*`, `gap-*`), then with a rule (`border-b border-border`, `divide-y divide-border`), and only then with a surface.
3. **One level of containment max.** Never nest a card in a card. Never put a bordered box inside a bordered section.
4. **Page-level sections are not cards.** A heading + content block on the page canvas needs no border.

**When a card IS justified** (all must be true: it is a discrete object, and at least one of these):
- It is a **distinct, independently actionable entity** (a project tile the user opens, a freelancer profile summary).
- It is **draggable / sortable / selectable** (kanban item, selectable plan tile, radio-card).
- It is **overlaid or detached** from the page flow (popover, dialog, dropdown, toast).
- It is a **single, deliberate focal point** on the page (one summary panel, one primary form), not a repeating pattern.
- It must be **visually compared side by side** (pricing tiers).

**When a card is WRONG** (use the alternative):
| Content | Use instead |
|---|---|
| Row of 4 KPIs | One flat stat strip, `grid` with `gap-*`, hairline `border-s border-border` between items |
| List of records | Table or `divide-y` list |
| Settings groups | Section heading + `border-b` rows |
| Read-only key/value details | `<dl>` with aligned label/value, no box |
| A page section titled "Recent activity" | Heading + borderless `divide-y` list |

**Human visual rhythm**
- Pick **one focal point per screen** (usually the primary metric or primary action) and give it more space and a larger scale step than its neighbors.
- Vary section weight: a dominant 2/3 column beside a quiet 1/3 column beats three equal columns.
- Use generous, **unequal** vertical gaps: tight inside a group, wide between groups. Uniform `gap-6` everywhere is the machine signature.

---

### B. Cure "Icon Inflation" (Purposeful Visual Restraint)

**The tell:** an icon beside every stat, heading, subtitle, and bullet; often inside a tinted rounded square.

**Rules**
1. **Ban icons on routine content:** stat labels, section headings, subtitles, list bullets, table headers, form labels.
2. **Ban the icon-in-a-colored-box trope:** `p-2 rounded-lg bg-primary/10 text-primary` beside a heading or stat. Delete the box *and* usually the icon.
3. **Typography first.** Hierarchy comes from the type scale, weight, and spacing. Only when that fails to communicate should an icon be considered.
4. **Icons are allowed for:**
   - Interactive controls (icon buttons, menu triggers, toggles), always with an accessible name.
   - Primary navigation items.
   - Critical orientation or state where text alone is slow (error/warning indicator, file type, sort direction).
5. **Ban redundant icons:** a money icon beside "$500.00", a calendar beside "Mar 12, 2026", a user icon beside a person's name, a clock beside "2 hours ago".
6. **Consistency when you do use them:** one size, one stroke weight, `text-muted` by default.

**Test:** *delete the icon. Does the user lose information or an affordance?* If not, it stays deleted.

---

### C. Statuses, Badges & Alert Desensitization

**The tell:** every column is a colored pill; the page screams equally at everything, so nothing is important.

**Rules**
1. **No badge overload.** Not every column, tag, or state earns a pill.
2. **Rank states by actionability:**
   | Tier | Examples | Treatment |
   |---|---|---|
   | Needs action now | Action Required, Pending Approval, Failed, Overdue | Prominent badge using the design system's status tokens |
   | Notable, passive | In review, Draft | Plain text with `text-foreground`, optionally a small dot |
   | Routine / done | Completed, Active, Paid | Calm `text-muted` text, **no pill** |
3. **Max one prominent badge per row.** If a row would carry two, one of them is wrong.
4. **Categories and tags** (skills, types) render as plain comma-separated text or a quiet inline list, not a pill cloud.
5. **Alerts:** no heavy-bordered, fully tinted alert boxes for routine information. Use:
   - Quiet helper text under the relevant field (`text-muted`, caption scale), or
   - A minimal start-accent line: `border-s-2 border-border ps-3` (logical properties only).
   Reserve a full alert component for blocking errors and irreversible consequences.
6. **Never rely on color alone** to convey status (WCAG): pair with a word.

---

### D. Typographic Hierarchy & Tabular Rigor

**Rules**
1. **Use Cairo and the existing scale only.** Hierarchy comes from scale step + weight + spacing, not from wrapping text in boxes or adding colors.
2. **Tabular figures are mandatory** for every financial sum, date, time, metric, percentage, counter, ID, and table number column:
   - Class: `tabular-nums`
   - Where Cairo supports it, also enable the feature set via the design system's utility or a single shared class:
     `font-feature-settings: "tnum", "cv05";`
   - Align numeric columns to the **inline end** (`text-end`) so digits stack cleanly in both RTL and LTR.
3. **Two-tone text discipline:** primary content `text-foreground`, secondary metadata `text-muted`. Do not add a third gray. Do not use `text-muted` for anything the user must read to proceed.
4. **Kill "slate mush":** if secondary text fails AA against its background, raise it to `text-foreground`. Never solve it with opacity (`opacity-60`, `text-foreground/50`) on text.
5. **Limit weights per view:** typically regular + one emphasis weight. No bold-on-bold hierarchies.
6. **Numbers are content:** the key figure gets the larger scale step; its label stays caption-sized and muted above or below it.

---

### E. Move Past Cookie-Cutter Empty States

**The tell:** a centered `border-dashed` box, a sad/inbox icon, and "No data found yet."

**Rules**
1. **Ban** the dashed container + generic icon + generic sentence combination.
2. **An empty state is onboarding, not an apology.** It must answer: *what goes here, and what do I do now?*
3. **Structure:** one short heading (what this area is for), one line of context (why it matters / what happens next), **one** primary CTA (verb + object). Optional secondary text link.
4. **Left-align (start-align) inside the layout flow** where the page is data-oriented; centered treatments only for full-page first-run screens.
5. **Filtered-empty is different from first-run-empty.** For filters, say what was filtered and offer "Clear filters".
6. If the user cannot act (permissions), say who can.

---

### F. Microcopy & Voice (Purge AI Corporate Clichés)

**Banned phrases (non-exhaustive):** "Empower your…", "Seamless experience", "Next-gen", "Supercharge", "Unlock the power of", "Elevate your…", "Streamline your workflow", "Revolutionize", "Effortlessly", "Welcome back! 👋", "Oops! Something went wrong", "We're sorry for the inconvenience".

**Rules**
1. **Say the thing.** Labels are nouns ("Active projects"), buttons are verb + object ("Invite freelancer"), confirmations are past-tense facts ("Changes saved").
2. **Be short.** Headings 1–4 words. Helper text ≤ 1 sentence. Descriptions only when they add information.
3. **No exclamation marks, no emoji, no second-person hype.**
4. **Errors:** what happened + what to do. "Couldn't send the invite. Check the email address and try again." No apology paragraphs, no blaming the user, no raw codes as the message.
5. **Confirmations of destructive actions** name the object and the consequence: "Delete 'Logo redesign'? This can't be undone."
6. **Arabic copy is written, not translated.** Write natural Arabic first; don't mirror English sentence structure. Keep the same tone rules (direct, brief, no marketing language). Keep terminology consistent across the product.

---

### G. Real-World Data Stress-Testing

Design for the ugly case first; the happy case takes care of itself.

**Stress inputs to test every component against**
- Project title of 3+ lines; a single unbroken 60-character token
- Long Arabic full names, mixed Arabic/Latin strings, names with diacritics
- Amounts: `0`, `0.5`, `1,234,567.89`, negative values, long currency codes
- 0 items, 1 item, 500 items; very long breadcrumbs (6+ levels)
- Missing optional fields (no avatar, no description, no date)

**Defensive patterns**
- `min-w-0` on flex/grid children that contain text; `truncate` for single-line, `line-clamp-*` for multi-line, with the full value available via `title`/tooltip or detail view.
- Never size a container to the happy-path text. Let content decide height; constrain width.
- Wide tables scroll inside their own `overflow-x-auto` wrapper; the page never scrolls sideways.
- Skeleton and empty states must not shift layout when data arrives.

**Bidirectional (RTL/LTR) safety, strictly logical properties**
| Instead of | Use |
|---|---|
| `ml-*` / `mr-*` | `ms-*` / `me-*` |
| `pl-*` / `pr-*` | `ps-*` / `pe-*` |
| `left-*` / `right-*` | `start-*` / `end-*` |
| `text-left` / `text-right` | `text-start` / `text-end` |
| `border-l` / `border-r` | `border-s` / `border-e` |
| `rounded-l-*` / `rounded-r-*` | `rounded-s-*` / `rounded-e-*` |
| `float-left` | `float-start` |

- Directional icons (arrows, chevrons) must flip in RTL: `rtl:-scale-x-100` (or `rtl:rotate-180` where appropriate). Non-directional icons must not flip.
- Wrap user-generated mixed-direction strings with `dir="auto"` (or `<bdi>`) so punctuation and numbers don't scramble.
- Never hard-code `dir`; inherit from the document.

---

## 2. Working Method

### 2.1 When auditing / reviewing
1. Read the design system doc. List the tokens you will rely on.
2. Scan the page and produce findings grouped by rule (A–G), each with **file:line, what's wrong, the fix**, and severity (`blocker` = breaks tokens/a11y/RTL, `major` = visible AI-tell, `minor` = polish).
3. Report what is already good. Do not rewrite things that don't violate a rule.

### 2.2 When refactoring
1. Preserve behavior, props, data flow, and accessibility semantics. This is a visual/structural change only.
2. Make the **smallest diff** that resolves the finding. Prefer deleting wrappers over restyling them.
3. Apply rules in this order: **A → B → C → D → E → F → G**. Structure first, decoration last.
4. Re-run the Self-Review Checklist (§4) before returning code.

### 2.3 When building new UI
1. Start from the content, not the container. Write the real copy and real data shapes first.
2. Draft with **zero cards and zero icons**, then add only what fails to communicate.
3. Decide the single focal point before laying out secondary content.

---

## 3. Practical Refactoring Cheatsheet (Before vs. After)

> Examples use only the project's semantic tokens. Status token names (`text-danger`, etc.) must be verified against `docs/prolance_design_system.md`; use whatever the doc defines.

### 3.1 AI Card Grid with Icon Boxes → Flat, Breathable Layout

**Before: the AI tell** (4 identical cards, icon boxes, redundant icons, equal weight)

```tsx
<div className="grid grid-cols-1 gap-6 md:grid-cols-2 lg:grid-cols-4">
  {stats.map((s) => (
    <div key={s.label} className="rounded-xl border border-border bg-surface p-6">
      <div className="mb-4 w-fit rounded-lg bg-primary/10 p-2 text-primary">
        <s.icon className="h-5 w-5" />
      </div>
      <p className="text-sm text-muted">{s.label}</p>
      <p className="mt-1 text-2xl font-bold text-foreground">{s.value}</p>
      <p className="mt-2 text-xs text-muted">Compared to last month</p>
    </div>
  ))}
</div>
```

**After: one focal metric, quiet supporting stats, no boxes, tabular figures, logical properties**

```tsx
<section className="py-8">
  <h2 className="text-lg font-semibold text-foreground">Overview</h2>

  <div className="mt-6 grid gap-y-8 lg:grid-cols-3 lg:gap-x-12">
    {/* Focal point: gets the space and the larger scale step */}
    <div className="lg:col-span-1">
      <p className="text-sm text-muted">Earnings this month</p>
      <p className="mt-1 text-4xl font-bold tabular-nums text-foreground">
        $12,480.00
      </p>
      <p className="mt-1 text-sm tabular-nums text-muted">+8% vs. last month</p>
    </div>

    {/* Supporting metrics: flat strip, hairline separators, no containers */}
    <dl className="grid grid-cols-3 lg:col-span-2 lg:ps-12 lg:border-s lg:border-border">
      {[
        ["Active projects", "7"],
        ["Open proposals", "23"],
        ["Avg. response", "2h 10m"],
      ].map(([label, value]) => (
        <div key={label} className="pe-6">
          <dt className="text-sm text-muted">{label}</dt>
          <dd className="mt-1 text-2xl font-semibold tabular-nums text-foreground">
            {value}
          </dd>
        </div>
      ))}
    </dl>
  </div>
</section>
```

**What changed:** 4 cards → 0 containers; 4 icon boxes → 0 icons; equal weight → 1 dominant metric; `tabular-nums` on every figure; `ps-*/pe-*/border-s` for RTL safety; no new tokens.

---

### 3.2 Badge-Overloaded Table → Scannable Data View

**Before: every cell is a pill, icons everywhere, left/right hard-coded**

```tsx
<table className="w-full">
  <tbody>
    {rows.map((r) => (
      <tr key={r.id} className="border-b border-border">
        <td className="py-3 pr-4">
          <span className="rounded-full bg-primary/10 px-2 py-1 text-xs text-primary">
            {r.category}
          </span>
        </td>
        <td className="py-3 pr-4 text-foreground">{r.title}</td>
        <td className="py-3 pr-4">
          <DollarSign className="inline h-4 w-4" /> {r.budget}
        </td>
        <td className="py-3 pr-4">
          <span className="rounded-full border border-border px-2 py-1 text-xs">
            <Clock className="inline h-3 w-3" /> {r.deadline}
          </span>
        </td>
        <td className="py-3 text-left">
          <Badge>{r.status}</Badge>
        </td>
      </tr>
    ))}
  </tbody>
</table>
```

**After: text-first, status ranked by actionability, one badge only where action is needed**

```tsx
<div className="overflow-x-auto">
  <table className="w-full text-start">
    <thead>
      <tr className="border-b border-border text-sm text-muted">
        <th className="py-3 pe-4 text-start font-medium">Project</th>
        <th className="py-3 pe-4 text-end font-medium">Budget</th>
        <th className="py-3 pe-4 text-end font-medium">Deadline</th>
        <th className="py-3 text-start font-medium">Status</th>
      </tr>
    </thead>
    <tbody className="divide-y divide-border">
      {rows.map((r) => (
        <tr key={r.id}>
          <td className="max-w-0 py-4 pe-4">
            <p dir="auto" className="line-clamp-2 font-medium text-foreground">
              {r.title}
            </p>
            {/* Category demoted from pill to quiet metadata */}
            <p className="mt-0.5 truncate text-sm text-muted">{r.category}</p>
          </td>
          <td className="py-4 pe-4 text-end tabular-nums text-foreground">
            ${r.budget}
          </td>
          <td className="py-4 pe-4 text-end tabular-nums text-muted">
            {r.deadline}
          </td>
          <td className="py-4">
            {r.needsAction ? (
              <Badge /* design-system status badge for action-required */>
                {r.status}
              </Badge>
            ) : (
              <span className="text-sm text-muted">{r.status}</span>
            )}
          </td>
        </tr>
      ))}
    </tbody>
  </table>
</div>
```

**What changed:** 3 pills/row → at most 1, only for items needing action; money/clock icons removed (the formatted values already say it); numeric columns `text-end` + `tabular-nums`; long titles clamp to 2 lines with `dir="auto"`; `pr-*`/`text-left` replaced by logical properties; table scrolls inside its own wrapper.

---

### 3.3 Dashed Empty State → Action-Oriented State

**Before**

```tsx
<div className="flex flex-col items-center justify-center rounded-xl border-2 border-dashed border-border p-12 text-center">
  <Inbox className="mb-4 h-12 w-12 text-muted" />
  <h3 className="text-lg font-semibold text-foreground">No data found yet</h3>
  <p className="text-sm text-muted">There's nothing here yet. Check back later!</p>
</div>
```

**After: says what this area is for, and gives one clear next step**

```tsx
<section className="max-w-md py-12">
  <h3 className="text-lg font-semibold text-foreground">No proposals yet</h3>
  <p className="mt-2 text-sm text-muted">
    Proposals from freelancers appear here once your project is published.
  </p>
  <div className="mt-6 flex items-center gap-4">
    <Button>Publish project</Button>
    <a href="/help/proposals" className="text-sm text-primary underline-offset-4 hover:underline">
      How proposals work
    </a>
  </div>
</section>
```

**What changed:** dashed box, decorative icon, and filler sentence removed; heading names the real object; one primary CTA (verb + object); start-aligned in the page flow; no new tokens.

---

## 4. Self-Review Checklist (execute before returning ANY UI code)

Answer each question explicitly in your head; fix and re-check on any "no". Do not return code until all 8 pass.

1. **Token integrity:** Does every color, font, radius, and spacing value come from the Design System tokens, with zero arbitrary values (`[#…]`, `[14px]`), zero raw palette classes, and no edits to config or the design doc?
2. **Card audit:** Could any card be replaced by whitespace, a `border-b`, or a `divide-y` list? Is there any card-in-card or any wall of identical bordered boxes? Does every remaining card meet the "justified" criteria in §1.A?
3. **Icon restraint:** Is every remaining icon either an interactive control, navigation, or critical orientation? Are there zero icons in tinted boxes and zero icons redundant with adjacent text or a formatted value?
4. **Status hierarchy:** Is there at most one prominent badge per row, reserved for states that need action, with routine/completed states shown as calm text and no color-only meaning?
5. **Typographic rigor:** Does hierarchy come from the existing Cairo scale and weights (not boxes), do all numbers, dates, and amounts use `tabular-nums` and `text-end` alignment, and does all text meet WCAG 2.2 AA with only `text-foreground` / `text-muted` for text color?
6. **Copy & empty states:** Is every string free of AI clichés (§1.F), direct and brief, and does every empty state explain what belongs there with a single clear CTA (no dashed box, no generic icon)?
7. **Stress & overflow:** Does the layout survive 3-line titles, long Arabic names, `0` and `1,234,567.89`, 0/1/500 items, and deep breadcrumbs via `min-w-0`, `truncate`/`line-clamp`, and scroll containers, without layout shift?
8. **RTL safety:** Are there zero physical properties (`ml/mr/pl/pr/left/right/text-left/text-right/border-l/border-r`), do directional icons flip under `rtl:`, and does mixed-direction user content use `dir="auto"`?

---

## 5. Output Contract

When returning results, always include:

1. **Summary**: 2–3 lines on what changed and why (by rule letter, e.g., "A, B, C").
2. **Code**: the refactored component(s), complete enough to drop in.
3. **Token note**: confirm no tokens were added or altered; flag any case where the design system seemed to lack something (do not patch it).
4. **Checklist result**: "8/8 passed", or list any item intentionally waived and why.

If the user's request conflicts with the Design System (e.g., "make it blue", "use Inter"), **do not comply silently and do not refuse silently**: explain that the design system is authoritative and propose a solution within the tokens.
