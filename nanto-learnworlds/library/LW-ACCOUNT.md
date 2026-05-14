# LearnWorlds Account — Nanto Academy

## Account Config

| Field | Detail |
|---|---|
| **Plan** | Learning Center |
| **Learner-facing URL** | https://academy.nanto.com |
| **Master admin (multi-school)** | https://account.learnworlds.com/dashboard/schools |
| **School admin dashboard** | https://academy.nanto.com/author/dashboard |
| **Courses list** | https://academy.nanto.com/author/courses |
| **School created** | 26 Mar 2026 |
| **Brand** | Nanto Amber #F59E0B / Near black #141414 / White #FFFFFF |

## Saved Browser State

State saved as `learnworlds-admin` (146 cookies). Load with:

```bash
$B state load learnworlds-admin
$B goto https://academy.nanto.com/author/dashboard
```

If session expired, navigate to https://account.learnworlds.com/login and log in, then save again:
```bash
$B state save learnworlds-admin
```

## Active Clients / Cohorts

| Client | Status | Notes |
|---|---|---|
| Brookside | First live pilot | Multiple courses live |
| Megna Homes (MPM) | Trial | 2-stage model: SOP dev then digitise |

## Existing Courses (as at 13 May 2026)

| Title | titleId (for URL) | id |
|---|---|---|
| Root Cause Analysis for Aviation Operations | root-cause-analysis-for-aviation-operations | 69f80eda106afab51208f15e |
| Disciplinary Procedure at Brookside | disciplinary-procedure-at-brookside | 69ea4c5e13c338f4320a5dc7 |
| M-Pesa & Mobile Money Fraud: Employee Awareness | m-pesa-and-mobile-money-fraud-employee-awareness | 69d5db830065210de9082024 |
| Disciplinary Action in Kenya | disciplinary-action | 69cd638ba7cb131bd30b8db4 |
| Know Brookside: Our Story, Our Brands, Our Products | know-brookside | 69c974d92ef5dbcbcc09736e |
| Welcome to Brookside: Your First 30 Days | welcome-to-brookside-your-first-30-day | 69c96f8c149bf8627303ca0b |
| Food Handler Hygiene & PPE | compliance-food-handler-hygiene | 69c9671a3143749ca208e6e4 |

Get current list from API (run in browser JS console or $B js):
```js
fetch('/api/author/products?fields=id,title,titleId,type&paginationType=paginated&page=1&itemsPerPage=20&type=course&sortField=created&sortDirection=desc').then(r=>r.json()).then(d=>console.log(JSON.stringify(d.data)))
```

## Course Library Path

```
/Users/darack/Library/CloudStorage/ZohoWorkDriveTrueSync-NantoHQ/10-PROJECTS/ACADEMY/COURSE-LIBRARY
```

Zips are in `[COURSE_CODE]-[ShortName]/SCORM-Packages/Zips/`.
