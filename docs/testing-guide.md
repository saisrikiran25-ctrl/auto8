# Testing Guide

This document contains 8 realistic test scenarios to validate the end-to-end functionality of the **Student Deadline Command Center Automation**. Use these to test the n8n logic, prompt templates, and output parsers.

---

## Test Scenarios

### Scenario 1: Single Assignment Due in 2 Days
*   **Test Name:** `test_01_single_imminent_deadline`
*   **Sample Input Conditions:**
    *   `analysis_mode`: `prioritize`
    *   One deadline item: Title: "Lab Report 3", Course: "CHEM 2080", Due Date: +48 hours from current local time, Estimated Hours: 4.0, Weight: 5%.
*   **Expected Behavior:** The system should bypass the weekly planner logic and run the prioritization chain. Since the deadline is only 2 days away, it should return a high priority score and recommend the student "Start Now".
*   **What to Validate:**
    *   `priority_score` is computed as $\ge 70$ due to time urgency.
    *   `recommended_next_actions` directs the student to start immediately.
    *   No weekly plan fields are present in the final output.

### Scenario 2: Multiple Deadlines in One Week with One Exam
*   **Test Name:** `test_02_mixed_week_with_exam`
*   **Sample Input Conditions:**
    *   `analysis_mode`: `weekly_plan`
    *   One exam: Title: "Midterm 1", Course: "MATH 1920", Due Date: +4 days, Estimated Hours: 10.0, Weight: 20%.
    *   Two assignments: "HW 4" (due +2 days, 2h effort, 2% weight), "Writing Prep" (due +6 days, 1.5h effort, 1% weight).
*   **Expected Behavior:** The workflow routes to the weekly planner. The math exam dominates the priorities. Study blocks are scheduled across the week, spacing out the math review.
*   **What to Validate:**
    *   The weekly summary highlights the MATH midterm as the core focus.
    *   The study plan schedules multiple blocks for MATH 1920 prior to the exam date.
    *   Low-weight homework is scheduled during transition windows.

### Scenario 3: Overdue Assignment Risk Trigger
*   **Test Name:** `test_03_overdue_work_alert`
*   **Sample Input Conditions:**
    *   `analysis_mode`: `prioritize`
    *   One item: Title: "Problem Set 2", Course: "PHYS 2213", Due Date: -24 hours (yesterday), Status: `not_started`, Weight: 8%.
*   **Expected Behavior:** The deterministic check node flags the item as overdue. A critical alert flag is raised, bypassing normal AI analysis.
*   **What to Validate:**
    *   `overdue_flag` is set to `true`.
    *   `overall_risk_level` is set to `critical`.
    *   The message output prefixes the status with `🚨 OVERDUE`.

### Scenario 4: Weak Input Payload with Missing Due Date
*   **Test Name:** `test_04_weak_input_fallback`
*   **Sample Input Conditions:**
    *   `analysis_mode`: `prioritize`
    *   One item: Title: "Read Chapter 4", Course: "HIST 1200", Due Date: `null` or missing, Estimated Hours: `null`.
*   **Expected Behavior:** The deterministic validation node detects that the incoming signal lacks a valid due date. The request is routed to the Low-Signal Fallback Node.
*   **What to Validate:**
    *   LLM nodes are not invoked (saves API cost).
    *   The final response status is `fallback`.
    *   The warning list details: `Missing due_date field for HIST 1200: Read Chapter 4`.

### Scenario 5: Administrative Fee Deadline Mixed with Academic Work
*   **Test Name:** `test_05_admin_vs_academic_routing`
*   **Sample Input Conditions:**
    *   `analysis_mode`: `prioritize`
    *   Item A: "Tuition Installment Payment", Type: `fee_payment`, Due: +3 days, Weight: 0%.
    *   Item B: "Essay 1 Draft", Type: `assignment`, Due: +3 days, Weight: 10%.
*   **Expected Behavior:** The priority scoring algorithm processes both, but weights the essay higher for study block planning. The fee payment is flagged as a critical administrative checklist item but receives 0 study blocks.
*   **What to Validate:**
    *   `Tuition Installment Payment` has no study blocks scheduled.
    *   The prioritization reasoning categorizes the fee payment under administrative checklists, requiring payment action, not research study.

### Scenario 6: Student Overloaded with Commits (Massive Workload)
*   **Test Name:** `test_06_workload_saturation`
*   **Sample Input Conditions:**
    *   `analysis_mode`: `weekly_plan`
    *   Student Profile: `weekly_available_study_hours`: 10.
    *   Deadlines: 4 different items totaling 18.0 hours of estimated study time due within the next 5 days.
*   **Expected Behavior:** The system detects that the required effort (18h) exceeds the study capacity (10h). It triggers a workload overload alarm and uses the AI to recommend deferring low-impact homework.
*   **What to Validate:**
    *   `overall_risk_level` is `critical_overload`.
    *   `needs_human_review` is set to `true`.
    *   `defer_if_needed_items` lists the lower-weighted assignments.

### Scenario 7: Weekly Plan Under Study-Hour Constraints
*   **Test Name:** `test_07_tight_study_constraints`
*   **Sample Input Conditions:**
    *   `analysis_mode`: `weekly_plan`
    *   Student Profile: `weekly_available_study_hours`: 12, `block_length_minutes`: 60.
    *   Deadlines: Items requiring 11.5 hours of study.
    *   Calendar: High quantity of busy blocks (classes and labs).
*   **Expected Behavior:** The planner generates exact study sessions of 60 minutes each. It fits them around the busy blocks, leaving 30 minutes of headroom.
*   **What to Validate:**
    *   Each recommended study block has a `session_length_minutes` of exactly `60`.
    *   None of the study block windows overlap with the calendar busy array.

### Scenario 8: Malformed AI Output Fallback
*   **Test Name:** `test_08_llm_json_recovery`
*   **Sample Input Conditions:**
    *   LLM output contains conversational filler: "Sure, here is the JSON you requested: \n```json\n { ... } \n``` I hope this helps!"
*   **Expected Behavior:** The workflow's JSON regex parser isolates the raw `{}` block, strips out the conversational text, and successfully parses the JSON.
*   **What to Validate:**
    *   The workflow does not crash.
    *   The parser returns a valid Javascript object and routes to the final response node.
