---
name: offispace-rhq-pre-announcement-study
description: "Retrospective study of what happened in the ~90 days (and up to a year) BEFORE companies announced their Riyadh office / regional HQ (RHQ), using LinkedIn Sales Navigator in Chrome plus web search. Rebuilds each company's pre-announcement timeline (Saudi hires, relocations, set-up roles, posts, news) and distils the common pattern into a playbook of early signals. Use when Khaled asks what happened before a company announced its Riyadh office, to learn from past RHQ openings, back-test or study pre-announcement signals."
---

# RHQ Pre-Announcement Study (learn from past Riyadh openings)

Question this answers: **for companies that already announced a Riyadh office/RHQ, what was
visible on LinkedIn and in the news in the 90 days before the announcement — and is there a
pattern Offispace can watch for next time?**

Runs in Chrome via the `mcp__claude-in-chrome__*` tools. If deferred, load them in one call:

`ToolSearch` with query `"select:mcp__claude-in-chrome__tabs_context_mcp,mcp__claude-in-chrome__navigate,mcp__claude-in-chrome__computer,mcp__claude-in-chrome__read_page,mcp__claude-in-chrome__tabs_create_mcp,mcp__claude-in-chrome__tabs_close_mcp,mcp__claude-in-chrome__find,mcp__claude-in-chrome__get_page_text,mcp__claude-in-chrome__javascript_tool"`

Call `tabs_context_mcp` first and work in a new tab. Follow the UI-reliability rules in
`offispace-linkedin-lead-search` section 8 (verify filters with `get_page_text`, use `find` for
"Include" buttons, re-apply filters that silently clear). Resolve each company's exact LinkedIn
name with that skill's step 1.

## Ground rules (account safety)

Read-only: no InMails, connection requests, follows or saved leads. Human pace, no bulk export,
no scripted pagination. Per company cap ~5 result pages and ~10 profile opens; do **5 companies
per run** and continue in the next run. Stop and tell Khaled on any CAPTCHA or usage warning.

## Case list (seed — confirmed 2025 openings)

`T0` is the **first public mention** of the Riyadh office (licence news or plan announcement),
not the inauguration — verify it in step 1 because some differ a lot (Citi: licence Nov 2024,
opening Oct 2025).

| Company | Opening/launch reported | Notes |
|---|---|---|
| Salesforce | 2025-01-21 | plan announced |
| NetApp | 2025-02-01 | functional date; find announcement date |
| Bain & Company | 2025-04/05 | |
| BNY | 2025-05-01 | licence |
| PepsiCo | 2025-05 | KAFD |
| Globant | 2025-05-27 | |
| Wipro | 2025-06-09 | moved from Al Khobar — existing KSA base |
| Ericsson | 2025-07-01 | existing KSA base |
| Hanwha Aerospace | 2025-09-03 | |
| Plaza Premium Group | 2025-09-14 | existing KSA ops |
| Tarabut | 2025-09-30 | |
| EY MENA | 2025-09-30 | existing KSA base |
| SAS | 2025-10-09 | |
| Citi | 2025-10-25 | licence 2024-11 |
| State Street | 2025-10-27 | |
| ESL FACEIT Group | 2025-11-05 | |
| EFFY | 2025-11-06 | |
| bioMérieux | 2025-11-25 | |
| SentinelOne | 2025-12-01 | |
| Goldman Sachs | 2025-12-11 | |

Tag each case **new entrant** (little/no KSA presence before) or **existing base** (already had a
Saudi office). Patterns differ; analyse the two groups separately. Prefer new entrants first.

## Per-company procedure

### 1. Pin T0 (web search)
Search `"<company>" Riyadh "regional headquarters"` and `"<company>" Saudi licence MISA`. Take the
earliest dated article as T0; record date + URL. Also note the inauguration date if different.

### 2. Who was in Saudi Arabia, and since when (Sales Navigator)
`https://www.linkedin.com/sales/search/people` → **Current company** = X, **Geography** = Saudi
Arabia. Record the total result count (`ksa_now`). Then extract each card:

```js
Array.from(document.querySelectorAll('li.artdeco-list__item')).map(li => ({
  text: li.innerText.replace(/\s+/g,' ').slice(0,400),
  lead: li.querySelector('a[href*="/sales/lead/"]')?.href
}))
```

From each card's "X years Y months in role" text compute `start_month = today − time_in_role`.
Keep only people whose start month falls between **T0−12 months and T0+3 months**.

Repeat with **Past company** = X + Geography = Saudi Arabia to catch people who joined for the
set-up and have since left (survivorship check).

### 3. Confirm the important hires (open profiles, max ~10)
Open profiles for: country head / GM / MD, Director+, government affairs, and set-up roles.
Record exact start month, **previous role and its location** (relocated from Dubai? internal
transfer? hired from a Saudi competitor?), and whether the title says KSA/Saudi/MENA.

Classify each hire:

| Code | Category |
|---|---|
| LEAD | Country head / GM / MD / Regional Director based in KSA |
| GOV | Government affairs / relations, public policy, GRO/PRO, Saudization |
| SETUP | Office manager, EA, HR, finance/tax (ZATCA), legal, compliance |
| WORK | Facilities, workplace, real estate, fit-out (Offispace's buyer) |
| FRONT | Sales, client delivery, consultants |

### 4. Public activity before T0 (web + LinkedIn)
- Web search for dated news in T0−180..T0: executive appointments for KSA, MoUs at LEAP/FII,
  MISA meetings, office lease/tower news (KAFD, Kingdom Tower, Laysen Valley, Roshn Front).
- `site:linkedin.com/posts "<company>" Riyadh` and the key hires' names + "Riyadh" — note posts
  dated before T0 about relocating, hiring, office search, or the RHQ licence.
- Historical job posts are not visible on LinkedIn once closed; only record them if a dated
  copy is found on the web. Note "not observable" rather than "none".

### 5. Build the timeline
Bucket every event by months relative to T0: **T−12..T−7**, **T−6..T−4**, **T−3..T−1 (the 90-day
window)**, **T0..T+3**. One row per event in `rhq-study-evidence.csv`:
`company,group,T0,event_date,months_from_T0,category,title,prev_location,source_url,confidence`

`confidence`: high = profile start month + dated source; medium = time-in-role estimate only;
low = inferred.

## Cross-company synthesis (after ≥8 companies)

Write `rhq-study-findings.md` with:

1. **Sequence:** the typical order of categories (e.g. LEAD → GOV → SETUP → WORK → T0), with the
   median months-before-T0 for each first occurrence.
2. **90-day window profile:** what % of companies showed each category in T−3..T−1, and the
   median number of KSA hires in that window.
3. **Earliest reliable signal:** the category that appeared before T0 in the most companies, and
   its median lead time.
4. **Relocation share:** how many leaders moved from Dubai/elsewhere vs local hires.
5. **New entrant vs existing base:** where the patterns differ.
6. **Offispace angle:** when the WORK/facilities person appeared relative to T0 — i.e. how early
   a fit-out conversation was possible, and who it was with.
7. **Caveats:** no control group (companies that hired in Riyadh but never announced) — suggest 5
   to add next; time-in-role is month-level and promotions reset it; profiles are updated late.

Report to Khaled in chat: the 5–7 headline findings with counts, each tied to example companies
and evidence URLs, plus the remaining case list for the next run.
