---
spec_id: 001-create-and-list-tasks-in-memory
title: Create and list tasks in memory
status: approved
priority: P1
shape: service-python
---

# 001 — Create and list tasks in memory

## Problem

The Task API has no tasks yet: it offers only health and readiness, so a client cannot record anything to do. This is the first feature of the service. Callers need to create a task with a title and, optionally, a due date, and to read back all tasks they have created. Keeping tasks in memory makes the API usable now without choosing a database.

## Stories

As an API client, I want to create a task with a title and an optional due date, so that I can record what has to be done and by when.

As an API client, I want to list all tasks, so that I can see what has been recorded.

## Acceptance criteria

- **AC-001.1** Given the service is running, when a client sends `POST /tasks` with JSON `{"title": "Buy milk", "due_date": "2026-12-01"}`, then the response is `201` with a JSON body holding `id` (a non-empty string unique among tasks), `title` `"Buy milk"` and `due_date` `"2026-12-01"`. (smoke)
- **AC-001.2** Given the service is running, when a client sends `POST /tasks` with only `{"title": "Buy milk"}` or with `"due_date": null`, then the response is `201` and the body's `due_date` is `null`.
- **AC-001.3** Given a create request whose `title` is missing, not a string, empty or only whitespace, or longer than 200 characters, when it is sent, then the response is `422` with a JSON error naming the `title` field, and no task is created.
- **AC-001.4** Given a create request whose `due_date` is present but is not a calendar date in the form `YYYY-MM-DD` (for example `"tomorrow"`, `"2026-02-30"` or `"2026-12-01T10:00:00Z"`), when it is sent, then the response is `422` with a JSON error naming the `due_date` field, and no task is created.
- **AC-001.5** Given tasks A, B and C were created in that order, when a client sends `GET /tasks`, then the response is `200` with a JSON array of exactly those three tasks in creation order, each with the same `id`, `title` and `due_date` the create response returned. (smoke)
- **AC-001.6** Given no task has been created since the service started, when a client sends `GET /tasks`, then the response is `200` with an empty JSON array `[]`.
- **AC-001.7** Given tasks were created, when the service restarts, then `GET /tasks` returns `[]`.

## Non-goals

- Persistence across restarts or sharing tasks between several instances.
- Reading, updating, completing or deleting a single task.
- Filtering, sorting options, search or pagination of the list.
- Authentication, users or per-user task lists.
- Due times or time zones; a due date is a calendar date only.

## Assumptions

- **A1** Tasks live at `/tasks`: `POST /tasks` creates, `GET /tasks` lists (other option: a versioned prefix such as `/v1/tasks`; affects AC-001.1, AC-001.2, AC-001.5, AC-001.6).
- **A2** A task in responses has exactly the fields `id`, `title`, `due_date`; no creation timestamp or status (other option: add `created_at` and `done`; affects AC-001.1, AC-001.5).
- **A3** `id` is a server-generated opaque string (for example a UUID); clients supply no id and must not rely on its format (other option: sequential integers; affects AC-001.1).
- **A4** Title is 1 to 200 characters after trimming surrounding whitespace, and is stored trimmed (other option: no trimming, other limit; affects AC-001.3).
- **A5** `due_date` is an ISO 8601 calendar date `YYYY-MM-DD`; dates in the past are accepted (other option: reject past dates, or accept date-times; affects AC-001.4).
- **A6** Validation errors answer `422` with the framework's standard JSON error body that names the offending field (other option: `400` with a custom error body; affects AC-001.3, AC-001.4).
- **A7** The list is ordered by creation, oldest first, and returned in full with no limit (other option: newest first, or by due date; affects AC-001.5).
- **A8** Unknown extra fields in a create request are ignored (other option: reject with `422`; affects AC-001.1).

## Open questions

none

## Changelog

- 2026-11-15 created
- 2026-11-15 approved
