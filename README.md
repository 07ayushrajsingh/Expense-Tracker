# Expense Tracker and Budget Manager

A personal finance web application that records where your money goes and warns you **before** a budget runs out, not after.

Track income and expenses under your own categories, set monthly limits for the whole month or for individual categories, and see a month of spending as charts instead of rows. Every user's data is separate, and budget alerts fire the moment an expense is saved — not at the end of the month when nothing can be done about it.

![Python](https://img.shields.io/badge/Python-3776AB?style=flat&logo=python&logoColor=white)
![Flask](https://img.shields.io/badge/Flask-000000?style=flat&logo=flask&logoColor=white)
![SQLite](https://img.shields.io/badge/SQLite-003B57?style=flat&logo=sqlite&logoColor=white)
![Bootstrap](https://img.shields.io/badge/Bootstrap%205-7952B3?style=flat&logo=bootstrap&logoColor=white)
![Chart.js](https://img.shields.io/badge/Chart.js-FF6384?style=flat&logo=chartdotjs&logoColor=white)

---

## Features

### Money tracking
- Income and expense entries with amount, category, date and an optional note
- Twelve sensible categories created for every new account, plus your own on top
- Filter by date range, type and category — filters live in the URL, so a view can be bookmarked
- Full create, edit and delete flows, with a "save and add another" shortcut for entering a day's expenses in one go

### Budgets that actually warn you
- Overall monthly limits and category-level limits in the same place
- **Amber warning at 80%**, red alert when a limit is crossed
- Alerts appear the moment an expense is saved, not only on the budget page
- One-click copy of last month's budget

### Dashboard
- Balance panel with an income-versus-spent split bar
- Doughnut chart of expenses by category
- Day-wise spending bars for the selected month
- Six-month income versus expense trend
- All charts fed by a JSON API, so the page paints before the data arrives

### Security
- Flask-Login sessions with hashed passwords — plain text is never stored
- **OTP-based password reset** over SMTP, with hashed one-time codes, 10-minute expiry, a 5-attempt limit and a 60-second resend cooldown
- Per-user data isolation on every query, with ownership checks on single-record routes
- Parameterised SQL throughout

---

## Tech Stack

| Layer | Technology |
|---|---|
| Backend | Flask 3 |
| Database | SQLite 3 |
| Authentication | Flask-Login + Werkzeug password hashing |
| Templating | Jinja2 with `DictLoader` |
| Frontend | Bootstrap 5.3 |
| Charts | Chart.js 4.4 |
| Email | smtplib over Gmail SMTP |

---

## Project Structure

```
expense_tracker/
├── app.py                routes, schema, business logic, helpers
├── templates.py          all Jinja2 pages and the stylesheet
├── config.example.py     template for your own config.py
├── expense.db            SQLite database, created on first run (gitignored)
└── requirements.txt
```

Every page lives as a string in the `TEMPLATES` dictionary inside `templates.py` and is loaded through Jinja's `DictLoader`, which keeps the whole project in a handful of files.

---

## Database Schema

| Table | Purpose |
|---|---|
| `users` | Account details and password hash |
| `categories` | Per-user income and expense categories |
| `transactions` | Every income and expense entry |
| `budgets` | Monthly limits — `category_id` is `NULL` for an overall limit |
| `password_resets` | Hashed OTPs with expiry, attempt count and used flag |

Composite indexes on `(user_id, txn_date)` and `(user_id, month)` cover the access path every screen uses.

---

## Setup

### Prerequisites

- Python 3.10 or above

### Installation

```bash
git clone https://github.com/07ayushrajsingh/Expense-Tracker.git
cd Expense-Tracker

python -m venv .venv
.venv\Scripts\activate         # Windows
source .venv/bin/activate      # Linux / macOS

pip install -r requirements.txt
```

### Configuration

Copy `config.example.py` to `config.py` and fill it in:

```python
OTP_MODE = "terminal"       # use "gmail" to send real email
MAIL_USER = ""              # your gmail address
MAIL_PASS = ""              # 16-character gmail app password
```

`terminal` mode prints the OTP in your console and needs no email setup — that's the easiest way to try the reset flow.

For `gmail` mode, enable 2-step verification on the Google account and generate an **app password**; the normal account password will not work with SMTP.

### Run

```bash
python app.py
```

Open **http://127.0.0.1:5000**, register an account, and start adding entries. The database and all tables are created automatically on first run.

---

## Routes

| URL | Method | Purpose |
|---|---|---|
| `/register`, `/login`, `/logout` | GET, POST | Account and session |
| `/forgot-password`, `/reset-password` | GET, POST | OTP-based password reset |
| `/` | GET | Month-wise dashboard |
| `/api/chart-data` | GET | JSON feed for the three charts |
| `/transactions` | GET | Filterable list with totals |
| `/transactions/add`, `/transactions/<id>/edit` | GET, POST | Add and edit entries |
| `/categories` | GET, POST | Manage categories |
| `/budget`, `/budget/copy` | GET, POST | Set and copy monthly limits |

---

## Design Notes

**One table for two kinds of budget.** Leaving `category_id` nullable lets the same table store both the whole-month limit and the per-category limits, with `NULL` read as "Overall".

**Validation at two levels.** Form checks give the user a readable message; `CHECK` constraints in the schema make sure bad data can't enter even if a request reaches the database another way.

**Categories with history are protected.** A category that already has transactions can't be deleted, so past entries never lose their label.

---

## Roadmap

- [ ] Export a monthly statement to Excel or PDF
- [ ] Recurring transactions for rent, EMI and subscriptions
- [ ] Attach a bill photo to an entry
- [ ] Search by note text
- [ ] Email reminder when a budget is crossed
- [ ] Savings goal tracking
- [ ] Dark mode

---

## Author

**Ayush Raj Singh**
