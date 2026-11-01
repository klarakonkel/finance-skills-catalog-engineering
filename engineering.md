# App: Catalog of Finance Skills - Instructions for Engineering

If something is unclear in this file or seems wrong, flag before acting on it/coming up with a different design.

## 1. Stack - USE ONLY THESE, NO EXTERNAL TECHNOLOGIES ALLOWED
- Python, Flask
- HTML + plain CSS
- SQLite via Python's 'sqlite3' module, or sqlalchemy
- Flask-Login, werkzeug for password hashing
- NO external APIs

## 2. File structure

```
app/
  __init__.py          create_app(): config, db teardown, Flask-Login, register blueprints
  config.py            settings from environment variables, with safe local defaults
  db.py                get_db() (one connection per request, stored on g), close_db(), init_db(); enables PRAGMA foreign_keys=ON
  schema.sql           all CREATE TABLE / CREATE INDEX statements
  models.py            User class for Flask-Login (UserMixin); no SQL here
  repositories/        ALL SQL lives here; functions only, no Flask imports
    users.py
    skills.py
    tags.py
    favourites.py
  services/
    permissions.py     can_view(user, skill), can_edit(user, skill), is_admin(user); the ONLY place access rules live
    skills.py          business rules: validation, status transitions, saving a skill + its tags in one transaction
  routes/              Flask blueprints; handle HTTP + forms; call services/repositories; never contain SQL
    auth.py            register, login, logout
    catalog.py         public listing, filters, skill detail, .md download
    my_skills.py       create / edit / delete / submit own skills
    favourites.py      add / remove / list favourites
    admin.py           review queue, approve, reject
  templates/           Jinja2 only; no business logic beyond loops/conditionals
  static/style.css
scripts/
  seed.py              creates the database from schema.sql, imports seed/, creates the admin user
seed/                  provided dataset (skills.json + skills/<id>.md) — do not modify
tests/
  test_auth.py
  test_permissions.py
  test_skills.py
README.md              setup and run instructions
requirements.txt
```

## 3. Database schema

Table: Users
Columns:
Id 	INTEGER PK
Username 	TEXT UNIQUE NOT NULL,
Password_hash	TEXT NOT NULL,
Preferred_name	TEXT
Role	TEXT NOT NULL DEFAULT ‘user’ (CHECK (role IN (‘user’, ‘admin’))

Table: Skills
Id	INTEGER PK
owner_id INTEGER FK TO Users(Id)
Status	TEXT NOT NULL CHECK (status in “draft”, “pending”, “approved”, “rejected”)
Title	TEXT NOT NULL,
Description	TEXT, #this will be a tl/dr: what the skill does, what the inputs are, what it produces
Content	TEXT, #this will be the full SKILL.md text (stored directly instead of a file path)
Author		TEXT #display name, for example github username
Source_url	TEXT NULL #link to github for imported skills,
License	TEXT
Created_at, updated_at

Table: favourites
Columns:
User_id	INTEGER FK to users(id)
Skill_id		INTEGER FK to skills(id)
PRIMARY KEY (user_id, skill_id)

Table: tags
Columns:
id 	INTEGER PK,
Name	TEXT NOT NULL, #for example “Accounting” or “Tax”, or “Personal Finance” or “small business” or “EU”, “USA” etc
kind 	TEXT NOT NULL CHECK (kind in ‘category’, ‘jurisdiction’)
UNIQUE (name, kind)


Table: skills_tags
Columns:
Skill_id 	INTEGER FK to skills(id) ON DELETE CASCADE
Tag_id		INTEGER FK to tags(id) 
PRIMARY KEY (skill_id, tag_id)

CHECK: if the status is “draft” then it’s fine for description, content, category, and jurisdiction. to be missing (for now) but once it is submitted for approval, all the fields are required

## 4. Routes
| Method | Path | Access | Purpose |
|---|---|---|---|
| GET | `/` | anyone | published skills with filters and search |
| GET | `/skills/<id>` | can_view | detail page: title, author, tags, what/inputs/output, content in `<pre>`, source + license if present |
| GET | `/skills/<id>/download` | can_view | returns content as `SKILL.md` (`text/markdown`, attachment) |
| GET, POST | `/register` | anonymous | create account |
| GET, POST | `/login` | anonymous | log in |
| POST | `/logout` | logged in | log out |
| GET | `/my/skills` | logged in | own skills, all statuses, with status shown |
| GET, POST | `/my/skills/new` | logged in | create draft |
| GET, POST | `/my/skills/<id>/edit` | owner | edit (applies status rules) |
| POST | `/my/skills/<id>/submit` | owner | draft → pending |
| POST | `/my/skills/<id>/delete` | owner | delete |
| GET | `/my/favourites` | logged in | favourites list |
| POST | `/skills/<id>/favourite` | logged in + can_view | add |
| POST | `/skills/<id>/unfavourite` | logged in | remove |
| GET | `/admin/review` | admin | pending skills |
| POST | `/admin/skills/<id>/approve` | admin | pending → published |
| POST | `/admin/skills/<id>/reject` | admin | pending → rejected |

## 5. Milestones

1. Database and seed script
2. Login and registration
3. Public catalog and filters
4. My skills (create, edit, delete)
5. Favourites
6. Admin review

After each milestone: summarise what changed, list files touched, and how to try it manually.

## 6. Definition of done

- starts with `flask --app run` 
- application behaves as expected (anyone can browse published skills, logged-in users can favourite skills and create their own private skills, which they can submit for review; admin can approve or reject submissions)
- all routes behave as specified