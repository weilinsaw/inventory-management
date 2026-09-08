---
name: vue-perf-audit
description: Analyzes this app's Vue component structure and produces a prioritized report of performance and code-reuse opportunities. Use this skill when asked to analyze, audit, or review Vue components for performance or duplication, or to suggest optimizations for the client app.
---

# Vue Performance & Reuse Audit

This is a **guidelines skill**: it tells you what to look for, where to look, and how to report it. It is **advisory only** — running this skill never edits files. If the user later asks you to act on a finding, that follow-up still goes through this repo's normal rules (any `.vue` edit is delegated to **vue-expert** per root `CLAUDE.md`).

## Scope

- Target: `client/src/views/*.vue` and `client/src/components/*.vue` only. Never touch `server/` — this is a frontend-only audit.
- Default scope is the whole tree. If the user (or the skill's `args`) names a specific file, component, or folder, scope the scan to just that target instead — this is common right after building a new feature (e.g. auditing `Restocking.vue` right after it was added).
- Skim `client/CLAUDE.md` first if it's not already in context — the checklist below is this app's own documented conventions turned into grep-able rules, and the report should cite it when a finding contradicts it.

## Process

1. `Glob` the target scope for `.vue` files.
2. Run the mechanical checks below with `Grep` across that file set first — they catch most issues cheaply and give you line numbers for the report.
3. `Read` any file that a mechanical check flagged, plus any file that looks large (rough gut check: template > ~100 lines or `<script>` > ~150 lines) even if no grep hit — size alone is a code-reuse signal per this repo's own "When to extract component" guidance in `client/CLAUDE.md`.
4. Cross-reference duplication across files, not just within one file — the highest-value reuse findings in this app are things repeated across *several* views/components (a shared data-loading shape, a shared modal skeleton, a shared status→badge map), not local nitpicks.
5. Write the report per the Output format below. Do not edit any file as part of this process.

## Performance checklist

- **`v-for` keyed by index** — `:key="index"` (or a destructured loop index) instead of a stable field (`sku`, `id`, `month`). This app's own `client/CLAUDE.md` explicitly forbids it (Vue reuses/misassigns DOM nodes on reorder). Grep: `v-for="\(.*index.*\).*:key="index"` and similar — but also read the surrounding loop, since sometimes the visible var is named something else.
- **Computation in a method instead of `computed()`** — a `setup()` function whose template calls a plain function (not a `computed` ref) to derive a value from reactive state on every render, instead of caching it in a `computed()`. Look for functions returned from `setup()` that read `.value` off refs and are invoked directly in the template (`{{ someMethod() }}` or `:prop="someMethod()"`) rather than bound as `{{ someComputed }}`.
- **Unbounced `watch` on rapidly-changing input** — a `watch()`/`watchEffect()` on a `ref` that's bound to a text input (search boxes, free-typed filters) without `watchDebounced` from `@vueuse/core`, per the debounce pattern this repo's own `client/CLAUDE.md` documents. Triggering an API call or heavy recompute on every keystroke is the failure mode.
- **Filtering inside the template loop instead of in a computed** — `v-for` combined with a sibling `v-if` on each iteration (filtering during render) instead of a `computed()` that pre-filters the array once per dependency change.
- **Object/array literals created inline in template bindings** — `:style="{ ... }"`, `:class="[...]"`, or `:options="{...}"` with a literal built directly in the template (not pulled from a `computed`/ref), which allocates a new object every re-render and can defeat child-component memoization.
- **Duplicate API calls across sibling components for the same data** — more than one component independently calling the same `api.getX()` in its own `onMounted` where the data could be fetched once and shared (via a composable or lifted to a parent) instead.

## Code-reuse checklist

- **Repeated modal boilerplate** — this app has multiple modal components (check `client/src/components/*Modal.vue`) that each hand-roll the same `<Teleport to="body">` + `<Transition name="modal">` + `.modal-overlay`/`.modal-container`/`.modal-header`/`.close-button` structure and near-identical `.modal-enter-*`/`.modal-leave-*` CSS. If 3+ files repeat this skeleton, flag it as a candidate for a shared `BaseModal.vue` (slot-based: header slot or title prop, default slot for body, footer slot) that each modal wraps instead of reimplementing.
- **Repeated data-loading boilerplate** — `client/CLAUDE.md` documents a standard `loading`/`error`/`data` ref + try/catch/finally shape that every view repeats by hand. If most views duplicate this near-verbatim, flag it as a candidate for a `useAsyncData(fetchFn)` composable (`composables/`) returning `{ data, loading, error, reload }`, consistent with this repo's existing composable pattern (`useFilters`, `useI18n`, `useAuth`).
- **Repeated status/trend → badge-class maps** — small `status → 'success'|'warning'|'danger'|'info'` (or `trend → ...`) mapping objects/functions redefined in more than one file instead of one shared helper (e.g. alongside `utils/currency.js`).
- **Repeated formatting logic** — currency/date formatting reimplemented inline in a component instead of importing the existing `formatCurrency` (`utils/currency.js`) or a shared date formatter.
- **Repeated inline SVG icons** — the same icon markup (or near-identical stroke-icon markup) copy-pasted across components instead of extracted into a small shared icon set/component, once there are enough of them to be worth it.
- **Components past this repo's own extraction threshold** — per `client/CLAUDE.md`'s "When to extract component" (template > 100 lines, logic > 150 lines, or reused in multiple places), flag components that have grown past this without being split.

## Output format

Report directly in chat (do not write a report file unless asked). Structure:

```
## Performance
1. [file:line] <one-line issue> — <concrete suggested fix, naming the actual file/pattern to reuse in this repo>
...

## Code Reuse
1. [file:line, file:line, ...] <one-line issue, naming how many files repeat it> — <concrete suggested fix>
...
```

- Order each section by impact (a pattern repeated across many files or affecting a frequently-rendered list outranks a one-off in a rarely-visited view).
- Every suggested fix must be concrete and specific to this codebase — name the actual composable/util/component to introduce or reuse, not generic advice like "consider memoizing."
- If a category has no findings, say so in one line rather than omitting the section.
- End with one sentence offering to implement any of the findings if the user wants — but do not start implementing unless they say yes.
