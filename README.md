# Student Deadline Command Center Automation

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![n8n Compatible](https://img.shields.io/badge/n8n-Compatible-orange.svg)](https://n8n.io/)

A high-performance, production-quality academic workload intelligence automation built on n8n. It centralizes assignment deadlines, exam dates, class notices, and calendar items into a single, structured system that calculates priority, structures study blocks, warns of overload risks, and issues reminders.

---

## 1. Product Overview

The **Student Deadline Command Center Automation** is not a generic study app, to-do list, or a motivational therapist. It is an **academic operating and planning system** designed to solve the operational bottleneck of fragmented academic deadline management. 

Instead of waiting for you to enter tasks manually or expecting you to read across multiple platforms, this system aggregates data, normalizes it, runs deterministic checks, and uses structured AI reasoning to construct an actionable study plan and pinpoint capacity bottlenecks.

*   **Product Promise:** Convert scattered academic deadlines into a single, prioritized student action system.
*   **Core Focus:** Deadline visibility, automated prioritization, risk detection, and study capacity scheduling.

---

## 2. The Problem It Solves

Students manage academic lives across a highly fragmented ecosystem: LMS portals (Canvas/Blackboard), emails, Google Calendar, class WhatsApp groups, Discord servers, Notion pages, and PDF syllabi. This fragmentation leads to:
1.  **Deadline Fragmentation:** Crucial deliverables are missed because dates reside in separate databases.
2.  **Planning Fallacy & Poor Prioritization:** Tasks are viewed as a flat list rather than a prioritized timeline accounting for relative weights and effort.
3.  **Last-Minute Rushes:** Major exams or papers are tackled too late because they weren't broken down into visible study blocks early enough.
4.  **No Risk Visibility:** No early warnings when a student's week is mathematically overloaded.

This automation resolves these bottlenecks by acting as an active **student operations assistant**.

---

## 3. What It Does & Does Not Do

### What It Does
*   **Ingestion & Normalization:** Merges items from webhooks, calendars, forms, spreadsheets, and emails into a clean schema.
*   **Deterministic Validation:** Catches missing titles, invalid dates, completed tasks, and duplicate entries.
*   **Internal Diagnostics (`_debug_` convention):** Standardized workflow variables are stored in `_debug_` namespaced fields (e.g., `_debug_validation`, `_debug_overdue`, `_debug_scoring_completed`) to preserve intermediate logic states for audit logging and tracing without leaking details into final production outputs.
*   **Priority Calculation:** Computes a priority score based on days remaining, grade weight, and estimated effort.
*   **AI Reasoning Layer:** Translates the prioritizations into concrete daily actions and schedules structured study blocks.
*   **Overload Detection:** Issues warnings when daily study requirements exceed available hours.
*   **Dual Modes:** Support for **Mode A: Deadline Prioritization Analysis** (ad-hoc submission analysis) and **Mode B: Weekly Academic Planning** (comprehensive 7-14 day strategy).

### What It Does NOT Do
*   **It is not a motivation/habit app:** It will not send inspirational quotes or monitor daily habits.
*   **It is not a therapist:** It will not discuss stress or address mental health. It acts as an operational schedule optimizer.
*   **It is not an LMS replacement:** It does not host files, track grades, or submit assignments.
*   **It does not replace institutional systems:** Canvas, Blackboard, or university registrar calendars remain the absolute source of truth.

---

## 4. Main Modes of Operation

| Mode | Trigger | Input | Output |
| :--- | :--- | :--- | :--- |
| **Mode A: Deadline Prioritization** | Ad-hoc / Webhook | Single task or a small set of newly discovered deadlines. | Categorized urgency, priority rank, immediate action recommendation, and suggested reminder cadence. |
| **Mode B: Weekly Academic Planning** | Recurring (e.g., Sunday 6 PM) | Next 7–14 days of deadlines, busy calendar blocks, and weekly study hour constraints. | Weekly focus areas, daily study schedules with estimated blocks, conflict alerts, and catch-up plans. |

---

## 5. Repository Structure

```text
student-deadline-command-center-automation/
├── README.md                                    # Main repository overview and documentation portal
├── assets/
│   ├── architecture-overview.md                 # Deep-dive of data pipelines, deterministic logic, and AI flow
│   ├── weekly-digest-example.md                 # Markdown representation of a finalized weekly planner message
│   └── risk-alert-example.md                    # Markdown representation of an automated overload warning
├── docs/
│   ├── setup-guide.md                           # Comprehensive import and deployment steps in n8n
│   ├── credential-setup.md                      # Security best practices for connecting external OAuth apps
│   ├── customization-guide.md                   # How to tune thresholds, study hours, and academic levels
│   ├── testing-guide.md                         # Suite of 8 realistic test cases for QA validation
│   └── output-examples.md                       # Sample payloads for all output states (JSON formats)
├── prompts/
│   ├── system-deadline-prioritization-analysis.md # Urgency and prioritization AI agent system prompt
│   ├── user-deadline-intake-template.md         # Runtime prompt template for Mode A ingestion
│   ├── system-weekly-academic-planner.md        # Academic planner agent system prompt
│   └── user-weekly-academic-planner-template.md # Runtime prompt template for Mode B weekly digests
├── schemas/
│   ├── student-deadline-input-payload-example.json # Input payload detailing student context and priorities
│   ├── deadline-prioritization-output-schema.json # JSON Schema validating Mode A outputs
│   └── weekly-academic-plan-output-schema.json    # JSON Schema validating Mode B outputs
└── workflow/
    └── student-deadline-command-center.json     # The complete, importable n8n workflow file
```

---

## 6. Getting Started (Setup Summary)

1.  **Import Workflow:** Copy the JSON file in `workflow/student-deadline-command-center.json` and paste it directly into your n8n workflow editor.
2.  **Configure LLM Nodes:** Create credentials for your preferred AI provider (e.g., Google Gemini, OpenAI, or Anthropic) in the LLM nodes.
3.  **Map Inputs:** Connect the Webhook trigger to your manual submission form, Notion database, or email parser.
4.  **Test Execution:** Run a sample payload from `schemas/student-deadline-input-payload-example.json` to verify schema validation and parser nodes.
5.  *Detailed instructions are available in [setup-guide.md](docs/setup-guide.md).*


---

## 7. Example Payload Summary

### Input (Snippet)
```json
{
  "student_id": "std_9921",
  "analysis_mode": "weekly_plan",
  "student_profile": {
    "name": "Sarah Chen",
    "academic_level": "university_undergrad",
    "weekly_available_study_hours": 15
  },
  "deadline_items": [
    {
      "title": "CS 3110 Midterm Exam",
      "course_name": "CS 3110: Advanced Data Structures",
      "deadline_type": "exam",
      "due_date": "2026-06-25T14:00:00Z",
      "estimated_hours": 8.0,
      "weight_percent": 25.0,
      "importance_level": "high"
    }
  ]
}
```

### Output (Snippet)
```json
{
  "weekly_summary": "High workload concentrated on Thursday due to the CS 3110 Exam. Prioritize active study blocks early in the week.",
  "top_three_focus_areas": [
    "CS 3110 Midterm Prep (8 hours)",
    "CHEM 2080 Lab Report (4 hours)",
    "FWS Essay Draft (3 hours)"
  ],
  "overall_risk_level": "medium",
  "needs_human_review": false
}
```

---

## 8. Limitations & Edge Cases

*   **Timezones:** If calendar entries do not specify an offset, local times may clash.
*   **Weak Signals:** Form submissions lacking due dates or course weight default to mid-range priority parameters.
*   **Context Windows:** Very large deadline histories (50+ items) will require adjustments to the prompt limits in n8n.
*   **LLM Hallucinations:** Validation nodes exist post-AI to check that estimated hours and due dates match the original input and have not been fabricated.

---

## 9. Safety, Boundaries & Disclaimer

> [!WARNING]
> This automation is a **planning, prioritization, and scheduling assistant** meant to optimize student time allocation. It is not an academic advisory tool, counseling resource, or a guarantor of academic outcomes.

*   **Institutional Source of Truth:** Students are solely responsible for verifying the accuracy of submission links, deadlines, exam rooms, and course policy changes in official systems (Canvas, Blackboard, Registrar).
*   **Safe Communication Boundary:** The system prompts are engineered to prevent diagnostic or shaming language. The automation evaluates workload volume and schedules mathematically, avoiding comments on personal character, ability, or mental health.
*   **No Grades Guaranteed:** Generating a study block structure does not guarantee academic success or completion. The student must execute the blocks.

---

## License

This project is licensed under the MIT License - see the LICENSE file for details.
