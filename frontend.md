# Frontend Engineering Reference

Read this when building or changing UI. Project design system and existing components always win over anything here.

## Contents
- Before building
- Component design
- State and data
- UI states
- Accessibility
- Responsive and RTL
- Styling
- Checklist

## Before building

- Find the existing component, hook, utility, or style token first. Search by behavior ("modal", "table", "toast"), not only by name.
- Look at two or three neighboring components to learn the project's file layout, naming, prop style, and styling approach, then match them.
- Do not add a component library, CSS framework, icon pack, or state library unless the user approves. Use what is installed.

## Component design

- A component has one responsibility you can describe in a short phrase. Split by responsibility (a list, a row, a filter bar), not by line count.
- Separate business logic from presentation only when the logic is non-trivial or reused: pure functions for calculations, hooks or services for data access. Do not create a custom hook for a single `useState`, and do not wrap trivial logic in layers.
- Keep props minimal and explicit. Avoid passing whole objects when two fields are used, and avoid long prop lists that signal the component does too much.
- Prefer composition (children, slots) over boolean-flag explosion (`isSmall isPrimary isRounded isLoading ...`).
- Avoid prop drilling past two or three levels only when it is actually painful; then use the project's established mechanism (context, store), not a new one.

## State and data

- Keep state as local as possible and derive instead of duplicating: if a value can be computed from existing state, compute it.
- One source of truth per piece of data. Do not mirror server data into local state unless you are editing a draft of it.
- Do async work through the project's existing data-fetching pattern. Handle cancellation or stale responses where races are possible (rapid input, route changes).
- Validate user input on the client for usability, but never rely on it for security.

## UI states

Every data-driven view decides, deliberately, what the user sees in each of these:

- **Loading:** skeleton or spinner that does not shift layout.
- **Empty:** explains what is missing and offers the next action.
- **Error:** says what failed in plain language and offers retry where it makes sense; never a blank screen or a silent failure.
- **Success:** including the confirmation after a mutation.
- **Partial/disabled:** buttons disabled or showing progress while a request is in flight, to prevent double submission.

## Accessibility

- Use semantic HTML first (`button`, `a`, `nav`, `main`, `label`, `table`). A clickable `div` is a bug.
- Every form control has a visible or programmatic label. Every image has meaningful `alt` (or empty `alt` if decorative).
- Everything operable by mouse is operable by keyboard, with a visible focus indicator. Modals trap focus and restore it on close; `Escape` closes them.
- Do not convey information by color alone. Maintain sufficient contrast.
- Use ARIA only to fill gaps semantic HTML cannot; wrong ARIA is worse than none.
- Announce dynamic changes (toasts, validation errors) to assistive tech where appropriate.

## Responsive and RTL

- Design mobile-first or follow the project's approach; verify at narrow, medium, and wide widths.
- For RTL languages (Arabic, Hebrew), use logical properties (`margin-inline-start`, `ps-*`/`me-*` in Tailwind) instead of left/right, set `dir` on the root, and mirror directional icons. Check that numbers, mixed-direction text, and form alignment still read correctly.
- Never hard-code user-facing strings if the project has an i18n mechanism; use it.

## Styling

- Use the project's tokens, spacing scale, and typography instead of arbitrary values. Reuse existing utility patterns before inventing new classes.
- Avoid the generic "AI look": identical rounded cards with gradient headers, stock hero sections, decorative emoji, random shadows. Derive visual decisions from the project's identity.
- Extract repeated UI into a component when the pattern recurs and shares behavior, not for every repeated class string.

## Checklist

- [ ] Reused existing components/utilities where they existed
- [ ] All data states handled (loading, empty, error, success)
- [ ] Keyboard and screen-reader usable
- [ ] Works at mobile and desktop widths (and RTL if the app supports it)
- [ ] No new dependency; no unrelated files touched
- [ ] Project's lint, type check, and tests run
