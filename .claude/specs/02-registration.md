# Spec: Registration

## Overview
This step wires up the registration form so users can actually create an account.
`GET /register` already renders the empty form; Step 2 adds the `POST /register`
handler that validates input, hashes the password, writes the new user row, and
redirects to the login page. It also adds a `create_user()` helper to
`database/db.py` and sets `app.secret_key` so Flask flash messages work.

## Depends on
Step 1 — Database setup (`init_db()`, `get_db()`, `users` table). All already
implemented in `database/db.py`.

## Routes
- `POST /register` — validates form data, creates user, redirects to `/login` — public

## Database changes
No new tables or columns. The `users` table (`id`, `name`, `email`,
`password_hash`, `created_at`) is already created by `init_db()`.

New helper added to `database/db.py`:
- `create_user(name, email, password)` — hashes password, inserts row, returns
  the new user `id`. Raises `ValueError` if the email is already taken.
- `get_user_by_email(email)` — returns a single `sqlite3.Row` or `None`.

## Templates
- **Modify:** `templates/register.html`
  - Add `method="POST"` and `action="{{ url_for('register') }}"` to the `<form>`
  - Add `name` attributes to all inputs (`name`, `email`, `password`, `confirm_password`)
  - Render flashed error/success messages above the form

## Files to change
- `app.py` — add `app.secret_key`, import `redirect`, `url_for`, `flash`, `request`;
  convert `register()` into a GET/POST handler
- `database/db.py` — add `create_user()` and `get_user_by_email()`
- `templates/register.html` — wire up the form and flash block
- `static/css/register.css` — create if page-specific styles are needed (can be
  empty to start)

## Files to create
- `static/css/register.css` — page-specific stylesheet (link from `register.html`)

## New dependencies
No new dependencies. `werkzeug.security.generate_password_hash` and
`werkzeug.security.check_password_hash` are already available via Flask's
dependency on Werkzeug.

## Rules for implementation
- No SQLAlchemy or ORMs — raw `sqlite3` only
- Parameterised queries only — never f-strings in SQL
- Passwords hashed with `werkzeug.security.generate_password_hash`
- Use CSS variables — never hardcode hex values
- All templates extend `base.html`
- `app.secret_key` must be set before any `flash()` call; use a hard-coded dev
  secret (`"dev-secret-change-in-prod"`) — do not read from env unless asked
- Duplicate email must return a flashed error and re-render the form (HTTP 200),
  not abort
- Password mismatch (confirm ≠ password) must be caught server-side, not only
  client-side
- After a successful registration redirect to `url_for('login')` — do **not**
  auto-login the user (that's Step 3)
- Do **not** implement `GET /logout` or any other stub route

## Definition of done
- [ ] Submitting the form with a new name/email/matching passwords creates a row
      in `users` and redirects to `/login`
- [ ] Submitting with an email that already exists re-renders the form with a
      visible error message (no duplicate row created)
- [ ] Submitting with mismatched passwords re-renders the form with a visible
      error message
- [ ] Submitting with any blank field re-renders the form with a visible error
      message
- [ ] The password stored in the DB is a Werkzeug hash, never plaintext
- [ ] `/register` (GET) still renders the empty form with no regression
- [ ] No other stub routes are changed
