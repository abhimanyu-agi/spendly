# Spec: Login and Logout

## Overview
This step makes authentication functional. `GET /login` already renders the form shell;
Step 3 adds the `POST /login` handler that validates credentials against the database,
sets a Flask session on success, and redirects the user into the app. It also implements
`GET /logout` (currently a string stub) to clear the session and redirect to the landing
page. Together these two routes complete the full login/logout cycle that all
authenticated features (Step 4 onward) depend on.

## Depends on
- Step 1 — Database setup (`get_db()`, `users` table, `get_user_by_email()`)
- Step 2 — Registration (`create_user()`, hashed passwords stored in DB)

## Routes
- `POST /login` — validates email + password, sets `session['user_id']` and
  `session['user_name']`, redirects to `/` on success — public
- `GET /logout` — clears the session, flashes a goodbye message, redirects to `/` — public

## Database changes
No new tables or columns. `get_user_by_email(email)` already exists in `database/db.py`
and returns a `sqlite3.Row` with `id`, `name`, `email`, `password_hash`.

## Templates
- **Modify:** `templates/login.html`
  - Add `method="POST"` and `action="{{ url_for('login') }}"` to the `<form>`
  - Add `name` attributes to inputs: `email`, `password`
  - Render flashed error messages above the form
- **Modify:** `templates/base.html`
  - Add logout link (visible only when `session.user_id` is set) to the nav

## Files to change
- `app.py`
  - Import `session` from `flask` (add to existing import line)
  - Import `check_password_hash` from `werkzeug.security` (or use the helper already
    imported via `db.py` — add the import directly in `app.py`)
  - Convert `login()` to a GET/POST handler
  - Implement `logout()` — clear session, flash message, redirect to `/`
- `templates/login.html` — wire up the form and flash block
- `templates/base.html` — add conditional logout link in nav

## Files to create
- `static/css/login.css` — page-specific stylesheet (linked from `login.html`);
  can be empty to start, styles added as needed

## New dependencies
No new dependencies. `werkzeug.security.check_password_hash` is already available
via Flask's dependency on Werkzeug.

## Rules for implementation
- No SQLAlchemy or ORMs — raw `sqlite3` only
- Parameterised queries only — never f-strings in SQL
- Password verification with `werkzeug.security.check_password_hash` only —
  never compare plaintext passwords
- Use CSS variables — never hardcode hex values
- All templates extend `base.html`
- On failed login (wrong email or wrong password) show the **same generic error**
  ("Invalid email or password.") for both cases — do not reveal which field was wrong
- After successful login redirect to `url_for('landing')` — profile redirect comes
  in Step 4 once that page is implemented
- `logout()` must call `session.clear()` (not `session.pop` per key) and redirect
  to `url_for('landing')`
- Do **not** implement any other stub route (`/profile`, `/expenses/add`, etc.)
- Session key names must be `user_id` (int) and `user_name` (str) — used by Step 4+

## Definition of done
- [ ] Submitting valid credentials sets the session and redirects to `/`
- [ ] Submitting a non-existent email shows "Invalid email or password." and
      re-renders the login form (no 500, no stack trace)
- [ ] Submitting a correct email with the wrong password shows the same generic
      error and re-renders the form
- [ ] Submitting with any blank field shows a validation error and re-renders
      the form without hitting the database
- [ ] Visiting `/logout` clears the session and redirects to `/`
- [ ] After logout, visiting `/logout` again is safe (no KeyError, session already empty)
- [ ] The nav in `base.html` shows a logout link only when a user is logged in
- [ ] No other stub routes are changed
