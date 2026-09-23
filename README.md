# JS: Operation Jetstream

Brennan's personal job search system: a daily report of new marketing jobs within 45 minutes of Frederick, MD
(21701) plus remote, each scored against his resume, and a tracker for every application after that.

| Path | What it is |
|---|---|
| `app/jetstream.html` | The report and tracker page, published as a private Claude artifact with its own database: https://claude.ai/artifact/8J323njpmtgdyiRyF7U6Ds |
| `config/profile.json` | Search rules and resume data the matching uses |
| `docs/morning-run.md` | The steps the daily morning run follows, and the job record format |

## How it works

1. **Morning report.** Every morning Claude collects jobs from forwarded LinkedIn, Indeed, Glassdoor and
   ZipRecruiter alerts, watch-list companies' careers pages, USAJOBS, Frederick Works and Frederick government
   boards. It drops sales roles, anything over a 45-minute drive, and anything paying under $100K, then scores
   what's left and posts only the new jobs.
2. **Pick.** Brennan taps *Apply to this* or *Pass* on each card. Passed jobs never come back.
3. **Research.** Claude researches each picked company and saves a profile on the job's tracker row.
4. **Track.** Stages, dates, 14-day follow-ups, contacts, interviews, salary and notes, with monthly stats.
5. **Fill.** Using Claude in Chrome, Claude opens one tab per job and fills every page of the application from the
   resume, writes a tailored cover letter, and stops before the final Submit button for Brennan to review.
