# Integration Schema Mapping Guide

This document lists the field-by-field translation rules from common third-party student ingestion services to the **Student Deadline Command Center Automation** normalization layer.

---

## 1. Canvas LMS API Integration

When syncing with Canvas LMS endpoints (e.g., `/api/v1/courses/:course_id/assignments`), map the returned JSON objects using the following criteria:

| Canvas API Property | Normalized Schema Target | Translation / Default Rule |
| :--- | :--- | :--- |
| `name` | `title` | Pass raw string. |
| `course_id` (or prefix) | `course_name` | Cross-reference course code ID from your enrollment details. |
| `submission_types` | `deadline_type` | Map `discussion_topic` to `assignment`; map `online_quiz` to `quiz`; map `examination` to `exam`. |
| `due_at` | `due_date` | Convert ISO-8601 UTC string directly. |
| `points_possible` | `weight_percent` | Check against syllabus mapping or compute relative weight based on course point total. |
| `description` | `notes` | Strip HTML tags using regex before saving. |
| `html_url` | `link_reference` | Map directly. |

---

## 2. Notion Assignment Database

If you keep deadlines in a Notion Table Database, set up your property titles exactly as listed or adjust your n8n Notion block mapping:

| Notion Property Type | Property Title | Normalized Schema Target |
| :--- | :--- | :--- |
| Title | `Name` | `title` |
| Select | `Course` | `course_name` |
| Select | `Type` | `deadline_type` (`Assignment`, `Exam`, `Quiz`, `Admin`) |
| Date | `Due Date` | `due_date` |
| Number | `Estimated Hours` | `estimated_hours` |
| Number | `Weight (%)` | `weight_percent` |
| Status | `Status` | `status` (`Not started`, `In progress`, `Completed`) |
| Text | `Notes` | `notes` |

---

## 3. Google Calendar (iCal/ICS Ingest)

For calendar events, n8n parses Google Calendar fields as follows:

| Google Calendar Property | Normalized Schema Target | Translation / Default Rule |
| :--- | :--- | :--- |
| `summary` | `title` | If title contains "EXAM", map `deadline_type` to `exam`. |
| `start.dateTime` | `due_date` | Use start time of the deadline event as absolute UTC time. |
| `description` | `notes` | Extract URLs for mapping references. |
| `organizer.displayName` | `course_name` | Default course code matching organiser domain or email group name. |
