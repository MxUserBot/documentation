# Building documentation with MkDocs

MkDocs turns a folder of .md files into a beautiful static site.

## Installation

```bash
pip install mkdocs mkdocs-material
```

## Config

Create `mkdocs.yml` in the project root:

```yaml
site_name: MXUserbot Docs
theme:
  name: material
  palette:
    - scheme: default
      toggle:
        icon: material/toggle-switch-off-outline
        name: Switch to dark mode
    - scheme: slate
      toggle:
        icon: material/toggle-switch
        name: Switch to light mode
nav:
  - Home: index.md
  - How it works: 2-how-it-works.md
  - Pros/Cons: 6-pros-cons.md
  - Development:
    - Key concepts: dev/0-key-concepts.md
    - SAS verification: dev/1-sas-verification.md
    - Emoji callbacks: dev/3-emoji-callbacks.md
    - Writing modules:
      - Introduction: dev/writing-modules/00-index.md
      - Basics: dev/writing-modules/01-basics.md
      - Commands: dev/writing-modules/02-commands.md
      - Answer: dev/writing-modules/03-answer.md
      - Watcher/Cron/Events: dev/writing-modules/04-watcher-cron-events.md
      - FSM: dev/writing-modules/05-fsm.md
    - Security: dev/5-security.md
    - ZIP modules: dev/7-zip-modules.md
    - Utils reference: dev/8-utils-reference.md
  - System modules:
    - Ping: system_modules/01-ping.md
    - Help: system_modules/02-help.md
    - Prefix: system_modules/03-set_prefix.md
    - Sudo: system_modules/04-sudo.md
    - Verif: system_modules/05-verif.md
    - Shell: system_modules/06-shell.md
    - Loader: system_modules/07-loader.md
```

## Running

```bash
# dev server with auto-reload
mkdocs serve

# build static site to site/
mkdocs build
```

## Deploy

```bash
# to GitHub Pages
mkdocs gh-deploy

# or just upload site/ to any hosting
```

## Why MkDocs + Material

- **Zero-config** compared to Docusaurus — no React/Node needed
- .md files **as-is** — nothing to change
- Nested folders supported (`dev/`, `system_modules/`)
- Dark/light theme out of the box
- Full-text search
- Tons of plugins: diagrams, formulas, versioning
- Generates clean static files — host anywhere
