---
toc: true
layout: post
title: Trimester 2 Retrospective
description: My work on the database over tri2
permalink: /tri2retrospective/retrospective
---

## Analytics

<img width="781" height="212" alt="Image" src="https://github.com/user-attachments/assets/27f6a1e2-61ad-4ddf-b8aa-b55200b5a639" />

Numerous pull requests, commits, and merges across both Flask and Spring repositories. The focus was on database reliability, data migration tooling, and the new database automator project.

[Flask Pull Requests](https://github.com/Open-Coding-Society/flask/pulls?q=is%3Apr+is%3Aclosed+author%3Ailluminati1618)

[Spring Pull Requests](https://github.com/Open-Coding-Society/spring/pulls?q=is%3Apr+is%3Aclosed+author%3Ailluminati1618), and [Spring Pull Requests (with Mihir)](https://github.com/Open-Coding-Society/spring/pulls?q=is%3Apr+is%3Aclosed+author%3AHypernova101)

[Database Automator Progress Commits](https://github.com/illuminati1618/tracking-database-automator)

## Overview

This trimester, my work centered on **database reliability and data protection** for the Open Coding Society's Flask and Spring backends. The work fell into three major chapters:

1. **Data Recovery** — Restoring lost data after a production database failure
2. **Reproducible Data Migration** — Building a chunked export/import system so data can move safely between environments
3. **Database Automator** — Starting a new automated protection system (log capture, filtered alerting, Aurora snapshots)

---

## Part 1 — Data Recovery

### The Incident

The Flask production database (Aurora/MySQL) encountered a failure due to **foreign key relationship errors** in the ORM. When `db_init.py` tried to drop and recreate tables, MySQL's foreign key constraints blocked the operation.

### The Fix

The `db_init.py` script was repaired to **disable foreign key checks before dropping tables**, then re-enable them before recreation:

```python
# Disable foreign key checks for MySQL (no effect on SQLite)
if db.engine.url.drivername in ['mysql', 'mysql+pymysql']:
    db.session.execute(db.text('SET FOREIGN_KEY_CHECKS=0;'))
    db.session.commit()

db.drop_all()

if db.engine.url.drivername in ['mysql', 'mysql+pymysql']:
    db.session.execute(db.text('SET FOREIGN_KEY_CHECKS=1;'))
    db.session.commit()

db.create_all()
```

This works because nothing matters before deleting everything — removing foreign keys prior to drop is safe. The script also detects the database engine type so it works for both production (MySQL) and local (SQLite).

### Temporary SQLite Fallback

While the MySQL database was being fixed, I switched production to use a **local SQLite file** on the AWS instance. This preserved user accounts and microblog data that would otherwise have been lost.

The procedure:
1. Take Flask Docker instance down
2. Comment out MySQL connection strings in `.env`
3. Recreate the Docker instance (now running on SQLite)
4. After MySQL was fixed: reverse the process to switch back

This meant we ended up with data split across two databases — MySQL had some data, the temporary `.db` file had other data. That led directly into Part 2.

---

## Part 2 — Reproducible Data Migration

### The Problem

We needed a way to reliably **pull data from production** and **push data to production** — but nginx timeouts made it impossible to transfer everything in a single request.

### Solution: Chunked Export/Import API

I built a new `data_export_import_api.py` in Flask with:

**Paginated exports** — each data type has its own endpoint with pagination:

```
GET /api/export/users?page=1&per_page=50
Response: { count, total, page, per_page, has_next, has_prev, data: [...] }
```

**Dependency-aware imports** — data must be imported in the correct order because of foreign key relationships:

```python
# Order matters:
# 1. Sections (no dependencies)
# 2. Users (depends on sections)
# 3. Topics (no dependencies)
# 4. Personas (no dependencies)
# 5. User Personas (depends on users + personas)
# 6. Microblogs (depends on users + topics)
# 7. Posts (depends on users)
# 8. Classrooms (depends on users)
# 9. Feedback, Study
```

**Chunked upload scripts** — the restore script batches uploads at 50 records per batch with 180-second timeouts per batch:

```python
large_datasets = ['users', 'microblogs', 'posts', 'user_personas', 'personas']
batch_size = 50 if data_type in large_datasets else None
batches = [data_list[i:i+batch_size] for i in range(0, len(data_list), batch_size)]
```

**Retry logic** — automatic retry on 504 Gateway Timeouts (up to 3 attempts with 2-second delays).

**Default data filtering** — the migration scripts exclude default/test users and sections so only real student data is transferred:

```python
DEFAULT_DATA = {
    'users': ['admin', 'user', 'niko'],
    'sections': ['CSA', 'CSP', 'Robotics', 'CSSE'],
    'topics': ['/lessons/flask-introduction', '/hacks/javascript-basics', ...]
}
```

### Relevant PRs and Commits

| Commit | Description |
|--------|-------------|
| [`ea121c9`](https://github.com/Open-Coding-Society/flask-tracking/commit/ea121c9) | Create a new export/import API with dependency-aware ordering |
| [`fd83605`](https://github.com/Open-Coding-Society/flask-tracking/commit/fd83605) | Add pagination to data export/import API with `joinedload` for N+1 prevention |
| [`0553210`](https://github.com/Open-Coding-Society/flask-tracking/commit/0553210) | Implement chunked exporting to avoid nginx timeout |
| [`9e39f85`](https://github.com/Open-Coding-Society/flask-tracking/commit/9e39f85) | Add chunking to migration/restore scripts (215 insertions) |
| [`2854221`](https://github.com/Open-Coding-Society/flask-tracking/commit/2854221) | Add error handling and retry catches to migration and restoration |

### The Data Merge

With the migration tools in place, I combined the MySQL and SQLite data:

1. **Pull** data from MySQL (current production data)
2. **Switch** to SQLite backend on the server
3. **Pull** data from the .db file (old data from the fallback period)
4. **Switch** back to MySQL backend
5. **Push** combined data to production
6. **Verify** via `flask.opencodingsociety.com` and direct database queries

### Spring Boot Migration Scripts

I also contributed to Spring's migration tooling:

- **`db_prod2local.py`** — Pulls production data to local SQLite using form-based authentication with session cookies
- **`db_local2prod.py`** — Pushes local data to production
- **`mysqlbackup.py`** — Backs up MySQL to SQLite format with schema type conversion (INT→INTEGER, VARCHAR→TEXT)
- **`mysqlrestore.py`** — Restores SQLite back to MySQL with reverse type mapping

Key Spring commits:
- [`4f5593fd`](https://github.com/CyberLord09/spring-tracking/commit/72c38bf6) — Create local-to-production database script
- [`72c38bf6`](https://github.com/CyberLord09/spring-tracking/commit/72c38bf6) — Force admin role requirement for exports/imports API
- [`090fe96e`](https://github.com/CyberLord09/spring-tracking/commit/090fe96e) — Add local-to-prod documentation into Spring README

---

## Part 3 — Database Automator (New Project)

After experiencing data loss firsthand, I started building an **automated protection system** to prevent it from happening again. This is the `tracking-database-automator` project.

### Phase 1: Log Capture (Complete)

A Dockerized Python service that streams logs from sibling containers via the Docker socket:

```
[flask_web_1 container]  ──► docker.sock ──► [log-capture] ──► ./logs/flask_web_1.log
[spring-web-1 container] ──►               [log-capture] ──► ./logs/spring-web-1.log
```

- Connects to `/var/run/docker.sock` to read container logs in real-time
- Auto-retries when containers restart
- Heartbeat logging every 60 seconds

### Phase 1.5: Log Filtering (Complete)

A filter layer that reduces the raw logs down to only security-relevant events:

**Flask rules:**
- Password resets, user deletions, user creation, login attempts
- All 4xx and 5xx responses

**Spring rules:**
- ERROR-level logs, exceptions, auth warnings
- Password and deletion operations

Filtered output goes to `logs/important/` — this becomes the input for future anomaly detection.

### Phase 2: Automated Snapshots (In Progress)

A cron-based snapshot system that handles both database types:

- **Aurora/RDS** (Flask production): Uses `boto3` to create tagged AWS snapshots
- **SQLite** (Spring production): Copies the `.db` file to timestamped backups
- **Retention policy**: 7 daily, 4 weekly, 3 monthly — older snapshots auto-deleted
- Runs daily at 2 AM UTC via cron inside the Docker container

---

## Future Plans

### Near-Term
- **Anomaly detection** — Sliding window counters on the filtered logs to flag mass password resets, bulk deletions, and other suspicious patterns
- **Slack alerting** — Webhook notifications when anomalies are detected
- **Snapshot API** — An endpoint the GitHub Pages frontend can call to trigger on-demand snapshots

### Longer-Term
- **Control panel** — GitHub Pages frontend for viewing alerts, triggering snapshots, approving migrations, and initiating restores
- **Pre-migration auto-snapshots** — Automatically take a snapshot before any database migration runs
- **Race condition prevention** — Audit critical code paths and add proper locking mechanisms

---

## Reflection

The biggest lesson this trimester was that **data loss is not an "if" but a "when"**. Having experienced it, the priority shifted from reactive fixes to proactive protection. The export/import API and migration scripts turned a multi-hour manual recovery into a repeatable 10-minute process. The database automator takes it further — the goal is to detect problems before they cause damage and guarantee recovery when they do.