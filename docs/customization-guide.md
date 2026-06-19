# Customization Guide

This guide explains how to adapt the **Student Deadline Command Center Automation** to fit your specific academic requirements, communication habits, and time availability.

---

## 1. Modifying the Priority Scoring Logic

The priority score is calculated inside the n8n JavaScript code block titled `Calculate Priority Signals`. You can change the math to place more weight on grade impact versus timeline urgency.

### How to Adjust Weights
Locate the variables defined at the top of the node script:
```javascript
// Default weight settings (Must sum to 1.0)
const W_TIME = 0.50;   // urgencies based on due dates
const W_WEIGHT = 0.30; // grade impact
const W_EFFORT = 0.20; // time required to complete
```
*   **For Exam-Heavy courses:** Increase `W_WEIGHT` to `0.50` and decrease `W_TIME` to `0.30`. This ensures major exams dominate the priority score even if they are weeks away.
*   **For Short-Deadline courses (e.g., Bootcamps):** Increase `W_TIME` to `0.70` and decrease `W_WEIGHT` to `0.15` to prioritize rapid turnarounds.

### Changing Thresholds
To adjust what qualifies as a "Critical" task vs. "Routine", edit the threshold conditions:
```javascript
if (score >= 80) {
  item.priority_tier = 'critical';
} else if (score >= 50) {
  item.priority_tier = 'important';
} else {
  item.priority_tier = 'routine';
}
```

---

## 2. School vs. University Context Customizations

The system behaves differently based on the `academic_level` field sent in the student profile:
*   `university_undergrad` & `graduate_research`: Study block sizes are set larger (90-120 mins) to support deep focus, and weight is given to research milestones.
*   `high_school`: Default study block sizes are set smaller (30-45 mins) to align with shorter attention spans, and administrative parent-facing summary templates can be activated.

To customize these profiles globally, open `prompts/system-weekly-academic-planner.md` and edit the guidelines under the `# STUDENT PROFILE RULES` header.

---

## 3. Adapting Timezone Behavior

Timezone mismatches can cause study blocks to overlap or miss deadlines.
1.  Always pass dates to the webhook trigger using an explicit timezone designator (ISO-8601, e.g., `2026-06-25T14:00:00-05:00` for EST).
2.  Set your local timezone in the n8n system settings so Javascript nodes parse strings using matching local times.
3.  Inside the n8n input normalization block, check that the global timezone parameter is passed to the AI:
    ```json
    "timezone": "America/New_York"
    ```

---

## 4. Customizing Study Block Preferences

You can adjust how study blocks are allocated by modifying the `study_preferences` parameters in the input payload:
*   `block_length_minutes`: Default is `90`. Change to `50` for Pomodoro-style scheduling or `120` for long research paper sessions.
*   `preferred_study_times`: Specify `afternoon`, `morning`, or `evening`. The AI planner is instructed to map sessions to these times.
*   `quiet_hours`: Specify hour ranges (e.g., `["22:00-08:00"]`) when the planner must never schedule study blocks.

---

## 5. Changing Communication & Reminder Channels

The workflow has output branches for Telegram, Email, and Notion. You can adjust the destination and notification frequency:
*   **Slack/Discord:** Replace the Telegram push node with an n8n Discord/Slack webhook node to post summaries to a private study server.
*   **Daily Alerts vs. Weekly Digests:** You can set a rule inside n8n to send **Daily Prioritizations** only if `priority_score >= 80` (Critical), while deferring routine reminders to the **Sunday Weekly Digest**.

---

## 6. Tuning AI Tone and Strictness

You can modify the "strictness" of the scheduler and how it addresses workload.
*   **Academic Tone:** By default, prompts specify a `concise`, `professional`, and `objective` tone. If you want a more urgent tone, update `preferred_tone` to `urgent_direct` in the student profile.
*   **Strict Deferral Limit:** By default, the weekly planner will freely defer assignments representing < 3% weight. If you wish to forbid deferrals entirely, edit the system planner prompt to include: `CRITICAL: You are strictly forbidden from deferring any items marked as mandatory, regardless of weight.`
