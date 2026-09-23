# Operation Jetstream: morning run

The steps Claude follows each morning to build Brennan's job report. The report and tracker live in the
Operation Jetstream artifact; every step below reads or writes that artifact's database.

Search rules and resume data: `config/profile.json` (mirrored in the database at `config/profile`).

## 1. Collect

| Source | How |
|---|---|
| LinkedIn, Indeed, Glassdoor, ZipRecruiter | Gmail `brennan@bnolandesign.com`, label `Jetstream`, emails since the last run. Alerts are forwarded from `brennolan98@gmail.com`. If an alert says there are more jobs than it lists ("See all 20 jobs"), add a warning to the run. |
| Application confirmations | Same label: `indeedapply@indeed.com`, LinkedIn "application was sent". Match to a tracker job and set stage `Applied` with that date. |
| Watch list companies | `watchCompanies` collection. Detect the careers page's job system on first sight (Greenhouse, Lever, Ashby, Workday, other), store it as `ats`, then pull the full list of open jobs each run. |
| USAJOBS | `data.usajobs.gov/api/search`, location 21701, radius 40 miles, plus remote. |
| Frederick County Workforce Services | `frederickworks.com/job-openings/` |
| Frederick County Government, City of Frederick | Their governmentjobs.com boards. |
| Dice | Dice connector, marketing titles, 21701 radius plus remote. |

## 2. Filter

Drop a job when any of these holds:

1. **Already seen.** The document ID is a hash of normalized company + title + location. If `jobs/<id>` exists in
   any status (new, selected, dismissed), skip it. Dismissed jobs never come back, even when reposted under a new link.
2. **Not a target title.** Match `search.targetTitles`, including Senior and Director variants.
3. **Sales.** Any `salesExclusionSignals` in the description. Account Executive, Account Manager and Account
   Strategist stay only at an agency or when the duties are campaigns, brand, strategy or client marketing.
4. **Product Manager not about marketing.** Keep only when the description is mainly positioning, go-to-market,
   campaigns, brand or customer insight.
5. **Too far.** Geocode the job location, get the driving time from 21701, and drop anything over 45 minutes.
   Remote jobs skip this check. Hybrid jobs must pass it.
6. **Too little pay.** Drop only when the top of the posted range is under $100,000 a year. Before calling a salary
   not listed, check the full posting, the company careers page, the Maryland/DC pay-range disclosure and a web search.

Count every drop by reason for the run record.

## 3. Score and describe

For each job that passes, write a card with a 0–100 match score from the resume in `config/profile`:

- Title and seniority fit (about 7 years; Senior/Manager is the sweet spot; flag Director roles asking for 10+
  years or large teams as `stretch: true`)
- Overlap between the duties and resume skills and proof points
- Boosts: food & beverage (`foodBev: true`), watch-list company (`watched: true`), agency
- 1-sentence `fitSummary`, 2–3 `fitPros`, 1–3 `fitCons`. Never claim experience the resume doesn't show.

Look up the Glassdoor rating. Leave it empty when no rating card is found.

## 4. Write

- One `jobs/<id>` document per new job with `status: "new"` and `reportDate` set to today.
- One `runs/<YYYY-MM-DD>` document: `date`, `time`, `newCount`, `filteredOut`, `dropReasons` (map),
  `sources` (array of `{name, found, kept}`), `warnings` (array of strings).
- Tracker upkeep: jobs in stage `Applied` with no response `markNoResponseAfterDays` (30) after `appliedAt`
  move to stage `No response`.

## Job document

```json
{
  "status": "new | selected | dismissed",
  "reportDate": "2026-09-24",
  "title": "Senior Brand Manager",
  "company": "Example Foods Co.",
  "url": "https://…",
  "source": "LinkedIn alert | Indeed alert | Glassdoor alert | ZipRecruiter alert | Greenhouse | Lever | Ashby | USAJOBS | Frederick Works | Frederick County Gov | Dice | Manual",
  "postedAt": "2026-09-22",
  "location": "Frederick, MD",
  "workplace": "On-site | Hybrid · 3 days | Remote",
  "driveMinutes": 12,
  "salaryMin": 105000,
  "salaryMax": 125000,
  "salaryPeriod": "year",
  "salarySource": "posting | careers page | MD pay disclosure | estimate",
  "salaryText": "free text when the range can't be parsed",
  "benefits": ["Medical/dental/vision", "401(k) match"],
  "glassdoorRating": 4.1,
  "glassdoorUrl": "https://…",
  "matchScore": 86,
  "fitSummary": "…",
  "fitPros": ["…"],
  "fitCons": ["…"],
  "foodBev": true,
  "watched": false,
  "stretch": false,
  "description": "full posting text, saved so it survives the posting being taken down",
  "postingSnapshotAt": "2026-09-24"
}
```

Fields the tracker adds as Brennan works a job: `selectedAt`, `dismissedAt`, `stage`, `stageHistory`,
`appliedAt`, `followUpAt` (applied + 14 days), `heardBack`, `respondedAt`, `lastContactAt`, `deadline`,
`accountSite`, `salaryAsked`, `salaryOffered`, `resumeVersion`, `coverLetter`, `contacts`, `interviews`,
`answers`, `notes`, and `research` (`{summary, …}`, written by Claude after a job is selected).
