# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

- Install: `pip install -r requirements.txt` (Flask 3.1, pytest, pytest-flask)
- Run dev server: `python app.py` (Flask debug mode on http://localhost:5001)
- Tests: none exist yet. `pytest` and `pytest-flask` are pinned in requirements, so tests are expected to be written with pytest. Once they exist, run one with `pytest path/to/test_file.py::test_name`.
- No linter or formatter is configured.

## Architecture

Spendly is a Flask expense-tracker built as a step-by-step project. Much of the app is still scaffolding.

- **`app.py`** holds the Flask app and every route. Routes are grouped into two blocks:
  - Live pages that render templates: `/`, `/register`, `/login`, `/terms`, `/privacy`.
  - Placeholder routes that return plain strings with a "coming in Step N" message: `/logout`, `/profile`, `/expenses/add`, `/expenses/<id>/edit`, `/expenses/<id>/delete`. Replace each one with a real implementation when its step is built.
- **`database/db.py`** is only a stub. Its header comment describes the intended API: `get_db()` (SQLite connection with `row_factory` and foreign keys enabled), `init_db()` (`CREATE TABLE IF NOT EXISTS`), and `seed_db()` (sample data). Nothing imports it yet, so the app has no database layer in use.
- **Templates**: every page extends `templates/base.html`, which provides the navbar, a `content` block, and the footer. Pages fill `{% block title %}`, `{% block head %}` for page-specific CSS, `{% block content %}`, and `{% block scripts %}` for page-specific JS. Links in `base.html` use `url_for(...)`, so route names must match function names in `app.py`.
- **Static assets**: `static/css/style.css` holds the shared theme as CSS variables in `:root` (ink, paper, accent, and so on). `landing.css` is for the landing page only. `static/js/main.js` is a placeholder loaded on every page. `landing.html` keeps its page script inline in the `scripts` block.
- **`file.txt`** is a task brief written as a list of feature requests with suggested commit messages. It is not code. Check it for the intended scope before building a feature.

## Conventions

- Keep JavaScript vanilla. The project uses no frontend framework or libraries.
- Keep changes to the page you were asked to change. Briefs in `file.txt` say explicitly "do not modify anything else."
