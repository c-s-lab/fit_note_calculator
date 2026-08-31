# Do I need a fit note? (UK)

A small SvelteKit app that tells you whether you should get a fit note, based on how long you've been off sick.

The rule it applies reflects current GOV.UK, NHS and Acas guidance:

- If you're off sick for 7 calendar days or fewer, you don't need to give your employer a fit note — you can self-certify instead.
- If you're off sick for more than 7 calendar days, you should get a fit note from a registered healthcare professional.

This is a static, frontend-only calculator. It doesn't store or send your data anywhere — the calculation happens entirely in your browser.

## Running locally

```bash
pnpm install
pnpm run dev
```

## What to fill in

The form asks for two dates:

| Field | What to enter |
| --- | --- |
| **First day you were unable to work because you were ill** | The date your sickness absence started — the first day you couldn't work, not the day you first felt unwell if that was earlier. |
| **Last day you were unable to work because you were ill** | The *last* day you were too ill to work — **not** the day you returned to work. If you were off Monday–Friday and back at your desk on Monday, enter that Friday, not the following Monday. |

Both fields are required, and the last day can't be before the first day — the app shows a validation message if either check fails.

You are **not** asked for your normal working pattern (which days you usually work, full-time vs part-time, etc). That's intentional: the 7-day threshold is based on calendar days, including weekends, bank holidays, and any days you don't normally work. Your work schedule has no effect on the result.

## How the result is calculated

The app counts every calendar day from your first day of illness to your last day of illness, inclusive of both ends:

- **7 calendar days or fewer** → you don't need a fit note. You can normally self-certify.
- **More than 7 calendar days** → you should get a fit note from a healthcare professional.

## Examples

### 1. A three-day cold (Monday to Wednesday)

- Start: Monday.
- End: Wednesday.
- Duration: 3 calendar days → **no fit note needed**. You can self-certify with your employer.

### 2. Off sick Monday through the following Sunday

- Start: Monday.
- End: the following Sunday.
- Duration: 7 calendar days → **no fit note needed**, even though this spans a full week on the calendar — the weekend at the end still falls within the 7-day allowance.

### 3. Off sick Monday through the following Monday

- Start: Monday.
- End: the following Monday (8 days later).
- Duration: 8 calendar days → **fit note needed**.

This holds even for someone who only works Monday–Friday and was only scheduled to work 6 of those 8 days — the count is calendar days, not scheduled working days, and crossing into day 8 triggers the fit note requirement regardless of the worker's usual pattern.

### 4. Part-time worker (e.g. works Tuesdays and Thursdays only), off sick for 9 calendar days

Even though this worker was only scheduled to work 2–3 days during that window, the calculation still counts calendar days rather than scheduled work days — so 9 calendar days → **fit note needed**.

## Sources

- [GOV.UK — The fit note: guidance for patients and employees](https://www.gov.uk/government/publications/the-fit-note-a-guide-for-patients-and-employees/the-fit-note-guidance-for-patients-and-employees)
- [NHS — Getting a fit note](https://www.nhs.uk/nhs-services/gps/getting-a-fit-note/)
- [Acas — Fit notes and proof of sickness](https://www.acas.org.uk/fit-notes-and-proof-of-sickness)

## Disclaimer

This app provides general information based on current GOV.UK, NHS and Acas guidance. It doesn't account for your employer's individual sickness policy or every possible employment situation — check with your employer if you're unsure.
