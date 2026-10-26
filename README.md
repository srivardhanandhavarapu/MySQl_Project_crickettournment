# 🏏 Cricket Tournament Management System

A complete **SQL-only database project** that stores and analyses everything about a cricket tournament: coaches, teams, players, stadiums, tournaments, team registrations, matches, innings and every player's batting and bowling performance.

**Database name:** `Crickettournment_db`  
**Database:** MySQL 8.0  
**Type:** Pure SQL project (no front-end, no CTEs)

---

## 📌 Table of Contents

1. [Project Overview](#-project-overview)
2. [Objectives](#-objectives)
3. [Tools and Technologies](#-tools-and-technologies)
4. [ER Diagram](#-er-diagram)
5. [Database Design](#-database-design)
6. [Relationships](#-relationships)
7. [Sample Data](#-sample-data)
8. [SQL Concepts Covered](#-sql-concepts-covered)
9. [Sample Queries and Output](#-sample-queries-and-output)
10. [Practice Questions](#-practice-questions)
11. [How to Run](#-how-to-run)
12. [Repository Structure](#-repository-structure)
13. [Future Scope](#-future-scope)
14. [Author](#-author)

---

## 📖 Project Overview

Organising a cricket tournament involves a lot of connected information: which teams are taking part, who coaches them, which players are in each team, where each match is played, who won, and how every player performed. Keeping this in spreadsheets quickly becomes messy and error-prone.

This project solves that with a **normalised relational database** of **9 linked tables**. The database lets you answer questions such as:

- Who is the top run scorer of the tournament?
- Which team has won the most matches, and what does the points table look like?
- Which bowlers have taken 5 or more wickets?
- Which matches were played at a given stadium, and who won them?
- Which teams have registered for which tournament, and how much fee have they paid?

---

## 🎯 Objectives

- Design a proper database from scratch using an **ER diagram**.
- Convert the ER diagram into tables using **primary keys, foreign keys and constraints**.
- Handle many-to-many relationships using **linking tables** (`registrations`, `player_stats`).
- Insert realistic sample data (30 players, 11 matches, 41 player performances and more).
- Write queries from **basic to advanced** level to extract useful tournament insights.
- Practise advanced SQL objects: **views, a stored function, a stored procedure, a trigger and transactions**.

---

## 🛠 Tools and Technologies

| Tool | Purpose |
|------|---------|
| MySQL 8.0 | Relational database |
| MySQL Workbench (or any SQL client) | Writing and running queries |
| SQL | DDL, DML, DQL, joins, views, procedures, triggers |
| Git and GitHub | Version control and hosting |

---

## 🗺 ER Diagram

![ER Diagram](ER_Diagram_Crickettournment_db.png)

**Notation:** rectangle = entity, ellipse = attribute (underlined = primary key), diamond = relationship, `1` / `N` = cardinality.

---

## 🗄 Database Design

The database contains **9 tables**.

### 1. `coaches`
| Column | Type | Notes |
|--------|------|-------|
| coach_id | INT | Primary key, auto increment |
| coach_name | VARCHAR(50) | Not null |
| experience_years | INT | CHECK >= 0 |
| specialization | VARCHAR(30) | Batting / Bowling / Fielding / All-round |
| phone | VARCHAR(15) | Unique |
| email | VARCHAR(80) | Added using `ALTER TABLE` |

### 2. `teams`
| Column | Type | Notes |
|--------|------|-------|
| team_id | INT | Primary key |
| team_name | VARCHAR(50) | Not null, unique |
| city | VARCHAR(60) | Not null |
| founded_year | INT | |
| coach_id | INT | Foreign key → `coaches`, unique (one coach per team) |

### 3. `stadium`
| Column | Type | Notes |
|--------|------|-------|
| stadium_id | INT | Primary key |
| stadium_name | VARCHAR(60) | Not null |
| city | VARCHAR(50) | Not null |
| capacity | INT | CHECK > 0 |

### 4. `tournaments`
| Column | Type | Notes |
|--------|------|-------|
| tournament_id | INT | Primary key |
| tournament_name | VARCHAR(60) | Not null |
| season_year | INT | Not null |
| start_date, end_date | DATE | CHECK end_date >= start_date |
| prize_money | DECIMAL(12,2) | Default 0 |

### 5. `registrations` *(linking table: Teams ↔ Tournaments)*
| Column | Type | Notes |
|--------|------|-------|
| registration_id | INT | Primary key |
| tournament_id | INT | Foreign key → `tournaments` |
| team_id | INT | Foreign key → `teams` |
| reg_date | DATE | Not null |
| fee_paid | DECIMAL(10,2) | Default 0 |
| status | ENUM | Pending / Confirmed / Rejected |

`UNIQUE (tournament_id, team_id)` makes sure a team cannot register twice for the same tournament.

### 6. `players`
| Column | Type | Notes |
|--------|------|-------|
| player_id | INT | Primary key |
| player_name | VARCHAR(50) | Not null |
| age | INT | CHECK >= 15 |
| role | ENUM | Batsman / Bowler / All-rounder / Wicket-keeper |
| batting_style | VARCHAR(20) | Right-hand / Left-hand |
| jersey_no | INT | Unique within a team |
| team_id | INT | Foreign key → `teams` |

### 7. `matches`
| Column | Type | Notes |
|--------|------|-------|
| match_id | INT | Primary key |
| tournament_id | INT | Foreign key → `tournaments` |
| team1_id, team2_id | INT | Foreign keys → `teams`, CHECK team1 ≠ team2 |
| stadium_id | INT | Foreign key → `stadium` |
| match_date | DATE | Not null |
| match_type | ENUM | League / Semi-Final / Final |
| winner_team_id | INT | Foreign key → `teams`, `NULL` for an upcoming match |
| player_of_match | INT | Foreign key → `players` |

### 8. `innings`
| Column | Type | Notes |
|--------|------|-------|
| innings_id | INT | Primary key |
| match_id | INT | Foreign key → `matches` |
| innings_no | INT | 1 or 2 |
| batting_team_id, bowling_team_id | INT | Foreign keys → `teams` |
| total_runs | INT | |
| wickets_lost | INT | CHECK between 0 and 10 |
| overs | DECIMAL(3,1) | |

### 9. `player_stats` *(linking table: Players ↔ Matches)*
| Column | Type | Notes |
|--------|------|-------|
| stat_id | INT | Primary key |
| match_id | INT | Foreign key → `matches` |
| player_id | INT | Foreign key → `players` |
| runs, balls_faced, fours, sixes | INT | Batting performance |
| wickets, overs_bowled, runs_conceded | INT / DECIMAL | Bowling performance |

`UNIQUE (match_id, player_id)` allows only one stats row per player per match.

---

## 🔗 Relationships

| Relationship | Entities | Type | Implemented using |
|--------------|----------|------|-------------------|
| Manages | Coaches – Teams | 1 : 1 | `teams.coach_id` (unique FK) |
| Has | Teams – Players | 1 : N | `players.team_id` |
| Registers / Receives | Teams – Tournaments | M : N | `registrations` table |
| Contains | Tournaments – Matches | 1 : N | `matches.tournament_id` |
| Plays | Teams – Matches | M : N | `matches.team1_id`, `team2_id`, `winner_team_id` |
| Hosts | Stadium – Matches | 1 : N | `matches.stadium_id` |
| Has | Matches – Innings | 1 : N | `innings.match_id` |
| Performs / Records | Players – Matches | M : N | `player_stats` table |

---

## 📊 Sample Data

| Table | Records |
|-------|---------|
| coaches | 6 |
| teams | 6 |
| stadium | 4 |
| tournaments | 2 |
| registrations | 12 |
| players | 30 |
| matches | 11 (10 completed, 1 upcoming final) |
| innings | 20 |
| player_stats | 41 |

**Teams:** Chennai Strikers, Mumbai Warriors, Hyderabad Hawks, Delhi Dynamos, Kolkata Knights, Bengaluru Blasters  
**Tournaments:** Summer Premier Cup (2025) and Champions Trophy (2026)  
**Stadiums:** MA Chidambaram Stadium, Wankhede Stadium, Rajiv Gandhi Stadium, Eden Gardens

---

## 🧠 SQL Concepts Covered

| Category | Topics |
|----------|--------|
| **DDL** | `CREATE DATABASE`, `CREATE TABLE`, `ALTER TABLE`, `CREATE INDEX`, `DROP` |
| **Constraints** | `PRIMARY KEY`, `FOREIGN KEY`, `NOT NULL`, `UNIQUE`, `CHECK`, `DEFAULT`, `AUTO_INCREMENT`, `ENUM`, composite `UNIQUE` |
| **DML** | `INSERT`, `UPDATE`, `DELETE` |
| **Filtering** | `WHERE`, `AND / OR`, `LIKE`, `IN`, `BETWEEN`, `IS NULL`, `DISTINCT`, `ORDER BY`, `LIMIT` |
| **Joins** | `INNER JOIN`, `LEFT JOIN`, multi-table joins, `SELF JOIN` |
| **Aggregation** | `COUNT`, `SUM`, `AVG`, `MAX`, `MIN`, `GROUP BY`, `HAVING` |
| **Subqueries** | Scalar, `IN`, `NOT EXISTS`, correlated subquery |
| **Set operations** | `UNION` |
| **Functions** | `CASE`, string functions, date functions, `ROUND`, `IF` |
| **Window function** | `RANK() OVER` *(optional)* |
| **Views** | `v_match_summary`, `v_points_table`, `v_player_career` |
| **Stored objects** | Function `fn_strike_rate`, procedure `sp_team_report`, trigger `trg_matches_check` |
| **Transactions** | `START TRANSACTION`, `COMMIT`, `ROLLBACK` |

> ℹ️ This project deliberately does **not** use CTEs (`WITH ... AS`).

---

## 🔍 Sample Queries and Output

### 1. Top 5 run scorers (JOIN + GROUP BY)
```sql
SELECT p.player_name, SUM(ps.runs) AS total_runs
FROM player_stats ps
JOIN players p ON ps.player_id = p.player_id
GROUP BY p.player_name
ORDER BY total_runs DESC
LIMIT 5;
```
| player_name | total_runs |
|-------------|-----------:|
| Arjun Reddy | 286 |
| Sourav Das | 198 |
| Rahul Singh | 184 |
| Rohit Verma | 180 |
| Kiran Gowda | 159 |

### 2. Points table with rank (VIEW + RANK)
```sql
CREATE VIEW v_points_table AS
SELECT t.team_id, t.team_name,
       COUNT(m.match_id) AS wins,
       COUNT(m.match_id) * 2 AS points
FROM teams t
LEFT JOIN matches m ON t.team_id = m.winner_team_id
GROUP BY t.team_id, t.team_name;

SELECT team_name, points,
       RANK() OVER (ORDER BY points DESC) AS team_rank
FROM v_points_table;
```
| team_name | points | team_rank |
|-----------|-------:|----------:|
| Chennai Strikers | 6 | 1 |
| Hyderabad Hawks | 4 | 2 |
| Kolkata Knights | 4 | 2 |
| Bengaluru Blasters | 2 | 4 |
| Delhi Dynamos | 2 | 4 |
| Mumbai Warriors | 2 | 4 |

### 3. Bowlers with 5 or more wickets (HAVING)
```sql
SELECT p.player_name, SUM(ps.wickets) AS total_wickets
FROM player_stats ps
JOIN players p ON ps.player_id = p.player_id
GROUP BY p.player_name
HAVING SUM(ps.wickets) >= 5
ORDER BY total_wickets DESC;
```
| player_name | total_wickets |
|-------------|--------------:|
| Karthik Raj | 7 |
| Pritam Roy | 6 |
| Deepak Yadav | 5 |
| Naveen Shetty | 5 |

### 4. Trigger that stops invalid data
```sql
CREATE TRIGGER trg_matches_check
BEFORE INSERT ON matches
FOR EACH ROW
BEGIN
    IF NEW.team1_id = NEW.team2_id THEN
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'A team cannot play against itself';
    END IF;
END;
```
Trying to insert a match where `team1_id = team2_id` fails with the message *"A team cannot play against itself"*.

---

## 📝 Practice Questions

`Crickettournment_db_15_Questions.sql` contains **15 questions** (5 Basic, 5 Intermediate, 5 Advanced). Each one has the question in plain English, the SQL answer and the expected output.

| Level | Topics |
|-------|--------|
| **Basic** | WHERE with AND, LIKE, BETWEEN, IN with LIMIT, IS NULL |
| **Intermediate** | INNER JOIN, multiple JOINs, GROUP BY with SUM, HAVING, LEFT JOIN with COUNT |
| **Advanced** | Scalar subquery, correlated subquery, SELF JOIN, VIEW with RANK, stored FUNCTION |

---

## ▶️ How to Run

1. **Install MySQL 8.0** and MySQL Workbench.
2. **Clone the repository**
   ```bash
   git clone https://github.com/<your-username>/Crickettournment_db.git
   cd Crickettournment_db
   ```
3. **Run the main script** (creates the database, tables, data, queries, views and procedures)
   - In Workbench: `File → Open SQL Script → Crickettournment_db.sql → ⚡ Execute`
   - Or from the terminal:
     ```bash
     mysql -u root -p < Crickettournment_db.sql
     ```
4. **Run the practice questions**
   ```bash
   mysql -u root -p < Crickettournment_db_15_Questions.sql
   ```
5. Select the database in your client if needed: `USE Crickettournment_db;`

> The script starts with `DROP DATABASE IF EXISTS Crickettournment_db;`, so running it again resets everything to a clean state.

---

## 📁 Repository Structure

```
Crickettournment_db/
├── Crickettournment_db.sql                 # Full project: tables, data, queries, views, function, procedure, trigger
├── Crickettournment_db_15_Questions.sql    # 15 practice questions with answers and expected output
├── ER_Diagram_Crickettournment_db.png      # ER diagram
├── Cricket_Tournament_SQL_Project.pptx     # Project presentation (15 slides)
└── README.md                               # Project documentation
```

---

## 🚀 Future Scope

- Build a front-end (web or mobile app) on top of this database.
- Add a live scoreboard with ball-by-ball data.
- Automate the points table with triggers after every match result.
- Add more seasons, knockout brackets and player auction data.
- Create stored procedures for player and team career reports.
- Add user roles and permissions (admin, scorer, viewer).

---

## 👤 Author

**Your Name**  
SQL Project: Cricket Tournament Management System  
GitHub: [@your-username](https://github.com/your-username)

⭐ If you found this project useful, please give it a star!
