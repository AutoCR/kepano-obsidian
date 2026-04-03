# Repository Guidelines

## Project Structure & Module Organization
This repository is an Obsidian vault template, so the primary units are Markdown notes and vault configuration rather than source modules. Core content lives in `Daily/`, `Notes/`, `References/`, `Clippings/`, and top-level `.md` files. Reusable note scaffolds live in `Templates/`, with Obsidian Bases definitions under `Templates/Bases/`. Media and imported files belong in `Attachments/`. Vault behavior is configured in `.obsidian/`; change those files only when you intend to update the shared vault experience.

## Build, Test, and Development Commands
There is no build or test pipeline in this repository. The main development workflow is opening the folder as a vault in Obsidian and validating links, templates, and layout there.

`open -a Obsidian .`
Opens the repository as a local vault on macOS.

`git status`
Checks pending note, template, and config changes before committing.

`rg "term" .`
Searches note titles, frontmatter, and cross-references quickly.

## Coding Style & Naming Conventions
Write in Markdown with clear headings and concise paragraphs. Preserve existing filename patterns: descriptive title-based names such as `Templates/Book Template.md`, date-based daily notes such as `Daily/2026-04-03.md`, and imported references grouped by topic. Use sentence case for headings unless a proper noun requires otherwise. Keep lists readable, prefer relative wiki-style links when editing inside Obsidian, and avoid unnecessary reformatting of existing notes.

## Testing Guidelines
Testing is manual. For note or template changes, open the vault in Obsidian and verify that:

- internal links resolve
- templates insert correctly
- attachments render from `Attachments/`
- changes in `.obsidian/` do not break the expected layout or plugins

If you add a new template, create one sample note from it before submitting.

## Commit & Pull Request Guidelines
Recent history favors short, imperative commit subjects such as `feat: add new notes` and `Update attachments`. Follow that style: start with a verb, keep the subject brief, and group related vault changes together. Pull requests should explain the vault-facing impact, list changed folders, and include screenshots when modifying visual Obsidian behavior or `.obsidian` settings. Link any related issue or discussion when applicable.

## Configuration Tips
Treat `.obsidian/workspace.json` and other personal-state files carefully. Avoid incidental changes caused only by navigation, pane layout, or local plugin state unless the PR intentionally updates shared defaults.
