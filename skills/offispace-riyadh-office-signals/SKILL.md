---
name: offispace-riyadh-office-signals
description: "Detect foreign companies likely to open a Riyadh office / regional HQ (RHQ) roughly 90 days before they announce it, using LinkedIn Sales Navigator in Chrome. Scores hiring, relocation, headcount and posting signals into Hot/Warm/Watch tiers and hands Hot accounts to offispace-linkedin-lead-search. Use when Khaled asks to find companies about to open in Riyadh, run the RHQ/office-opening signal scan, early-warning or pre-announcement leads, or refresh the Riyadh watchlist."
---

# Riyadh Office-Opening Signal Scan (Sales Navigator)

Goal: find companies **before** they announce a Riyadh office/RHQ, so Offispace reaches the
facilities/fit-out buyer while the space is still being chosen. An announcement or inauguration
is the *end* of the process — the hiring and relocations that precede it are visible on LinkedIn
months earlier.

Runs in Chrome via the `mcp__claude-in-chrome__*` tools. If deferred, load them in one call:

`ToolSearch` with query `"select:mcp__claude-in-chrome__tabs_context_mcp,mcp__claude-in-chrome__navigate,mcp__claude-in-chrome__computer,mcp__claude-in-chrome__read_page,mcp__claude-in-chrome__tabs_create_mcp,mcp__claude-in-chrome__tabs_close_mcp,mcp__claude-in-chrome__find,mcp__claude-in-chrome__get_page_text,mcp__claude-in-chrome__javascript_tool"`

Call `tabs_context_mcp` first and work in a new tab. Reuse the UI-reliability rules from
`offispace-linkedin-lead-search` section 8 (verify every filter with `get_page_text`, use `find`
for "Include" buttons, re-apply filters that silently clear).

## Ground rules (account safety)

- Read-only. Never send InMails, connection requests, follows, or "save lead" on people unless
  Khaled asks. Saving **accounts** to the watchlist (below) is fine.
- Human pace: one action at a time, no bulk export, no scripted pagination loops. Cap a run at
  ~10 result pages and ~40 account/profile opens. Stop and tell Khaled immediately on any
  CAPTCHA, "unusual activity" or usage-limit warning.

## Why ~90 days works (the pre-announcement sequence)

An RHQ licence requires ≥15 full-time staff incl. ≥3 senior executives within year one, so set-up
leaves a hiring trail. Typical order (hypothesis — tune with the back-test in step 0):

| Lead time before announcement | What shows up on LinkedIn |
|---|---|
| 6–9 months | Country/GM/Regional MD for KSA appointed or relocates from Dubai/Cairo/London to Riyadh; Government Affairs / GRO hire |
| 3–6 months | Riyadh job posts for set-up roles: Office Manager, EA to GM, HR/Saudization, Finance/ZATCA tax, Legal; Saudi headcount jumps |
| 1–3 months | Workplace/Real-estate/Facilities roles touching KSA; employees post about "relocating to Riyadh", MISA meetings, LEAP/FII |
| 0 | Licence / opening announced (too late — exclude) |

## Inputs

- **Mode:** `discover` (find new candidates, default) or `monitor` (re-score the watchlist).
- **Watchlist file:** `riyadh-signals-watchlist.csv` in the working folder. Create it if missing
  with columns: `company,linkedin_url,hq_country,first_seen,last_scan,ksa_headcount,score,tier,evidence`.
- **Exclusion list:** companies that already announced/opened a Riyadh RHQ (they are no longer
  "pre-announcement"). Keep it at the bottom of the watchlist file under `# excluded`. Seed:
  Citi, Goldman Sachs, State Street, BNY, PepsiCo, Ericsson, Wipro, Globant, SAS, SentinelOne,
  Salesforce, EY, PwC, Deloitte, Bain, Bechtel, Northern Trust, IHG, Tarabut, Plaza Premium Group,
  ESL FACEIT Group, EFFY, bioMérieux, Hanwha Aerospace, NetApp, Moody's, Deutsche Bank, Lenovo,
  Mannai, Yalla, Morgan Stanley, JPMorgan, HSBC.

## 0. Back-test once before trusting scores (first run only)

Pick 3 companies from the exclusion list with known 2025 dates (SentinelOne – Dec 1 2025,
SAS – Oct 9 2025, State Street – Oct 27 2025). For each: People search, Current company = X,
Geography = Saudi Arabia; open the senior profiles and note **start date of the Riyadh role**.
Record how many Director+ and set-up roles started 3–9 months before the announcement. If the
pattern differs from the table above, adjust the weights in step 4 and write the change into
this file's "Tuning log" section.

## 1. Discover mode — new senior people in Riyadh at foreign companies

`https://www.linkedin.com/sales/search/people`, filters:

- **Geography:** Riyadh, Saudi Arabia (second pass: Saudi Arabia)
- **Changed jobs:** in past 90 days
- **Seniority level:** CXO, VP, Director, Owner/Partner
- **Company headquarters location:** **Exclude** Saudi Arabia
- **Company headcount:** 201+ (multinationals; drop to 51+ for a wider sweep)

Run these title passes as **Current job title** (one at a time):
1. Country Manager / General Manager / Managing Director — with KSA, Saudi, or no qualifier
2. Head of Saudi Arabia / Regional Director / Head of Middle East
3. Government Affairs / Government Relations / Public Policy
4. Head of HR / Finance Director / Head of Legal (KSA)

Extract result rows with `javascript_tool`:

```js
Array.from(document.querySelectorAll('li.artdeco-list__item')).map(li => ({
  text: li.innerText.replace(/\s+/g,' ').slice(0,300),
  lead: li.querySelector('a[href*="/sales/lead/"]')?.href,
  account: li.querySelector('a[href*="/sales/company/"]')?.href
}))
```

Group by company. Drop excluded companies and Saudi-HQ companies the filter missed. Every
remaining company is a **candidate** → add to watchlist (`first_seen` = today) and score it.

Supplementary pass (optional): LinkedIn content search
`https://www.linkedin.com/search/results/content/?keywords=` with
`"relocating to Riyadh"`, `"our new office in Riyadh"`, `"regional headquarters" Riyadh`,
`MISA licence`, sorted by Latest, past month. Note company + post URL; ignore posts that are
themselves the announcement.

## 2. Monitor mode — re-score watchlist accounts

For each watchlist company (not excluded):

1. **KSA headcount:** People search, Current company = X, Geography = Saudi Arabia → read the
   result count. Compare with `ksa_headcount` from the last scan.
2. **Recent KSA joiners:** same search + Changed jobs in past 90 days → list titles + start dates.
3. **Riyadh job posts:** account page → Jobs/"Hiring" link, or
   `https://www.linkedin.com/jobs/search/?keywords=<company>&location=Riyadh` → list titles + post dates.
4. **Account alerts / insights:** account page Insights → senior hires, headcount growth by
   function; check whether any senior hire is located in Saudi Arabia.
5. **Already announced?** Quick web search `"<company>" Riyadh "regional headquarters"`. If it's
   announced, move the company to `# excluded` and stop scoring it.

Save each new candidate to a Sales Navigator **account list** named "Riyadh Office Signals" so
LinkedIn's own account alerts (senior hires, headcount, news) accumulate between runs.

## 3. What counts as a signal

Only count evidence with a date in the last 180 days, tied to Saudi Arabia/Riyadh:

| Code | Signal | Points |
|---|---|---|
| S1 | Director+ starts a Riyadh/KSA role (new hire or relocation) | 25 each, cap 40 |
| S2 | KSA-titled country head (GM/MD/Country Manager KSA) | +15 bonus |
| S3 | Government Affairs / GRO / Saudization hire in KSA | 15 |
| S4 | Riyadh job posts for set-up roles (Office Mgr, EA, HR, Finance/Tax, Legal, PRO) | 10 each, cap 30 |
| S5 | Facilities / Workplace / Real-estate / Fit-out role tied to KSA | 20 (also an Offispace hot lead) |
| S6 | KSA headcount: ≤5 → ≥10, or +30% since last scan | 20 |
| S7 | Employee posts on relocating to Riyadh, MISA, RHQ, new office | 10 each, cap 20 |

Decay: evidence 91–180 days old counts half. Lone S7 posts with no S1–S6 cap the score at 15.

Tiers: **Hot ≥ 60**, **Warm 35–59**, **Watch 15–34**, below 15 drop from the report (keep in file).

## 4. Output

Update the CSV, then report to Khaled, Hot first:

| Company | HQ | Tier (score) | Key evidence (dated, with URLs) | Est. announcement window | Next step |
|---|---|---|---|---|---|

- **Est. window:** S1/S2 only → 3–9 months; S4/S5/S6 present → 0–3 months.
- **Next step for Hot:** run `offispace-linkedin-lead-search` on the company, scoped to
  Geography = Saudi Arabia, to find the facilities/fit-out champion (or the Dubai-based regional
  facilities lead running the set-up).
- Flag low-confidence calls plainly (e.g. "one relocation, could be a sales hire, not an office").

## Scheduling

Best run weekly (e.g. Sunday morning Riyadh time) as a Cowork scheduled task: `discover` then
`monitor`. Signals are relative to the previous snapshot, so keep the same watchlist file.

## Tuning log

- (empty — record weight changes from the back-test and from hits/misses here, with dates)
