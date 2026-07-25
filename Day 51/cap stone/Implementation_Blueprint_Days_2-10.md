# Implementation Blueprint — Admissions Lead Tracker
### AB Talks 60-Day Claude AI Challenge | 10-Day Capstone | Days 2–10

**How to use this document:** Each day below is self-contained. If a day's conversation starts fresh, paste that day's section (or the whole file) to the AI assistant as context, along with the actual code/files produced by the previous day. This blueprint is the single source of truth — the daily AI should build from it, not redesign it.

**Project recap:** A single-admin, browser-based Admissions Lead Tracker. Leads move through 5 stages (Inquiry → Visit → Application → Test/Interview → Admitted/Rejected). Core feature: a "Today's Follow-ups" dashboard so no lead is ever forgotten. Data lives in browser local storage — no backend, no login, no multi-user. Dark/premium visual style, consistent with the builder's existing single-file HTML app portfolio. Day 10 = live, deployed, polished v1.0.

**Locked scope (do not expand):** No multi-user, no cloud sync, no CSV import/export, no SMS/WhatsApp/email integration, no analytics beyond basic funnel counts. If a day's conversation drifts toward any of these, redirect back to this document.

---

## Day 2 — Tech Stack Decision, Architecture & Environment Setup

**🎯 Objective:** Choose the tech stack, set up the project skeleton, and get a blank app running.

**📖 What I'll learn:** How to choose a stack for a small single-user app; project scaffolding; local development workflow.

**🛠 Features to build:** None yet — this is setup only.

**📝 Step-by-step implementation plan:**
1. Decide the stack. Recommended default (single-file, no backend, matches existing portfolio): a single `index.html` file using vanilla JavaScript + CSS, with data persisted via `localStorage`. This avoids build tools, npm, and deployment complexity entirely — appropriate for <100 records and a single user.
   - Alternative (if more structure is wanted): React via CDN + Babel (as used in prior portfolio apps like Supply Chain Builder). Only choose this if comfortable debugging JSX-in-browser issues; the blueprint defaults to vanilla JS for reliability within 10 days.
2. Create the project folder structure (see Files section).
3. Write a minimal `index.html` with a title, dark background, and a placeholder heading to confirm styling loads.
4. Open the file directly in a browser to confirm it renders (no server needed for a single HTML file).
5. Initialize a git repository and make the first commit.
6. Create a GitHub repository and push the initial skeleton (for portfolio visibility, per the 60-day challenge).

**📂 Files and folders to create or modify:**
```
admissions-lead-tracker/
  index.html        (main app shell)
  style.css         (or inline <style> in index.html)
  app.js            (or inline <script> in index.html)
  README.md
```

**🔗 APIs, libraries, services, or tools to integrate:** None. Deliberately zero dependencies at this stage.

**🧪 Testing tasks:** Open `index.html` in at least two browsers (e.g., Chrome and Safari/Firefox) to confirm it loads without console errors.

**🐞 Common issues and debugging tips:**
- Blank page → check browser console for JS errors first.
- Styles not applying → confirm `<link>` path or `<style>` block is inside `<head>`.
- "It works on desktop but not iOS Safari" — per prior portfolio experience, avoid CDN React/JSX for this project; vanilla JS sidesteps this entirely.

**✅ End-of-day checklist:**
- [ ] Stack decision made and written down (one sentence, in README)
- [ ] Project folder created with the files above
- [ ] Blank styled page renders in browser with no console errors
- [ ] Git repo initialized, first commit made
- [ ] GitHub repo created and pushed

**📸 Expected project state and screenshots to capture:** Screenshot of the blank dark-themed page loaded in the browser; screenshot of the GitHub repo page.

**➡️ Handoff notes for Day 3:** Stack = vanilla HTML/CSS/JS, single-file app, localStorage for persistence. Folder structure exists. Next day builds the data model and core lead CRUD.

---

## Day 3 — Data Model & Core Lead CRUD

**🎯 Objective:** Build the underlying lead data model and the ability to add, view, edit, and delete leads.

**📖 What I'll learn:** Structuring application state in JavaScript; reading/writing `localStorage`; basic CRUD patterns.

**🛠 Features to build:** Add Lead form; Lead List view; Edit Lead; Delete Lead.

**📝 Step-by-step implementation plan:**
1. Define the lead data shape in `app.js`:
   ```js
   {
     id: string,          // unique, e.g. Date.now() + random
     name: string,
     phone: string,
     grade: string,       // grade applying for
     stage: string,       // one of the 5 funnel stages
     nextFollowUp: string,// ISO date string
     notes: string,
     createdAt: string
   }
   ```
2. Write `loadLeads()` and `saveLeads()` functions wrapping `localStorage.getItem`/`setItem` with `JSON.parse`/`stringify`.
3. Build an "Add Lead" form (name, phone, grade, next follow-up date, notes; stage defaults to "Inquiry").
4. Render a simple list/table of all leads (name, phone, grade, stage, next follow-up).
5. Add Edit functionality (click a lead → form pre-filled → save updates the record).
6. Add Delete functionality with a confirmation step.
7. Ensure every change immediately calls `saveLeads()` so data survives a refresh.

**📂 Files and folders to create or modify:**
```
app.js       (data model, CRUD functions, render logic)
index.html   (add form markup, list container)
style.css    (basic form/list styling — dark theme, minimal for now)
```

**🔗 APIs, libraries, services, or tools to integrate:** Browser `localStorage` API only.

**🧪 Testing tasks:**
- Add 5 test leads, refresh the browser, confirm they persist.
- Edit a lead's phone number, refresh, confirm the change persists.
- Delete a lead, refresh, confirm it's gone.
- Try submitting the Add form with empty required fields — confirm basic validation prevents it.

**🐞 Common issues and debugging tips:**
- Data disappears on refresh → confirm `saveLeads()` is actually called after every add/edit/delete, not just on page unload.
- `JSON.parse` throws on first load → guard with a check for `null`/empty localStorage value before parsing.
- Duplicate IDs → use `Date.now() + Math.random()` or a simple incrementing counter stored alongside the leads.

**✅ End-of-day checklist:**
- [ ] Lead data model defined
- [ ] Add Lead form works and saves to localStorage
- [ ] Lead list renders all saved leads
- [ ] Edit lead works and persists
- [ ] Delete lead works and persists
- [ ] Data survives a full page refresh

**📸 Expected project state and screenshots to capture:** Screenshot of the Add Lead form filled in; screenshot of the lead list with 5+ test leads.

**➡️ Handoff notes for Day 4:** Lead CRUD is fully functional with localStorage persistence. Data model is locked — do not change field names without updating all render/save functions. Next day builds the 5-stage funnel view and stage-moving UI.

---

## Day 4 — Funnel/Pipeline View & Stage Management

**🎯 Objective:** Let the admin see leads grouped by funnel stage and move leads between stages.

**📖 What I'll learn:** Rendering grouped/filtered views from a single data source; simple drag-free stage-change UI patterns.

**🛠 Features to build:** Funnel view (5 columns or grouped sections, one per stage, each showing lead count and lead cards); "Move to next stage" / stage-select control per lead.

**📝 Step-by-step implementation plan:**
1. Define the fixed stage order as a constant array: `["Inquiry", "Visit", "Application", "Test/Interview", "Admitted", "Rejected"]` (note: Admitted/Rejected are two separate terminal stages, both reachable from Test/Interview).
2. Build a funnel view: one column/section per stage, filtering the full lead list by `stage` and rendering a compact card per lead (name, phone, next follow-up).
3. Show the count of leads in each stage as a header on each column.
4. Add a stage-change control on each lead card (a dropdown or "Move to →" button) that updates `lead.stage` and calls `saveLeads()`.
5. Add a toggle or nav link between "Funnel View" and the "All Leads" list view from Day 3.
6. Style Admitted/Rejected stages distinctly (e.g., muted/archived look) since they're terminal.

**📂 Files and folders to create or modify:**
```
app.js       (add funnel rendering functions, stage constants, moveStage function)
index.html   (add funnel view container, view toggle)
style.css    (column/card layout for funnel — flexbox or CSS grid)
```

**🔗 APIs, libraries, services, or tools to integrate:** None new.

**🧪 Testing tasks:**
- Add leads across different stages, confirm each appears in the correct column.
- Move a lead from Inquiry → Visit, confirm it moves columns and count updates.
- Move a lead to Admitted, confirm it renders in the terminal/archived style.
- Refresh and confirm stage assignments persist.

**🐞 Common issues and debugging tips:**
- Counts not updating → make sure the render function re-reads from the current in-memory leads array after every state change, not a stale copy.
- Layout breaks on narrow screens → use CSS grid with `overflow-x: auto` or stack columns vertically below a breakpoint (relevant since admin may check on a phone).

**✅ End-of-day checklist:**
- [ ] Funnel view renders all 5+ stages with correct lead counts
- [ ] Leads can be moved between stages and it persists
- [ ] Admitted/Rejected leads are visually distinguished as terminal
- [ ] Toggle between Funnel View and All Leads list works

**📸 Expected project state and screenshots to capture:** Screenshot of the full funnel view with leads distributed across multiple stages.

**➡️ Handoff notes for Day 5:** Funnel view and stage transitions are complete. Stage constant array is defined in `app.js` — reuse it, don't hardcode stage names elsewhere. Next day builds the "Today's Follow-ups" dashboard, the core value feature.

---

## Day 5 — "Today's Follow-ups" Dashboard (Core Feature)

**🎯 Objective:** Build the dashboard that is the entire reason this tool exists: a view showing exactly which leads need a call today or are overdue.

**📖 What I'll learn:** Date comparison logic in JavaScript; designing the highest-priority screen of an app first.

**🛠 Features to build:** Dashboard view listing leads where `nextFollowUp <= today`, sorted soonest-first, with overdue leads visually flagged; make this the default/landing view of the app.

**📝 Step-by-step implementation plan:**
1. Write a `getTodayFollowUps(leads)` function: filters leads (excluding Admitted/Rejected) where `nextFollowUp` date is today or in the past.
2. Separate results into two groups: "Overdue" (date < today) and "Due Today" (date === today); sort each by date ascending.
3. Render the dashboard as the default view when the app loads (make it the first thing the admin sees).
4. Style overdue leads distinctly (e.g., red/amber accent) versus due-today leads.
5. Each dashboard entry shows: name, phone, current stage, next follow-up date, and a quick action to update the follow-up date (e.g., "mark done / reschedule") without leaving the dashboard.
6. Add an empty state ("No follow-ups due — you're all caught up") for when the list is empty.
7. Add a simple counter/badge (e.g., "3 overdue, 2 due today") at the top of the dashboard.

**📂 Files and folders to create or modify:**
```
app.js       (date logic, dashboard render function, reschedule action)
index.html   (dashboard container, make it default view on load)
style.css    (overdue vs due-today styling, empty state styling)
```

**🔗 APIs, libraries, services, or tools to integrate:** Native JS `Date` object only — no date library needed at this scale.

**🧪 Testing tasks:**
- Create leads with follow-up dates: yesterday, today, tomorrow, next week. Confirm only yesterday and today appear on the dashboard.
- Confirm yesterday's lead shows in the "Overdue" style and today's in the "Due Today" style.
- Reschedule a lead's follow-up date to next week from the dashboard; confirm it disappears from today's dashboard immediately.
- Confirm Admitted/Rejected leads never appear on the dashboard regardless of date.
- Test the empty state by rescheduling all leads into the future.

**🐞 Common issues and debugging tips:**
- Date comparison bugs (string vs Date object) → always convert `nextFollowUp` to a `Date` (or normalize to `YYYY-MM-DD` strings) before comparing; watch for timezone-related off-by-one-day errors — compare date-only, not date+time.
- Dashboard doesn't update after reschedule → confirm the reschedule action re-renders the dashboard, not just saves to storage silently.

**✅ End-of-day checklist:**
- [ ] Dashboard correctly filters to today/overdue leads only
- [ ] Overdue vs due-today are visually distinct
- [ ] Reschedule action works directly from the dashboard
- [ ] Dashboard is the default view on app load
- [ ] Empty state displays correctly when nothing is due

**📸 Expected project state and screenshots to capture:** Screenshot of the dashboard with a mix of overdue and due-today leads; screenshot of the empty state.

**➡️ Handoff notes for Day 6:** The core value feature (dashboard) is complete and is the app's landing view. This is the most important screen — prioritize protecting its correctness over new features going forward. Next day adds search/filter and polishes remaining functional gaps (FR-9, FR-10 from PRD).

---

## Day 6 — Search, Filter & Functional Polish

**🎯 Objective:** Round out remaining "Should" requirements and fix any functional gaps before visual polish begins.

**📖 What I'll learn:** Implementing simple client-side search/filter; doing a functional-requirements audit against a PRD.

**🛠 Features to build:** Search leads by name (from the All Leads list view); filter leads by stage; full pass against the PRD's functional requirements table.

**📝 Step-by-step implementation plan:**
1. Add a search input above the All Leads list; filter the rendered list in real time as the admin types (case-insensitive substring match on `name`).
2. Add a stage filter dropdown to the All Leads list (or funnel view) to show only one stage at a time, optional.
3. Re-open the PRD (Section 6, Functional Requirements) and check off each FR-1 through FR-10 against the actual working app. Fix any gaps found.
4. Fix any known bugs accumulated from Days 3–5 (keep a running bug list as you build each day; this is the day to clear it).
5. Confirm all data fields are editable and nothing is read-only that shouldn't be.

**📂 Files and folders to create or modify:**
```
app.js       (search/filter functions)
index.html   (search input, filter dropdown)
style.css    (search bar styling)
```

**🔗 APIs, libraries, services, or tools to integrate:** None new.

**🧪 Testing tasks:**
- Search a partial name, confirm matching leads filter correctly and non-matches disappear.
- Clear the search, confirm the full list returns.
- Go through the full PRD functional requirements table (FR-1 to FR-10) one by one, testing each in the live app.

**🐞 Common issues and debugging tips:**
- Search feels laggy → at <100 leads this shouldn't happen; if it does, confirm you're not re-reading from localStorage on every keystroke (filter the in-memory array instead).

**✅ End-of-day checklist:**
- [ ] Search by name works
- [ ] Optional stage filter works
- [ ] All FR-1 through FR-10 requirements verified working
- [ ] Known bug list from Days 3–5 is cleared

**📸 Expected project state and screenshots to capture:** Screenshot of search in action, showing filtered results.

**➡️ Handoff notes for Day 7:** All functional requirements from the PRD are implemented and verified. The app is feature-complete but visually basic. Next day is visual design and deployment — the app goes live.

---

## Day 7 — Visual Design Pass & Deployment

**🎯 Objective:** Apply the dark/premium visual style consistent with the existing app portfolio, and deploy the app to a live, public URL.

**📖 What I'll learn:** Applying a consistent design system with CSS; basic static-site deployment.

**🛠 Features to build:** No new features — visual polish and deployment only.

**📝 Step-by-step implementation plan:**
1. Define a small design system in CSS variables at the top of `style.css`: dark background (e.g., `#0f1420`), card background (e.g., `#1a2233`), accent/gold color (e.g., `#b8935a`), text colors, spacing scale, and a consistent border-radius.
2. Apply the design system across: dashboard, funnel view, all-leads list, and forms — consistency matters more than any single flourish.
3. Add clear visual hierarchy: dashboard as the primary/landing screen, prominent counts/badges, legible typography (system font stack is fine — no external font loading needed).
4. Add subtle empty/loading states and hover states for buttons and cards.
5. Do a full pass on mobile width (admin may check the dashboard on a phone) — confirm no horizontal overflow, tap targets are large enough.
6. Deploy: use a free static hosting option appropriate for a single HTML/CSS/JS app (e.g., GitHub Pages, since the repo is already on GitHub from Day 2). I will walk you through this step by step when you're ready — this is a manual step outside this chat.
7. Confirm the live deployed URL works end-to-end: add a lead, refresh, confirm dashboard reflects it.

**📂 Files and folders to create or modify:**
```
style.css    (design system variables, full visual pass)
index.html   (any markup tweaks needed for new styles)
README.md    (add live deployed link once available)
```

**🔗 APIs, libraries, services, or tools to integrate:** Free static hosting (e.g., GitHub Pages) — no paid services.

**🧪 Testing tasks:**
- Full click-through of every feature on the live deployed URL (not just localhost).
- Test on a real phone browser if possible, not just desktop resize.
- Confirm localStorage works correctly on the deployed domain (it's per-domain, so this is a fresh empty state on first visit — expected and fine).

**🐞 Common issues and debugging tips:**
- Deployed site shows blank page → check browser console for path errors (relative paths to CSS/JS can break depending on hosting config).
- Styles look different live vs local → hard-refresh (cache) before concluding something is broken.

**✅ End-of-day checklist:**
- [ ] Consistent dark/premium design system applied across all views
- [ ] Mobile layout confirmed working
- [ ] App deployed to a live public URL
- [ ] Full feature walkthrough verified on the live URL
- [ ] README updated with the live link

**📸 Expected project state and screenshots to capture:** Screenshots of the dashboard, funnel view, and all-leads view on the live deployed site; screenshot of the site open on a phone.

**➡️ Handoff notes for Day 8:** App is feature-complete, styled, and live. From here on, work is testing, hardening, and demo prep — no new features. Next day is dedicated testing with realistic sample data.

---

## Day 8 — Testing with Realistic Sample Data & Bug Hardening

**🎯 Objective:** Stress-test the live app with realistic sample data and fix any remaining bugs.

**📖 What I'll learn:** Manual QA methodology; thinking like a real end user instead of a developer.

**🛠 Features to build:** None — this is a testing and bug-fixing day. (Optional stretch: a "Load Sample Data" dev button, removed or hidden before final handoff.)

**📝 Step-by-step implementation plan:**
1. Create 15–20 realistic sample leads by hand on the live site, covering: a range of grades, follow-up dates spread across past/today/future, and leads at every funnel stage including some Admitted and Rejected.
2. Walk through every user flow end-to-end as if you were the actual school admin: check dashboard, call a "lead," update notes, move stage, reschedule, search for a name, delete a stale lead.
3. Test edge cases: empty name/phone submission, very long notes text, duplicate names, a lead with no follow-up date set, rapid repeated clicks on buttons.
4. Test browser refresh and closing/reopening the browser tab to confirm persistence holds.
5. Log every bug found in a simple running list (in README or a scratch file) with severity (blocker / minor).
6. Fix all blocker bugs today. Minor cosmetic bugs can roll to Day 9 if time is tight.

**📂 Files and folders to create or modify:**
```
app.js, index.html, style.css   (bug fixes as needed)
README.md                        (bug list / testing notes, optional)
```

**🔗 APIs, libraries, services, or tools to integrate:** None new.

**🧪 Testing tasks:** This entire day is testing — see implementation plan above.

**🐞 Common issues and debugging tips:**
- If bugs pile up faster than expected, prioritize anything that breaks the dashboard or data persistence first — those are the app's core promise.
- Resist the urge to add features while testing ("it would be nice if...") — capture these in the Future Scope list instead, don't build them now.

**✅ End-of-day checklist:**
- [ ] 15–20 realistic sample leads loaded on the live site
- [ ] Full user-flow walkthrough completed with no blocker bugs remaining
- [ ] Edge cases tested
- [ ] Persistence confirmed across refresh/reopen
- [ ] Bug list documented, blockers fixed

**📸 Expected project state and screenshots to capture:** Screenshot of the dashboard populated with realistic sample data, looking like a real school's admissions pipeline.

**➡️ Handoff notes for Day 9:** App is tested and stable with realistic data loaded. No known blocker bugs remain. Next day is final polish and a full README/documentation pass.

---

## Day 9 — Final Polish & Documentation

**🎯 Objective:** Final visual/UX polish pass and complete project documentation.

**📖 What I'll learn:** Writing a good README; the discipline of "polish, don't add" in the final stretch.

**🛠 Features to build:** None — polish only. Fix any remaining minor bugs from Day 8's list.

**📝 Step-by-step implementation plan:**
1. Fix remaining minor/cosmetic bugs from the Day 8 list.
2. Do a final visual consistency pass: spacing, alignment, button states, color contrast — compare dashboard, funnel, and list views side by side for consistency.
3. Write a complete `README.md`: project name, one-paragraph description, the live deployed link, feature list, how to use it, tech stack used, and a note on the local-storage limitation (data is per-browser/device).
4. Add a short "About this project" section to the README noting it was built as part of the AB Talks 60-Day Claude AI Challenge (#60DaysClaudeChallenge), consistent with the rest of the portfolio.
5. Clean up the codebase: remove any leftover debug `console.log` statements, commented-out dead code, or test data left over from Day 8 (decide whether to ship with the realistic sample data pre-loaded, or start empty for a new user — recommend shipping empty, with sample data only in a screenshot/demo).
6. Final commit and push to GitHub.

**📂 Files and folders to create or modify:**
```
README.md    (final, complete version)
app.js       (cleanup: remove debug code, dead code)
```

**🔗 APIs, libraries, services, or tools to integrate:** None new.

**🧪 Testing tasks:** One more full click-through on the live URL after cleanup, to confirm nothing broke during the polish pass.

**🐞 Common issues and debugging tips:**
- Removing "sample data" logic accidentally breaks something else → re-test the empty state (Day 5's empty-state dashboard) after this cleanup.

**✅ End-of-day checklist:**
- [ ] All minor bugs from Day 8 resolved
- [ ] Visual consistency pass complete
- [ ] README.md complete with live link, features, tech stack, limitations
- [ ] Codebase cleaned of debug/dead code
- [ ] Final commit pushed to GitHub

**📸 Expected project state and screenshots to capture:** Screenshot of the finished README on GitHub; screenshots of the final polished app for use in the pitch deck / LinkedIn post.

**➡️ Handoff notes for Day 10:** App is fully polished, documented, and live. Nothing left to build. Day 10 is demo prep, final QA, and public sharing.

---

## Day 10 — Final QA, Demo Prep & Launch

**🎯 Objective:** Final end-to-end QA of the live product, prepare the demo/pitch materials, and publicly ship the project.

**📖 What I'll learn:** How to present a finished project publicly; final release discipline.

**🛠 Features to build:** None. This is launch day.

**📝 Step-by-step implementation plan:**
1. Do one final full walkthrough of the live deployed app exactly as a first-time user (a school admin) would experience it.
2. Confirm the PRD's Definition of Done (Section 9) is fully met — check every item.
3. Prepare the pitch deck talking points (deck itself was generated on Day 1 — review and update any details that changed during the build, e.g. the final live link).
4. Take final polished screenshots/short screen-recording of the dashboard, funnel view, and adding a lead, for the LinkedIn post.
5. Write the LinkedIn post for #60DaysClaudeChallenge: what was built, the problem it solves, and the live link.
6. Publish: push final code to GitHub, confirm the live link is stable, post to LinkedIn.
7. Do a short retrospective note in the README or a `RETROSPECTIVE.md`: what went well, what you'd improve, what's next (from the Future Scope list).

**📂 Files and folders to create or modify:**
```
README.md          (final link check)
RETROSPECTIVE.md   (optional but recommended for the challenge)
```

**🔗 APIs, libraries, services, or tools to integrate:** LinkedIn (for sharing only, no integration).

**🧪 Testing tasks:** Final full regression walkthrough of every feature in the PRD, on the live URL, on both desktop and mobile.

**🐞 Common issues and debugging tips:**
- If anything breaks on this final check, fix only the specific bug — do not refactor or add features on launch day.

**✅ End-of-day checklist:**
- [ ] Every Definition of Done item from the PRD is confirmed met
- [ ] Final walkthrough completed with zero blocker bugs
- [ ] Screenshots/recording captured
- [ ] LinkedIn post published with live link
- [ ] Final code pushed to GitHub
- [ ] Retrospective notes written

**📸 Expected project state and screenshots to capture:** Full set of final app screenshots (dashboard, funnel, all-leads, add-lead form) for the portfolio and LinkedIn post.

**➡️ Project complete.** v1.0 of the Admissions Lead Tracker is live, polished, tested, and shared — the tenth app in the single-file/lightweight app portfolio and the flagship deliverable of this capstone.
