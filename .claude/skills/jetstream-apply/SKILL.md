---
name: jetstream-apply
description: Research and fill out Brennan's Operation Jetstream job applications in his Chrome, stopping before Submit. Use when Brennan says "fill out my Jetstream picks", "apply to my picks", or names a job from the Jetstream tracker to apply to.
---

# Operation Jetstream: research and fill applications

Brennan picks jobs in the tracker (https://claude.ai/artifact/8J323njpmtgdyiRyF7U6Ds) by tapping *Apply to this*.
This skill researches each pick's company and fills its application in his own Chrome, where his job-site logins are.
He always submits himself.

## 1. Load the picks

- Load the ArtifactData tool (ToolSearch) and read `config/profile` (resume details, `candidate.resumeFile`).
- Query `jobs` where `status == "selected"` and `stage` is `Researching` or `Ready to apply` (skip jobs already
  Applied or later). If Brennan named one job, do only that one.
- Tell him which jobs you'll work on, then start without waiting.

## 2. For each job

Use Claude in Chrome (load its tools per the chrome-browser skill). Work in new tabs; don't touch his other tabs.

1. **Research the company's website.** Find the official site and read up to 8 pages: home, about, mission or values,
   culture or careers, leadership, recent news. Note their culture, mission, stated values and the exact words they
   repeat. Also use `research` and `research.postingKeywords` from the job if present.
2. **Fill the application.** Open the job's `url`, go to the application and fill every page from `config/profile`.
   - Resume: upload `candidate.resumeFile` (`/Users/Bnolan98/Desktop/Jetstream/Brennan_Nolan_Resume.docx`). If the
     upload dialog can't be driven, stop on that field and ask Brennan to pick the file.
   - Cover letter: write one for this job that echoes their values and uses their words and the posting's keywords
     naturally, only where his real experience backs them up. Never claim experience he doesn't have. No keyword
     stuffing.
   - Ask Brennan, don't guess: salary expectations, work authorization or sponsorship, start date, references,
     demographic or disability questions, anything legal.
   - Account creation or login needed: stop and ask him to sign in, then continue.
   - **Stop before the final Submit/Apply button.** Leave the tab open on the review page.
3. **Save to the tracker** (one pinned `update` to `jobs/<id>`): `websiteResearch` {culture, mission, values[],
   keywords[], news[], pagesRead[], readAt}, `coverLetter` (full text), `answers` (every question and what you entered),
   `accountSite` (if an account was needed), `stage: "Ready to apply"` with a `stageHistory` entry if it changed. Don't
   set `Applied`; Brennan does that after he submits.

## 3. Finish

List each job: its tab, anything you need from him, and "ready for your review". Remind him to set the stage to
Applied in the tracker after he submits (or the morning run will pick it up from the confirmation email).
