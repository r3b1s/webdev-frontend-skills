---
name: framework-astro-dev
description:
  Core Astro knowledge plus hard-won gotchas for building and debugging Astro
  sites/dashboards - component model, rendering modes, the CSS scoping model
  (and how it silently breaks layout-authored styles), the dev-server/Vite
  proxy, and client-script timing. Read before starting Astro work, and
  especially before debugging "my styles aren't applying" or "my dev proxy
  breaks auth" issues.
license: MIT
metadata:
  author: workspace-hub
  version: '1.0.0'
  category: framework
  tags: [astro, vite, ssg, ssr, css-scoping, dev-server, dashboard]
  platforms: [web, javascript, typescript]
---

# Astro Development Skill

Astro is a component-based web framework that ships zero JS by default and
lets you opt individual components into interactivity ("islands"). Most of
what breaks in practice isn't the framework's headline features — it's a
short list of scoping, timing, and dev-server behaviors that are easy to
misread from source alone and only show up once you actually render the page
in a browser. This skill front-loads those.

## When to use this skill

- Starting or reviewing an Astro project (`.astro` files, `astro.config.mjs`).
- Debugging "my CSS isn't applying to X" in an Astro layout/page.
- Wiring an Astro dev server to a separate backend (proxying `/api` or similar).
- Any dashboard-style Astro app with client-side data loading and multiple
  independent page scripts sharing state.

## Core model

- **`.astro` files are components.** The fenced frontmatter (`---`) runs
  server-side (build time for static output, request time for SSR) and has
  no access to the DOM. Everything below the second `---` is the template:
  HTML plus JSX-like expressions, but it is **not** hydrated by default —
  it's rendered to plain HTML.
- **Zero JS by default.** A `<script>` tag in a `.astro` file is processed
  by Vite and shipped as a real ES module to the browser; it runs like any
  client script, but nothing in the *template* is interactive unless you add
  an island.
- **Islands** (`@astrojs/react`, `-vue`, `-svelte`, etc.) opt one component
  into client hydration via a `client:*` directive: `client:load` (hydrate
  immediately), `client:idle`, `client:visible`, `client:media`,
  `client:only`. Everything else on the page stays static HTML.
- **Output modes**: `output: "static"` (default; `astro build` emits plain
  files, `astro dev`/`astro preview` just serve them — no server-side
  request handling), `"server"` (every route is rendered on request; you get
  `pages/*.ts` API routes and `Astro.request`), `"hybrid"`/per-route
  `export const prerender = false` to mix the two.
- **Layouts + slots.** A layout is just a component that renders `<slot />`
  where the page content goes. `Astro.props` passes data down from a page to
  its layout the same way any component receives props.
- **File-based routing.** `src/pages/foo.astro` → `/foo`; `src/pages/index.astro`
  → `/`; `[slug].astro` for dynamic routes; `pages/api/foo.ts` for
  server-mode API endpoints.
- **`astro.config.mjs`'s `vite` key is a real Vite config** — anything Vite
  accepts (`server.proxy`, `build.rollupOptions`, plugins) goes here
  unchanged. `vite.server.*` only affects `astro dev`; it has zero effect on
  `astro build` or `astro preview` (there is no Vite dev server in either).
- **TypeScript, Markdown/MDX, and `astro:assets` image optimization** are
  built in; content collections (`src/content/`) give you schema-validated
  Markdown/data collections without a separate CMS.

## Gotchas (hard-won, not obvious from reading source)

### 1. Scoped `<style>` never reaches slotted or client-injected content — the big one

Astro scopes every `<style>` block in a `.astro` file by default: it rewrites
each selector to `.foo[data-astro-cid-HASH]` and stamps that
`data-astro-cid-HASH` attribute onto the elements *that component's own
template* renders directly.

The trap: a **layout's** `<style>` block only reaches markup the layout
itself writes (its nav, header, dialogs, etc.). It does **not** reach:

- Anything passed through `<slot />` — a page's own markup keeps *that
  page's* scope hash, not the layout's, so the layout's scoped rules never
  match it.
- Anything built at runtime via `el.innerHTML = "..."` from client script —
  scoping is a build-time transform of literal template markup; dynamically
  injected HTML never receives an astro-cid attribute at all, from any
  component.

What still *looks* like it's working even when this is broken: CSS custom
properties and any bare-tag selector on an element the layout *does* render
directly (`:root { --x: ... }`, `body { color: var(--ink) }`). Custom
properties inherit through the DOM regardless of scoping, and `<body>` is
usually layout-authored — so a whole app can look "mostly themed" (right
background, right text color) while every component-level rule (`.button`,
`.card`, `.data-table`, a custom grid class) is silently falling back to
browser defaults. **This is invisible from reading the CSS alone** — it
reads correct, and a quick glance at a screenshot can still look
"intentional" if the color inheritance carries enough of the illusion. The
only reliable check is opening the real page and reading
`getComputedStyle(el)` (or checking whether the element carries the
matching `data-astro-cid-*` attribute) for a rule you expect to be applying.

**Fix:** put layout-level CSS meant to style page/dashboard content in
`<style is:global>` instead of a scoped `<style>`. (Astro allows multiple
`<style>` tags per file — mixing one scoped block for layout-only chrome and
one `is:global` block for everything else is fine and common.)

### 2. Vite's dev-server proxy rewrites `Host` regardless of `changeOrigin`

If you proxy `/api` (or similar) from `astro dev` to a separate backend, Vite's
proxy sets the outbound request's `Host` header to the **proxy target**, not
the original incoming host — this happens whether `changeOrigin` is `true`,
`false`, or omitted. Any backend logic that compares `Origin` against `Host`
(same-origin checks, CSRF validation, pairing flows) will see them mismatch
on every single proxied request and reject it, even though the same request
works fine hit directly against the backend.

**Fix:** use the proxy's `configure` hook to put the original `Host` back
before the request goes out:

```js
// astro.config.mjs
vite: {
  server: {
    proxy: {
      "/api": {
        target: "http://127.0.0.1:PORT",
        configure(proxy) {
          proxy.on("proxyReq", (proxyReq, req) => {
            proxyReq.setHeader("host", req.headers.host);
          });
        },
      },
    },
  },
},
```

Diagnose this by adding a temporary log of `request.headers.host` and
`request.headers.origin` in the backend's same-origin check — don't assume;
confirm the actual header values reaching the backend.

### 3. `astro dev`'s bind address can end up IPv4/IPv6-split

Observed: `astro dev` listening on `[::1]:4321` only. `curl 127.0.0.1:4321`
then fails with connection refused while `curl localhost:4321` succeeds,
because `localhost` resolves to `::1` first on that machine. If a dev server
"isn't reachable" but the process is clearly running, check `ss -tlnp` for
which literal address it bound before assuming it crashed — and prefer
`localhost` over a hardcoded `127.0.0.1` when scripting against it, or pass
`--host` explicitly to pin one family.

### 4. `location.replace(path)` with a literal string drops the URL hash

A very common redirect bug, not Astro-specific but shows up constantly in
Astro "redirect this static route to a client route" patterns:

```js
// Drops any #fragment on the current URL — silently breaks anything
// (a one-time token, a pairing secret, a deep link) carried in the hash.
window.location.replace("/target");

// Preserves it.
window.location.replace("/target" + location.search + location.hash);
```

A `<meta http-equiv="refresh" content="0;url=/target">` fallback has the
same limitation and can't be fixed the same way (it's static markup, not
JS) — that's an acceptable no-JS fallback gap since anything relying on the
hash usually requires JS anyway.

### 5. A page script's first render always runs before a sibling async load resolves

If page A's own `<script>` calls `renderX()` unconditionally at the bottom,
and a *separate* script (e.g. a shared layout's bootstrap script) does an
async `fetch` and populates some shared `state` object — the page's first
`renderX()` call is **guaranteed** to run against empty/default state. Two
independent `<script type="module">` blocks (which is what Astro's `<script>`
tags become) cannot await each other; the synchronous call always wins the
race against any `await`, no matter what order the tags appear in the
document.

Symptom: the page's own render (a table, a chart, a stat) looks stuck in an
empty/loading state on first paint, only reflecting real data after a
manual refresh action re-triggers that page's render function — while
sibling UI driven by the shared bootstrap script (a sidebar summary, etc.)
already shows real data, because *that* script re-renders itself after its
own `await` resolves.

**Fix:** have the shared state-loader dispatch a custom event after it
updates state, and have each page listen for it and re-render:

```js
// shared loader
async function reload() {
  /* ...populate shared state... */
  document.dispatchEvent(new CustomEvent("app:state"));
}

// each page
document.addEventListener("app:state", renderX);
renderX(); // still call once immediately for the instant (if empty) first paint
```

### 6. Unsized SVG/canvas fills its grid/flex track, not a sane default

An `<svg viewBox="...">` (or a chart library's container) with no explicit
CSS width/height will happily expand to 100% of its parent's available
inline size. Inside a CSS Grid track defined with `fr`/`minmax()`, that can
balloon to hundreds of pixels instead of the small icon/chart you meant.
Always give chart/icon containers an explicit fixed size in CSS
(`width: 60px; height: 60px;` or similar) rather than relying on intrinsic
sizing — don't diagnose "why is my chart huge" by tweaking the chart
library's options before checking the container has a real size.

### 7. Canvas-rendered chart libraries (ECharts, etc.) can't read CSS custom properties

A library like Apache ECharts draws to `<canvas>`; its option config wants
literal color values, not `var(--x)` strings — it cannot resolve a CSS
custom property itself. Resolve at render time via
`getComputedStyle(document.documentElement).getPropertyValue("--x").trim()`,
and re-run `chart.setOption(...)` whenever the active theme changes (there's
no automatic reactivity to a CSS variable changing — you have to re-render
on your own theme-toggle event, e.g. the same custom-event pattern as #5).

Also: import ECharts modularly instead of the full bundle, since it ships
20+ chart types you almost certainly don't need:

```js
import * as echarts from "echarts/core";
import { GaugeChart } from "echarts/charts"; // only the chart types you use
import { CanvasRenderer } from "echarts/renderers";
import { TooltipComponent } from "echarts/components";
echarts.use([GaugeChart, CanvasRenderer, TooltipComponent]);
```

And a single ratio against a limit ("23% of budget used") is a **meter**
(ECharts `gauge` series, no pointer/needle, `axisLine.lineStyle.color` as a
`[[fraction, fillColor], [1, trackColor]]` stop list), not a 2-slice pie —
reach for `gauge` there, not `pie`.

### 8. Reuse chart instances; don't recreate them on every re-render

`echarts.getInstanceByDom(container)` returns an existing instance if one is
already attached; only call `echarts.init(container)` when it returns
`undefined`. Re-rendering by tearing down and recreating the DOM (e.g. via
`container.innerHTML = ...`) every time leaks chart instances and is far
slower than `chart.setOption(newOption, true)` on the same instance. Dispose
explicitly (`chart.dispose()`) when a container needs to go back to a plain
text/empty state, and hook `window.addEventListener("resize", ...)` to call
`chart.resize()` on every live instance — it will not resize itself.

## Fast verification technique

When you have Playwright's Chromium available (often already installed in
dev sandboxes — check before assuming you need to install anything), skip
writing a full spec file for a one-off "is this actually rendering
correctly" question. A throwaway script is faster and gives ground truth
instead of a guess:

```js
const { chromium } = require("playwright-core"); // or "playwright"
(async () => {
  const browser = await chromium.launch({ headless: true });
  const page = await browser.newPage({ viewport: { width: 1440, height: 900 } });
  page.on("dialog", (d) => d.accept()); // auto-accept confirm()/alert() so nothing hangs
  page.on("pageerror", (e) => console.log("[pageerror]", e.message));
  await page.goto("http://localhost:4321/...");
  await page.waitForTimeout(1000);
  const info = await page.evaluate(() => {
    const el = document.querySelector(".thing");
    return { display: el && getComputedStyle(el).display, attrs: el && el.getAttributeNames() };
  });
  console.log(JSON.stringify(info, null, 2));
  await page.screenshot({ path: "/tmp/check.png" });
  await browser.close();
})();
```

`page.evaluate()` returning real computed styles / DOM attributes is what
actually caught gotcha #1 above — reading the CSS source made it look
correct; only checking the rendered DOM revealed the scoping mismatch.
