# App: Catalog of Finance Skills - Instructions for Engineering

If something is unclear in this file or seems wrong, flag before acting on it/coming up with a different design.

## 1. Stack - USE ONLY THESE, NO EXTERNAL TECHNOLOGIES ALLOWED
- Python, Flask
- HTML + plain CSS
- SQLite via Python's 'sqlite3' module, or sqlalchemy
- Flask-Login, werkzeug for password hashing
- NO external APIs

## 2. File structure
routes/controllers - to handle HTTP
Data access - SQL, one module per table
Auth - login, sessions, check: “can this user be able to view/edit this skill”
Templates/views - in HTML
Database setup - schema file + seed script that imports /seed
Config (database path, port etc)



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

Method & path | Access |Purpose
GET / | anyone | Published skills, with filters
GET /skills/:id | published, owner or admin | Skill detail
POST /skills/:id/edit | owner only | Save changes; published -> pending
POST /admin/skills/:id/approve | admin only | Publish a pending skill

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