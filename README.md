# AGENTS.md Framework

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Build Status](https://img.shields.io/github/actions/workflow/status/Marcus-Jenshaug/AGENTS.md/ci.yml?branch=main)](https://github.com/YOUR-USERNAME/YOUR-REPO/actions)
[![Docs](https://img.shields.io/badge/docs-online-green.svg)](#documentation)

A framework-agnostic guide and configuration system for **AI-assisted code generation**.
This project provides a standard **AGENTS.md** process document and `agents.config.json` boilerplate so any team can:

* Convert design mockups into **production-ready pages and components**
* Bind generated UIs to backend APIs and authentication
* Ensure non-destructive, idempotent runs (safe to re-run anytime)
* Produce accessible (WCAG 2.1 AA) and responsive frontends
* Integrate seamlessly with CI/CD

---

## Why?

Modern projects often rely on manual, repetitive coding of UI pages from design files.
This repo defines a reusable agent workflow that allows **automated yet safe implementation** across any stack (React, Vue, Svelte, Angular, etc.).

---

## Features

* 🔍 **Mockup discovery** — detect new design files (`designmockup/`)
* ⚛️ **UI generation** — build components, pages, and routes automatically
* 🔗 **API integration** — connect frontend to backend APIs via config
* ♻️ **Idempotent** — no overwrites unless explicitly allowed
* ♿ **Accessibility-first** — WCAG 2.1 AA, keyboard & screen reader support
* 🚀 **CI/CD ready** — works with GitHub Actions or any pipeline
* 📖 **Documentation-driven** — always updates `README.md`, `ARCHITECTURE.md`, `STORAGE.md`, and `CHANGELOG.md`

---

## Quick Start

1. **Clone the repo**

   ```bash
   git clone https://github.com/YOUR-USERNAME/agents-md.git
   cd agents-md
   ```

2. **Add your mockups** under:

   ```
   designmockup/
     dashboard.png
     profile.html
     projects.svg
   ```

3. **Adjust config** (`agents.config.json`):

   ```json
   {
     "mockupDir": "designmockup",
     "routeBase": "/",
     "skip": ["globals"],
     "output": {
       "pagesDir": "frontend/src/pages",
       "componentsDir": "frontend/src/components",
       "routerFile": "frontend/src/router.tsx"
     }
   }
   ```

4. **Run the agent**

   ```bash
   npm run agent:plan        # Dry run
   npm run agent:generate    # Generate pages/components
   ```

---

## Documentation

* [AGENTS.md](./AGENTS.md) — Full process specification
* [agents.config.json](./agents.config.json) — Example configuration
* [ARCHITECTURE.md](./ARCHITECTURE.md) — System structure (customizable)
* [STORAGE.md](./STORAGE.md) — Storage rules (file or DB based)

---

## Contributing

Pull requests are welcome!
Please read [CONTRIBUTING.md](./CONTRIBUTING.md) (coming soon) before submitting changes.

---

## License

This project is licensed under the [MIT License](./LICENSE).

---

With this setup, anyone can plug in their mockups and configs, and the agent will generate code consistently across projects.
