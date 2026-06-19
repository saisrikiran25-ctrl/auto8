# User Prompt Template: Weekly Academic Planner

This template is dynamically populated by the n8n workflow for Mode B execution, typically triggered on a weekly scheduling cron.

---

## 1. Input Environment Context

*   **Current Date & Time:** {{ $json.current_time || new Date().toISOString() }}
*   **Target Week Boundaries:** From {{ $json.week_start }} to {{ $json.week_end }}
*   **Timezone:** {{ $json.timezone }}

---

## 2. Student Context & Capacity Parameters

*   **Student ID:** {{ $json.student_id }}
*   **Name:** {{ $json.student_profile.name }}
*   **Academic Level:** {{ $json.student_profile.academic_level }}
*   **Weekly Study Hours Capacity:** {{ $json.student_profile.weekly_available_study_hours }} hours
*   **Study Preferences:**
    *   Preferred Block Length: {{ $json.study_preferences.block_length_minutes }} minutes
    *   Preferred Study Times: {{ $json.study_preferences.preferred_study_times.join(', ') }}
    *   Reminder Delivery: {{ $json.study_preferences.preferred_reminder_channel }}

---

## 3. Academic Inventory

### Deadlines & Incomplete Tasks (Next 14 Days)
The following tasks are active and require time allocation:
```json
{{ JSON.stringify($json.deadline_items, null, 2) }}
```

### Fixed Calendar Commitments (Busy Slots)
The planner must not allocate study sessions during these blocks:
```json
{{ JSON.stringify($json.calendar_constraints.busy_slots, null, 2) }}
```

### Quiet Hours (No-Study Zones)
```json
{{ JSON.stringify($json.calendar_constraints.quiet_hours, null, 2) }}
```

---

## 4. Specific Action Instructions

1.  **Calculate the total hours required** for all tasks due between {{ $json.week_start }} and {{ $json.week_end }}. Compare this with the capacity of {{ $json.student_profile.weekly_available_study_hours }} hours.
2.  If the total hours required exceed capacity, flag `needs_human_review = true` and `overall_risk_level = "critical_overload"`. Populate `conflict_warnings` suggesting which tasks can be safely deferred.
3.  **Generate study blocks** of exactly {{ $json.study_preferences.block_length_minutes }} minutes. Map them to the days preceding each due date, prioritizing items with high `priority_score`.
4.  Fit blocks within the student's `preferred_study_times` (e.g., afternoon) and strictly avoid the `busy_slots` and `quiet_hours`.
5.  Check for **overdue items** in the inventory (due in the past, status != `completed`) and list them under `overdue_items` in the output.
6.  Return the finalized weekly planner output exactly in the JSON format specified in the system instructions. Do not add any leading or trailing conversational text.
