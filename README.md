# clone-website

A [Claude Code](https://claude.com/claude-code) skill that reverse-engineers a live website and rebuilds it as a pixel-perfect clone.

It does **not** work in two phases (inspect, then build). It works like a foreman walking a job site: it inspects one section, writes a full specification file for it, hands that file to a dedicated builder agent, and immediately moves on to the next section. Extraction and construction run in parallel.

## What makes it different

Most "clone this site" attempts fail the same few ways. This skill is built around avoiding them:

- **It identifies the interaction model before building.** A sticky sidebar that auto-switches as you scroll is a completely different component from a tab bar you click. Getting this backwards means a rewrite, not a CSS tweak — so the skill scrolls the page *before* it clicks anything.
- **It extracts every state, not just the default.** Tabs get clicked one by one. Scroll-triggered headers get measured at scroll 0 *and* past the threshold, and the diff between them becomes the behavior spec.
- **It uses real computed values.** Every CSS number comes from `getComputedStyle()`. "It looks like `text-lg`" is treated as a failure.
- **It keeps agent tasks small.** If a builder prompt exceeds ~150 lines of spec, the section gets split. Large scope is what makes agents approximate.
- **It writes a spec file per component** to `docs/research/components/`, before dispatching any builder. The spec is the contract, and it stays on disk as an auditable artifact.

## Requirements

- Claude Code
- A browser MCP server (Chrome MCP, Playwright MCP, Browserbase, or Puppeteer). Chrome MCP is preferred when several are available. **The skill cannot run without one.**
- An existing Next.js + Tailwind v4 + shadcn/ui scaffold in the working directory

## Install

```bash
mkdir -p ~/.claude/skills/clone-website
curl -fsSL https://raw.githubusercontent.com/arielaizn/clone-website/main/SKILL.md \
  -o ~/.claude/skills/clone-website/SKILL.md
```

Restart your Claude Code session so the skill is picked up.

## Use

```
/clone-website https://example.com
```

Multiple URLs are processed in parallel, with each site's artifacts kept in its own folder:

```
/clone-website https://example.com https://another.com
```

## What it produces

```
docs/
  design-references/          full-page + per-section screenshots
  research/
    BEHAVIORS.md              every scroll, click, hover and responsive behavior found
    PAGE_TOPOLOGY.md          section order, layering, interaction model per section
    components/*.spec.md      one exhaustive spec per component
scripts/
  download-assets.mjs         pulls every image, video and font to public/
src/
  components/                 the built components
  components/icons.tsx        inline SVGs extracted as React components
```

## Scope

Clones visual layout, styling, component structure, interactions, and responsive behavior, using real extracted assets and copy. Backend, auth, real-time features, SEO and accessibility audits are out of scope by default. Override any of that by saying so when you invoke it.

## License

No license has been chosen yet — all rights reserved by default until one is added.
