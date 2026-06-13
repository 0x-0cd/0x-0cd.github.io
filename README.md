# 0x-0cd.github.io

[![Website](https://img.shields.io/website?url=https%3A%2F%2F0x-0cd.github.io&style=flat-square&label=status&color=008c8c)](https://0x-0cd.github.io)
[![Hugo](https://img.shields.io/badge/Hugo-0.163.1-ff4088?style=flat-square&logo=hugo)](https://gohugo.io)
[![Theme](https://img.shields.io/badge/Theme-PaperMod-008c8c?style=flat-square)](https://github.com/adityatelange/hugo-PaperMod)
[![License](https://img.shields.io/badge/License-MIT-yellow?style=flat-square)](LICENSE)

Personal site & blog built with **Hugo** and **PaperMod** theme. Profile mode, teal accents, dark/light adaptive, Mermaid.js support.

## 🚀 Tech Stack

| | |
|---|---|
| **Framework** | [Hugo](https://gohugo.io) v0.163.1 |
| **Theme** | [PaperMod](https://github.com/adityatelange/hugo-PaperMod) (git submodule) |
| **Deployment** | GitHub Actions → GitHub Pages |
| **Search** | Fuse.js (client-side, no backend) |
| **Languages** | Markdown + Goldmark (allowUnsafe HTML) |

## 📂 Structure

```
.
├── assets/          # Custom CSS/JS
├── content/         # Markdown posts & pages
│   ├── _index.md    # Home page (profile mode)
│   ├── posts/       # Blog posts / notes
│   ├── archives/    # Archive layout
│   └── search/      # Search page
├── layouts/         # Custom overrides
├── static/          # Static assets (images, etc.)
├── themes/          # PaperMod submodule
└── hugo.yaml        # Site config
```

## 🛠 Local Development

```bash
# Clone with submodules
git clone --recurse-submodules git@github.com:0x-0cd/0x-0cd.github.io.git

# Or update submodules after clone
git submodule update --init --recursive

# Start dev server
hugo server -D

# Build production
hugo --minify
```

## 🤖 Co-crafted

This site is maintained with the help of [Emma](https://github.com/0x-0cd/0x-0cd) — an AI agent with a soul 🥹

> Site avatar by Emma • Powered by Hugo + ☕
