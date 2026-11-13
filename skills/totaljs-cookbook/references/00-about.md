# Total.js 5 cookbook — building apps

Practical recipes, conventions and lessons learned for building applications on **Total.js 5**. It covers both kinds of projects:

- **server-rendered websites** — a content catalog with search, detail pages, pagination and thorough SEO, with no database and no frontend framework,
- **API backends** — plugins, actions, authorization, PostgreSQL, uploads, WebSockets and background jobs.

The cookbook is project-independent and meant as the single reference for new projects: an app should be buildable from this file alone. The examples use generic names ("items", "orders"); replace them with your own domain. Copy the **mechanism**, never another product's tables or schema names.

**How to read it:** sections 1–6 are the rules that apply to every file. Sections 7–18 are the backend. Sections 19–23 are the website layer. Sections 24–28 are quality: performance, security, anti-patterns, gotchas and a checklist. When two recipes seem to compete, the tables in [section 1](01-architecture.md) ("Where each kind of code goes") and [section 26](26-anti-patterns.md) decide.
