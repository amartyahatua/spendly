# Spec: Registration

## Overview
This step makes the existing `/register` page work. Today the page renders and its form posts to `/register`, but the route only accepts GET, so submitting does nothing. This step adds POST handling that validates the form, hashes the password, stores a new row in `users`, and sends the visitor to the login page. It is the first step that writes user data, and it gives later steps (login, logout, profile, expenses) a real account to work with.

## Depends on
- Step 1 — Database setup: `get_db()`, `init_db()` and the `users` table must exist and work.

## Routes
- `GET /register` — renders the registration form (existing behaviour, unchanged) — public
- `POST /register` — validates the submitted name, email and password, creates the user, and redirects to `/login` on success; re-renders the form with an error on failure — public

## Database changes
No schema changes. The existing `users` table in `database/db.py` already has `name`, `email` (UNIQUE, NOT NULL), `password_hash` (NOT NULL) and `created_at`.

Add one helper to `database/db.py`:
- `create_user(name, email, password)` — hashes the password with `generate_password_hash`, inserts the row with a parameterised `INSERT`, commits, and closes the connection. It raises `sqlite3.IntegrityError` if the email already exists. The caller handles that error.

## Templates
- **Create:** none
- **Modify:**
  - `templates/register.html` — keep the `name`, `email` values after a failed submit (`value="{{ name or '' }}"` and `value="{{ email or '' }}"`), so the user does not retype them. Do not change the markup otherwise.

## Files to change
- `app.py` — change `/register` to accept `GET` and `POST`. On POST: read and strip form fields, validate them, call `create_user`, handle duplicate email, redirect to `login` on success. Pass `error`, `name` and `email` back to the template on failure.
- `database/db.py` — add `create_user()`.
- `templates/register.html` — preserve entered values (see above).

## Files to create
- None

## New dependencies
No new dependencies. Uses `werkzeug.security` (already installed), `sqlite3`, and Flask's `request` and `redirect`/`url_for`.

## Rules for implementation
- No SQLAlchemy or ORMs
- Parameterised queries only — no string formatting in SQL
- Passwords hashed with werkzeug (`generate_password_hash`); never store or log the plain password
- Use CSS variables — never hardcode hex values
- All templates extend `base.html`
- Validation rules:
  - `name`, `email`, `password` are all required (after stripping whitespace on name and email)
  - `email` is lowercased and stripped before storage and before the duplicate check
  - `email` must contain `@` and a `.` after it (basic check; the `type="email"` input is not enough on its own)
  - `password` must be at least 8 characters
  - Duplicate email shows the error "An account with that email already exists." and does not raise a 500
- Error messages are shown through the existing `{% if error %}` block in `register.html`
- Keep changes to `/register` only. Do not change the other placeholder routes or the other pages.
- Do not auto-login after registration. Redirect to `/login`.

## Definition of done
- [ ] `GET /register` still renders the form with status 200
- [ ] Submitting a valid name, email and 8+ character password redirects to `/login`
- [ ] After a successful registration, the new row exists in `users` with the email in lowercase
- [ ] The stored `password_hash` is not equal to the entered password and starts with a werkzeug hash prefix (e.g. `scrypt:` or `pbkdf2:`)
- [ ] Submitting an email that already exists re-renders the form with "An account with that email already exists." and does not return a 500 error
- [ ] Submitting with any empty field re-renders the form with an error and creates no row
- [ ] Submitting a password shorter than 8 characters re-renders the form with an error and creates no row
- [ ] Submitting an invalid email (e.g. `abc`) re-renders the form with an error and creates no row
- [ ] After a failed submit, the name and email fields still contain the values that were typed
- [ ] All SQL in `create_user` uses `?` placeholders
- [ ] `python app.py` starts without errors and the existing pages (`/`, `/login`, `/terms`, `/privacy`) still load
