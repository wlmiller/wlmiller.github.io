# Copilot Instructions for wlmiller.github.io

## Project Overview

- This repository is a Jekyll-based personal website.
- Main source files live at the repo root and in folders like `_layouts`, `_includes`, `_sass`, `articles`, and `assets`.
- Generated output is in `_site` and should be treated as build artifacts, not source of truth.

## Primary Goals

- Preserve the current visual style and information architecture unless explicitly asked to redesign.
- Prefer minimal, targeted edits over broad rewrites.
- Keep content accurate, readable, and professional.

## Source of Truth

- Edit source files, not generated files.
- Do not manually edit files under `_site/` unless the user explicitly asks.
- Prefer editing unminified assets (for example `assets/js/main.js`) over minified files (for example `assets/js/min/main-min.js`).
- Before adding new page content, first check whether the same content is already defined in `_includes/*` or `_layouts/*` and update the canonical source instead of duplicating it.

## Include Ownership and Duplication Prevention

- Treat top-level pages (for example `index.html`) as composition files whenever they render shared sections through includes.
- Treat include files as canonical owners of shared sections. For sidebar/about profile content, `_includes/sidebar.html` is the source of truth.
- Before adding or changing visible content in `index.html`, review `_includes/sidebar.html`, `_layouts/default.html`, and `_layouts/page.html` to confirm where the section is actually defined.
- Do not copy shared section text from includes into page files. If a section already exists in an include, modify the include.
- When adding new copy or headings, search `_includes/` and `_layouts/` for similar text first to avoid duplicate content and drift.

## Jekyll and Content Conventions

- Pages and posts should include valid YAML front matter (`---` blocks) when required by Jekyll.
- Keep existing front matter keys and values unless the change requires updating them.
- Maintain current layout usage (`default`, `page`, `post`) and include structure in `_layouts/` and `_includes/`.
- For markdown content, preserve heading hierarchy and use consistent voice with existing pages.

## HTML/CSS/JS Editing Rules

- Keep HTML semantic and accessible (meaningful headings, alt text for informative images, valid links).
- Reuse existing CSS classes and structure before introducing new styles.
- If adding CSS, place it in the appropriate source stylesheet (for this site, usually `css/main.scss` and `_sass/*`, or existing asset styles in use).
- Avoid large formatting-only changes.
- Keep JavaScript changes small and framework-free unless the user asks otherwise.

## Build and Validation

- Use Bundler-managed Jekyll commands:
	- `bundle install`
	- `bundle exec jekyll serve`
	- `bundle exec jekyll build`
- After meaningful edits, prefer running `bundle exec jekyll build` to catch template/front matter errors.

## File and Path Preferences

- Use relative links and paths consistent with the existing site patterns.
- Keep asset references consistent with current conventions in the edited file.
- Do not rename or move top-level pages without explicit user request.

## Safety and Change Discipline

- Do not remove or rewrite large content sections unless requested.
- If a requested change is ambiguous, choose the smallest safe interpretation and implement it.
- Preserve existing comments and attribution unless they are clearly outdated and the task requires cleanup.

## When Adding New Content

- Match the tone of the surrounding page.
- Keep biographies, publication lists, and project descriptions concise and factual.
- Prefer plain markdown for long-form content and existing HTML structure for pages that already use inline HTML blocks.
