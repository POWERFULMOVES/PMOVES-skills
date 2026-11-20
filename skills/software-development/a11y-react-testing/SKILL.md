---
name: a11y-react-testing
description: "Use when writing or repairing accessibility tests for React/Next.js components with Radix UI, Testing Library, and jest-axe — especially focus-order verification, composite widgets (tabs/sliders), and axe rule exclusions for deliberate ARIA patterns."
---

# a11y-react-testing — React accessibility testing patterns

Battle-tested patterns from repairing 82 suite tests across a Next.js + Radix + Recharts app (PMOVES-ToKenism-Multi, PRs #66/#67/#72).

## Focus-order verification: reachability, not press-counting

Strict 1:1 `user.tab()` press counting is brittle with composite widgets — Radix Tabs renders a `tablist` container with `tabindex=0` AND roving-tabindex triggers, so exact press positions shift. Verify **reachability in order** with a check-then-press window instead:

```typescript
const advance = async (expected: HTMLElement, direction: 'forward' | 'back') => {
  for (let press = 0; press < 6; press++) {
    // Check first: a pass may start with focus already on its target
    if (document.activeElement === expected) return true;
    await user.tab({ shift: direction === 'back' });
    if (document.activeElement === expected) return true;
  }
  return false;
};
```

Composite-widget rules (WAI-ARIA):
- `tablist` containers and `tabpanel` surfaces delegate focus — exclude from candidates (`querySelectorAll` filter on role)
- Tab triggers legally hold focus via roving tabindex — accept a tab trigger holding focus at any step
- A pass may START with focus on its target (e.g. reverse pass begins where forward ended) — check before pressing

## jsdom environment gotchas

- `document.documentElement.lang` is unset — set `document.documentElement.lang = 'en'` in `jest.setup.js` if any test asserts it
- Recharts/ResponsiveContainer needs a `ResizeObserver` mock whose callback receives `[{ contentRect: { width, height } }]` — an empty-args callback crashes consumers reading `entries[0].contentRect`
- Remove any broken `observe` wrapper that passes raw `arguments` to the original method
- `.next/standalone` build output causes jest-haste-map naming collisions — add `modulePathIgnorePatterns: ['<rootDir>/.next/standalone']`

## axe rule exclusions for deliberate patterns

`aria-allowed-role` rejects `role="slider"` on text inputs — but explicit slider semantics on number/text inputs is a valid documented pattern (ARIA 1.2 allows it when keyboard behavior matches). Exclude per-call with merge semantics, never clobber existing rules:

```typescript
await axe(container, {
  rules: {
    'aria-allowed-role': { enabled: false },
    'color-contrast': { enabled: true },
  },
});
```

When a harness exposes `excludedRules` options, MERGE caller exclusions with defaults — replacement silently re-enables defaults (a real review catch).

Never add a second positional object: `axe(el, {rules}, {options})` — jest-axe ignores the third argument and the first object's original options are lost.

## Radix UI in jsdom

- Sliders don't respond to `user.keyboard()` — dispatch `fireEvent.keyDown(slider, { key: 'ArrowRight' })` and assert `onValueChange` calls
- Horizontal sliders only step on Left/Right; Up/Down are vertical-slider keys
- `TabsContent` renders with `tabIndex=0` (a real focus stop) — account for it in focus walks or disable
- Inactive tab panels unmount by default; `forceMount` keeps them in DOM but `hidden` — queries still find them, focus walk skips them correctly
- Multiple renders of components containing `ModelExplanation` produce duplicate tablists — scope queries with `within(screen.getByRole('form'))` and recurse: `within(within(form).getByRole('tablist')).getByRole('tab', ...)`

## Test-fixture requirements

- Heading-role assertions need a real heading — shadcn `CardTitle` renders a plain `div`; add `role="heading"` (or fix the fixture, not the query)
- `getByRole('img')` expectations break when components migrate to `role="figure"` (correct for charts — keeps descendants AT-accessible)
- After form-scoped fixes, re-check sibling suites that render the same component bare — unscoped queries hit duplicates first
