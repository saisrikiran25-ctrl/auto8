# Risk Alert: Workload Capacity & Deadline Clashes (Example Output)

**System Alert ID:** alt_8829-x  
**Recipient:** Sarah Chen  
**Trigger:** Mode B Weekly Intake Parser  
**Calculated Risk Level:** 🔴 CRITICAL OVERLOAD  

---

## 🚨 Active Risk Summary

The automation has detected **three major conflicts** in your schedule for the upcoming week (Jun 15 - Jun 21, 2026). If unaddressed, there is a high statistical probability of late submissions or insufficient preparation time.

```text
--------------------------------------------------------------------------------
[ALERT 1]  CAPACITY DEFICIT: Required study hours exceed available hours by 4.5.
[ALERT 2]  SCHEDULING BIND: Three deadlines coincide on Thursday, Jun 18.
[ALERT 3]  IMPOSSIBLE WINDOW: Study blocks scheduled during fixed biology lab hours.
--------------------------------------------------------------------------------
```

---

## 🔍 Detailed Risk Breakdown

### 1. Capacity Deficit Alert
*   **Weekly Study Hours Target:** 15.0 hours max.
*   **Calculated Effort Required:** 19.5 hours (CS 3110: 10.0h, CHEM 2080: 6.0h, ENGL 1010: 3.5h).
*   **Net Capacity Gap:** **-4.5 hours** (Over-allocated by 30%).
*   *Deterministic Logic Flag:* Insufficient available study hours.
*   *Action Required:* You must defer non-critical items or increase weekly available hours.

### 2. Multi-Deadline Clashes (Thursday, Jun 18)
*   **Coincident Items:**
    1.  `CS 3110 Midterm Exam` (Weight: 25%, Due: 14:00)
    2.  `CHEM 2080 Pre-Lab Assignment 5` (Weight: 2%, Due: 09:00)
    3.  `ENGL 1010 Short Essay 2` (Weight: 10%, Due: 23:59)
*   *System Recommendation:* Competing deadline penalty applied.
    *   Shift `CHEM 2080 Pre-Lab Assignment 5` to Tuesday night.
    *   Shift `ENGL 1010 Short Essay 2` to Wednesday evening.
    *   Protect Thursday morning exclusively for CS 3110 exam review.

### 3. Impossible Study Windows (Calendar Overlap)
*   **Conflict Detected:** On Wednesday, Jun 17, the recommended study block for **CS 3110 (13:00 - 14:30)** directly overlaps with your fixed calendar event: **BIOL 1105 Laboratory (13:00 - 16:00)**.
*   *Error Type:* Physically impossible study window.
*   *Action Taken:* The study block has been automatically recalculated and shifted to **Wednesday, 16:30 - 18:00**.

---

## 🔄 Recommended Correction Protocol

To resolve this critical overload, implement the following adjustments:

```diff
- DEFER: ENGL 1010 Short Essay 2 (Estimated 3.5h) -> Postpone starting until Friday morning.
+ ADVANCE: CHEM 2080 Pre-Lab 5 -> Complete on Monday during routine hour block.
- CANCEL: Optional Study Group for CS 3110 -> Reallocate 2 hours to solitary practice exams.
```

---

## 🛡️ Safeguards & Data Truth

*   *Note on Data Validity:* These calculations are based on your manual settings of `weekly_available_study_hours` and the estimated effort in your Canvas syllabus parser.
*   *Disclaimer:* If the exam time has changed in your syllabus, you must update the core scheduler database manually. The system does not write to or override university portal deadlines.
