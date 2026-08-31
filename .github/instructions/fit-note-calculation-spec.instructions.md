# Fit Note Calculation Specification

## Purpose

Define the deterministic business logic for a simple frontend-only single-page application:

> **“Do I need a fit note? (UK)”**

The application determines whether a worker should obtain a fit note based on the **length of their sickness absence**.

The application asks for:

1. The first date the worker was unable to work because of illness.
2. The last date the worker was unable to work because of illness.

The application **shall not ask for the worker's usual working days in the calculation flow**.

The reason is that the relevant threshold is based on **calendar days**, not the number of working days missed.

---

## Authoritative sources

Use these sources as the basis for the business logic:

1. GOV.UK — *The fit note: guidance for patients and employees*
   [https://www.gov.uk/government/publications/the-fit-note-a-guide-for-patients-and-employees/the-fit-note-guidance-for-patients-and-employees](https://www.gov.uk/government/publications/the-fit-note-a-guide-for-patients-and-employees/the-fit-note-guidance-for-patients-and-employees)

2. NHS — *Getting a fit note*
   [https://www.nhs.uk/nhs-services/gps/getting-a-fit-note/](https://www.nhs.uk/nhs-services/gps/getting-a-fit-note/)

3. ACAS — *Fit notes and proof of sickness*
   [https://www.acas.org.uk/fit-notes-and-proof-of-sickness](https://www.acas.org.uk/fit-notes-and-proof-of-sickness)

These sources should be treated as the source of truth for the application's core 7-day calculation.

---

## 1. Core rule

A worker:

- **does not need a fit note based on the length of the absence** if they are unable to work for **7 calendar days or fewer**;
- **should get a fit note** if they are unable to work for **more than 7 calendar days**.

The threshold is therefore:

```text
1–7 calendar days → no fit note
8+ calendar days  → fit note
```

The calculation uses **calendar days**, not working days.

---

## 2. Calendar days

Every date in the continuous sickness absence counts as one calendar day.

This includes:

- Monday
- Tuesday
- Wednesday
- Thursday
- Friday
- Saturday
- Sunday
- bank holidays
- the worker's normal days off
- days on which the worker would not normally have worked

Do **not** exclude weekends or other non-working days.

Do **not** calculate the number of scheduled workdays missed.

---

## 3. Application inputs

The application should have exactly these inputs for the core calculation:

### Input 1: First day unable to work

Label:

> **What was the first day you were unable to work because you were ill?**

Store as:

```text
startDate
```

### Input 2: Last day unable to work

Label:

> **What was the last day you were unable to work because you were ill?**

Store as:

```text
endDate
```

The `endDate` represents the **last day the worker was unable to work**, not the day they returned to work.

---

## 4. Do not ask for usual working days

The application must **not require the user to enter their usual working days**.

Do not ask whether they normally work:

- Monday–Friday
- Tuesday–Saturday
- weekends
- part-time days
- irregular days

The worker's working pattern has **no effect on the 7-calendar-day threshold**.

For example, an absence lasting Monday through the following Monday is 8 calendar days regardless of whether the worker normally works:

```text
Monday–Friday
```

or:

```text
Monday, Wednesday, Friday
```

or:

```text
Saturday–Sunday
```

or any other schedule.

---

## 5. Why working days are not an input

The application answers:

> **How many consecutive calendar days was the worker unable to work because they were ill?**

It does not answer:

> **How many working days did the worker miss?**

These are different calculations.

Example:

```text
Normal working pattern:
Monday–Friday

Sickness absence:
Monday through the following Monday

Scheduled working days missed:
6

Calendar days absent:
8

Fit-note result:
FIT NOTE
```

The relevant value is **8 calendar days**.

---

## 6. Date validation

The application must validate the inputs before calculating.

### Missing start date

If `startDate` is missing:

```text
Validation error:
"The first day of illness is required."
```

### Missing end date

If `endDate` is missing:

```text
Validation error:
"The last day of illness is required."
```

### End date before start date

If:

```text
endDate < startDate
```

return a validation error:

> **The last day of illness cannot be before the first day of illness.**

Do not perform the fit-note calculation.

---

## 7. Same-day absence

A start date equal to the end date is valid.

Example:

```text
startDate = Monday
endDate   = Monday
```

The duration is:

```text
1 calendar day
```

Result:

```text
NO_FIT_NOTE
```

---

## 8. Inclusive duration calculation

The start and end dates are both included in the sickness absence.

Calculate:

```text
calendarDays =
    differenceInCalendarDays(endDate, startDate) + 1
```

Examples:

```text
Monday → Monday
= 1 day

Monday → Tuesday
= 2 days

Monday → Sunday
= 7 days

Monday → following Monday
= 8 days
```

The implementation must use a reliable **calendar-date difference** rather than relying on elapsed hours.

Avoid calculations that can be affected by daylight-saving-time transitions.

---

## 9. Decision algorithm

The complete calculation is:

```text
INPUT:
    startDate
    endDate

VALIDATE:
    startDate exists
    endDate exists
    endDate >= startDate

CALCULATE:
    calendarDays = inclusiveCalendarDayDifference(startDate, endDate)

DECIDE:
    IF calendarDays <= 7:
        status = "NO_FIT_NOTE"
    ELSE:
        status = "FIT_NOTE"

OUTPUT:
    calendarDays
    status
```

The decision function should conceptually be:

```text
getFitNoteStatus(startDate, endDate)
```

not:

```text
getFitNoteStatus(startDate, endDate, workingDays)
```

---

## 10. Boundary condition

The critical boundary is between **7 and 8 calendar days**.

Exactly 7 days:

```text
calendarDays = 7
→ NO_FIT_NOTE
```

Exactly 8 days:

```text
calendarDays = 8
→ FIT_NOTE
```

Do not implement the rule as:

```text
>= 7
```

The correct condition is:

```text
> 7
```

for requiring a fit note.

---

## 11. Critical example: Monday–Friday absence, return the following Monday

This is an important test case.

### Scenario

The worker normally works Monday–Friday.

They are unable to work:

```text
Monday
Tuesday
Wednesday
Thursday
Friday
```

They normally have Saturday and Sunday off.

They return to work on the following Monday.

### Calculation

The sickness absence is:

```text
Monday     = day 1
Tuesday    = day 2
Wednesday  = day 3
Thursday   = day 4
Friday     = day 5
Saturday   = day 6
Sunday     = day 7
```

The following Monday is the **return-to-work date**, so it is not part of the absence.

Therefore:

```text
calendarDays = 7
```

Result:

```text
NO_FIT_NOTE
```

The fact that the worker missed 5 scheduled working days does not change the result.

---

## 12. Eight-day version of the same scenario

### Scenario

The worker normally works Monday–Friday.

They are unable to work from Monday through the following Monday and return on Tuesday.

The absence is:

```text
Monday     = day 1
Tuesday    = day 2
Wednesday  = day 3
Thursday   = day 4
Friday     = day 5
Saturday   = day 6
Sunday     = day 7
Monday     = day 8
```

Therefore:

```text
calendarDays = 8
```

Result:

```text
FIT_NOTE
```

---

## 13. Return-to-work date

The application should avoid ambiguity by asking for the **last day unable to work**, rather than asking when the illness ended.

Use:

> **What was the last day you were unable to work because you were ill?**

If the worker returned to work on Monday, Monday is not part of the absence.

Example:

```text
Unable to work:
Monday → Sunday

Returned to work:
Monday
```

Input:

```text
startDate = Monday
endDate   = Sunday
```

Result:

```text
7 calendar days
NO_FIT_NOTE
```

---

## 14. Weekends

Weekends always count as calendar days when they fall within the sickness absence.

Example:

```text
startDate = Friday
endDate   = Sunday
```

Calculation:

```text
Friday    = day 1
Saturday  = day 2
Sunday    = day 3
```

Result:

```text
NO_FIT_NOTE
```

Another example:

```text
startDate = Friday
endDate   = following Friday
```

Calculation:

```text
calendarDays = 8
```

Result:

```text
FIT_NOTE
```

---

## 15. Bank holidays

Bank holidays count as calendar days.

The application does not need to know whether a particular date is a UK bank holiday.

For example:

```text
startDate = Monday
endDate   = following Monday
```

If one of the dates in between is a bank holiday, it still counts as one calendar day.

The calculation remains:

```text
8 calendar days
→ FIT_NOTE
```

Do not integrate a bank-holiday calendar into the core calculation.

---

## 16. Part-time and irregular workers

The same calculation applies regardless of the worker's normal schedule.

Example:

```text
Worker normally works:
Monday, Wednesday, Friday

Illness:
Monday → Sunday
```

The calculation is:

```text
7 calendar days
```

Result:

```text
NO_FIT_NOTE
```

Do not calculate:

```text
3 working days
```

for the fit-note threshold.

Similarly:

```text
Worker normally works:
Saturday and Sunday

Illness:
Saturday → following Saturday
```

The calculation is:

```text
8 calendar days
```

Result:

```text
FIT_NOTE
```

---

## 17. Required test cases

The implementation should pass all of the following.

| Scenario                      | Start | End           | Calendar days | Result      |
| ----------------------------- | ----- | ------------- | ------------: | ----------- |
| One-day absence               | Mon   | Mon           |             1 | No fit note |
| Two-day absence               | Mon   | Tue           |             2 | No fit note |
| Five-day absence              | Mon   | Fri           |             5 | No fit note |
| Exactly 7 days                | Mon   | Sun           |             7 | No fit note |
| Exactly 8 days                | Mon   | following Mon |             8 | Fit note    |
| Fri–Sun                       | Fri   | Sun           |             3 | No fit note |
| Fri–following Thu             | Fri   | Thu           |             7 | No fit note |
| Fri–following Fri             | Fri   | Fri           |             8 | Fit note    |
| Mon–Fri, return following Mon | Mon   | Sun           |             7 | No fit note |
| Mon–following Mon, return Tue | Mon   | Mon           |             8 | Fit note    |
| Weekend worker, 7 days        | Sat   | Fri           |             7 | No fit note |
| Weekend worker, 8 days        | Sat   | Sat           |             8 | Fit note    |
| Part-time worker, 7 days      | Mon   | Sun           |             7 | No fit note |
| Part-time worker, 8 days      | Mon   | Mon           |             8 | Fit note    |

For test cases described as "following Monday" or "following Friday", the dates must be calculated correctly rather than treating the weekday name alone as an input.

---

## 18. Expected result: 7 days or fewer

For:

```text
calendarDays <= 7
```

display a result equivalent to:

> ### You do not need a fit note based on the length of your sickness absence
>
> Your sickness absence lasted **X calendar days**.
>
> If you are off sick for 7 calendar days or fewer, you can normally self-certify your sickness.

Also explain:

> **Calendar days include weekends, bank holidays and days you do not normally work.**

---

## 19. Expected result: more than 7 days

For:

```text
calendarDays > 7
```

display a result equivalent to:

> ### You should get a fit note
>
> Your sickness absence lasted **X calendar days**, which is more than 7 calendar days.
>
> You should ask a healthcare professional for a fit note.

---

## 20. Do not overstate the legal conclusion

Avoid wording such as:

> “You are legally required to have a fit note.”

The application is implementing the standard **7-calendar-day rule** described by the supplied GOV.UK, NHS and ACAS guidance.

Prefer:

> **“Based on the 7-calendar-day rule, you should get a fit note.”**

and:

> **“You do not need a fit note based on the length of your sickness absence.”**

This avoids implying that the calculator determines every possible requirement an employer or employment arrangement may impose.

---

## 21. Employer policies and other evidence

The calculator should not attempt to determine an employer's individual sickness policy.

The result concerns the fit-note threshold based on the duration of sickness.

An employer may have other sickness-reporting or evidence procedures.

Therefore, include a concise disclaimer such as:

> **This calculator provides general information based on current GOV.UK, NHS and Acas guidance. It does not take account of your employer's individual sickness policy or every possible employment situation. Check with your employer if you are unsure.**

---

## 22. Self-certification

For absences of 7 calendar days or fewer, the application may explain that the worker can normally **self-certify** their sickness.

The application should not generate or imply an official self-certification form.

It should simply communicate the general rule:

```text
<= 7 calendar days
→ normally self-certify
```

---

## 23. No working-day calculation anywhere in the core logic

The following must **not** be used to determine the result:

```text
number of weekdays
number of scheduled shifts missed
number of working days missed
number of contracted days missed
number of days normally worked
```

The following must also not be used:

```text
UK bank holiday calendar
employee's work schedule
part-time/full-time status
weekday/weekend status
```

The only dates needed are:

```text
startDate
endDate
```

---

## 24. Minimal implementation

The application can be implemented with this minimal logic:

```text
function getFitNoteStatus(startDate, endDate):
    if startDate is missing:
        return validation error

    if endDate is missing:
        return validation error

    if endDate is before startDate:
        return validation error

    calendarDays =
        calendarDateDifference(startDate, endDate) + 1

    if calendarDays <= 7:
        return {
            days: calendarDays,
            status: "NO_FIT_NOTE"
        }

    return {
        days: calendarDays,
        status: "FIT_NOTE"
    }
```

No working-day logic should exist inside this function.

---

## 25. Product principle

The application's core principle is:

> **Count consecutive calendar days of sickness, inclusively, from the first day the worker was unable to work to the last day they were unable to work. If the total is 7 days or fewer, the worker can normally self-certify. If the total is more than 7 days, they should get a fit note.**

This principle should be treated as the canonical business rule for the application.
