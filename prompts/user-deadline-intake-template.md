# User Prompt Template: Deadline Intake

This is the template filled dynamically by the n8n runtime to analyze newly submitted academic events.

---

## Input Data Context

### Student Profile
*   **Student ID:** {{ $json.student_id }}
*   **Timezone:** {{ $json.timezone }}
*   **Preferred Tone:** {{ $json.preferred_tone }}
*   **Academic Level:** {{ $json.student_profile.academic_level }}
*   **Active Courses:** {{ $json.student_profile.courses.join(', ') }}
*   **Weekly Available Study Hours:** {{ $json.student_profile.weekly_available_study_hours }}

### New Deadline Items to Process
Below is the array of normalized task items extracted from current triggers:
```json
{{ JSON.stringify($json.deadline_items, null, 2) }}
```

### Reference Information & Current Time
*   **System Local Time:** {{ $json.current_time || new Date().toISOString() }}
*   **Time Since Epoch (UTC):** {{ new Date().getTime() }}

### Constraints & Preferences
*   **Calendar Constraints (Busy Slots):** {{ JSON.stringify($json.calendar_constraints.busy_slots) }}
*   **Quiet Hours:** {{ JSON.stringify($json.calendar_constraints.quiet_hours) }}
*   **Study Preferences:**
    *   Target Block Length: {{ $json.study_preferences.block_length_minutes }} minutes
    *   Preferred Reminder Channel: {{ $json.study_preferences.preferred_reminder_channel }}
    *   Preferred Study Times: {{ $json.study_preferences.preferred_study_times.join(', ') }}

---

## Instructions for Execution

1.  **Analyze the new items** against the student's active course list and weekly availability.
2.  Review the `priority_score` calculated programmatically in the incoming payload. If none is supplied, calculate it according to the core decay logic.
3.  Cross-reference the task `due_date` with the `busy_slots` to note direct calendar conflicts.
4.  Formulate the next actions, limit recommendations strictly to functional preparation steps, and assign them to start immediately, monitor, or defer.
5.  Return the output strictly in the schema JSON format specified in the system instructions. Do not write any conversational text.
