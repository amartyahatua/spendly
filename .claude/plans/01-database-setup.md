# Plan: 01 – Database Setup

## Context
`database/db.py` is a comment-only stub and nothing imports it, so Spendly has no data layer. Spec (`.claude/specs/database-setup.md`) asks for a SQLite layer (`get_db`, `init_db`, `seed_db`) wired into `app.py` startup. Auth, profile and expenses all depend on it. No new routes, no new packages.

Note: the requested spec path `01-database-setup.md` doesn't exist; the spec is `.claude/specs/database-setup.md`.

## Files
- Modify `database/db.py` (implement all three functions)
- Modify `app.py` (imports + startup calls only; routes untouched)
- No new files. `database/` has no `__init__.py`; `from database.db import ...` works as a namespace package when run from the project root.

## `database/db.py`
- `DB_PATH` = `spendly.db` in project root (computed from `__file__`); `CATEGORIES` list of the 7 fixed categories.
- `get_db()`: `sqlite3.connect(DB_PATH)`, `row_factory = sqlite3.Row`, `PRAGMA foreign_keys = ON`.
- `init_db()`: `CREATE TABLE IF NOT EXISTS` for `users` and `expenses` per spec; commit, close.
- `seed_db()`: returns early if `users` has rows; otherwise inserts Demo User (`demo@spendly.com`, password `demo123` hashed with `werkzeug.security`) and 8 expenses (one per category plus an extra Food) dated on days 1–8 of the current month. Parameterized `executemany`.

## `app.py`
- `from database.db import get_db, init_db, seed_db`
- After `app = Flask(__name__)`: `with app.app_context(): init_db(); seed_db()`.

## Verification (done)
- Import creates `spendly.db`; schema matches spec; 1 user, 8 expenses, all 7 categories, dates `YYYY-MM-DD` in current month.
- Re-running the import leaves counts unchanged (no duplicate seed).
- Demo password hash verifies with `check_password_hash`.
- Duplicate email → `IntegrityError: UNIQUE constraint failed`; expense with `user_id=999` → `IntegrityError: FOREIGN KEY constraint failed`.
- Test client: `/`, `/login`, `/register`, `/terms`, `/privacy`, `/logout` all return 200.

## Open points
- No `.gitignore` exists, so `spendly.db` shows as untracked. Not added, since it's outside the spec's files list.
- No tests added (spec doesn't require them).