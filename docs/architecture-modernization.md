# Architecture Modernization Analysis

Analysis of current tech stack and recommended changes if rebuilding from scratch.
Intended as a reference for a new project based on django-schema-graph with updated infrastructure.

---

## Current Stack (django-schema-graph v3.1.0)

| Layer | Current |
|---|---|
| Framework | Vue 2.6 |
| Build | Webpack 4 |
| UI components | Vuetify 2.2 |
| State | Custom reactive object (`graphData.js`) |
| Graph rendering | vis-network 7.3 |
| Data delivery | Inline `window.schema` in HTML |
| Backend | Django TemplateView + attrs + cattrs |

---

## Recommended Stack

| Layer | Current | Recommended |
|---|---|---|
| Framework | Vue 2 | Vue 3 or React |
| Build | Webpack 4 | Vite |
| UI components | Vuetify 2 | Tailwind CSS |
| State | Custom reactive obj | Pinia (Vue 3) / Zustand (React) |
| Graph rendering | vis-network | Cytoscape.js (or keep vis) |
| Data delivery | Inline `window.schema` | Async JSON endpoint |
| Backend | attrs + cattrs + TemplateView | Keep as-is |

---

## Rationale by Layer

### Frontend Framework: Vue 2 → Vue 3 or React

Vue 2 hit end-of-life in December 2023. Carrying dead weight.

- Vue 3 or React are both fine replacements
- Lean **React** for this project: component tree is shallow, no benefit to Vue's Options API, broader ecosystem for graph/canvas tooling
- If keeping Vue, upgrade to Vue 3 + Vite and drop Vuetify for Tailwind

### Build Tool: Webpack 4 → Vite

Webpack 4 is two major versions behind and slow. Vite gives near-instant dev startup and HMR. Config for a project this size would be ~10 lines vs. the current three-file webpack split (`webpack.common.js` + `webpack.dev.js` + `webpack.prod.js`). Strict upgrade, no real tradeoff.

### UI Components: Vuetify 2 → Tailwind CSS

Vuetify 2 weighs ~300kb and is tightly coupled to Vue 2. The UI only actually uses:
- A sidebar drawer
- Icon buttons
- Tooltips
- A progress bar

Tailwind (or even plain CSS) is far lighter and sufficient. The sidebar is not complex enough to justify a full Material Design framework.

### State Management: Custom reactive object → Pinia / Zustand

`graphData.js` is a hand-rolled reactive store. It works but is fragile:
- State updates require manually calling `update()`
- No clear separation — mutations, derived state, and rendering are mixed together
- No devtools support

**Pinia** (Vue 3) or **Zustand** (React) give:
- Devtools for inspecting state changes
- Clear separation of state, computed/derived values, and actions
- Automatic reactivity without manual `update()` calls

### Graph Library: vis-network → Cytoscape.js (or keep vis)

vis-network has friction points:
- Imperative/DOM-based API fights against Vue/React's declarative model (hence the awkward `graphData.js` reactive shim)
- Physics stabilization is slow to converge on large schemas, requiring the progress bar UX
- Custom node shapes (field-expansion boxes) require generating SVG/HTML label strings manually

**Cytoscape.js** has better performance on large graphs and a cleaner API for custom node rendering. Worth the switch if the project expects large schemas (100+ models) or needs richer node layouts.

**D3-force** is also viable but requires writing more layout code from scratch.

Swapping graph libraries is high-risk/high-effort — only worth it if performance or rendering flexibility becomes a real constraint.

### Data Delivery: Inline `window.schema` → Async JSON endpoint

For large Django projects the inline JSON blob can be very large and blocks initial page render.

Recommended: add a dedicated endpoint (`/schema/data.json`) and fetch async, showing a loading state.

Benefits:
- Non-blocking page load
- Data independently accessible for tooling and tests
- Easier to cache or version

### Backend: Keep as-is

The backend is well-designed and should be kept largely unchanged:
- `attrs` + `cattrs` for the data model and serialization is clean and correct
- `Node` / `Edge` / `Group` / `Graph` domain model is clear and well-scoped
- Single `TemplateView` is simple and works
- `get_schema()` cleanly iterates installed apps/models

Only change: switch to the async JSON endpoint described above.

---

## Why the Frontend Accumulated Debt

Vue 2 + Webpack 4 were reasonable choices circa 2020 when this project was likely initialized. The project hasn't had a major dependency refresh since. The backend held up well; the frontend did not age as gracefully.
