MATERNAL CARE TASK FORCE — MEMBER SITE
Deployment guide (Google Sheets + Apps Script + Google Sites)
Prepared 2026-09-03

WHAT THIS IS
A read-only member reference site for the Cecil + Harford Maternal Care
Task Force: purpose, problems, structure, funding, resources, data, meeting
summaries, action items, and a de-identified access-barrier tally. Content
lives in one admin-owned Google Sheet. An Apps Script web app renders it and
is embedded in a Google Site. All submissions come in through Google Forms.

FILES (all delivered as .txt; strip the .txt when creating each file)
  apps-script/appsscript.json.txt   -> appsscript.json (manifest; enable
                                       "Show appsscript.json" in Project Settings)
  apps-script/Config.gs.txt         -> Config.gs
  apps-script/DataAccess.gs.txt     -> DataAccess.gs
  apps-script/Audit.gs.txt          -> Audit.gs
  apps-script/Auth.gs.txt           -> Auth.gs
  apps-script/Api.gs.txt            -> Api.gs
  apps-script/Setup.gs.txt          -> Setup.gs
  apps-script/Seed.gs.txt           -> Seed.gs
  apps-script/SeedResources.gs.txt  -> SeedResources.gs
  apps-script/Remediate.gs.txt      -> Remediate.gs
  apps-script/SeedEvidence.gs.txt   -> SeedEvidence.gs
  apps-script/Index.html.txt        -> Index.html
  apps-script/Styles.html.txt       -> Styles.html
  apps-script/Client.html.txt       -> Client.html

HOSTING AND ACCESS DECISIONS (2026-09-03)
  - Hosted in the Performance Health Group Google Workspace during
    development and deployment. Owner: admin@performancehealthgroup.org.
  - Web app access: "Anyone" (no sign-in required). The deployment URL is
    therefore PUBLIC. Every row marked Published must be safe for public
    view; the Published gate is the only content control.
  - Admin list defaults to the account that runs runSetup.
  - See HANDOFF at the end for moving the project to a county account.

STEP 1 — Create the Apps Script project
  1. Signed in as admin@performancehealthgroup.org, go to
     script.google.com > New project. Name it "MCTF Member Site".
  2. Project Settings > check "Show appsscript.json manifest file in editor".
  3. Create each file listed above (File > New > Script or HTML) and paste
     the contents. File names must match exactly (Index, Styles, Client are
     HTML files). When Apps Script asks for the name, type it WITHOUT the
     extension: "Config", not "Config.gs".
  4. Run verifyProject in Setup.gs. It prints an OK/MISSING checklist for
     all 12 files. Do not continue until every line says OK.

     Why this matters: all .gs files share one global scope, so a missing
     file does not report itself. It surfaces later as a ReferenceError such
     as "getAdminEmails_ is not defined" (that symbol lives in Config.gs)
     or "CONFIG is not defined". If you see one, the named symbol's file is
     missing, empty, or misnamed.

STEP 2 — Provision the spreadsheets and forms
  0. Run verifyProject in Setup.gs first (see Step 1.4).
  1. In the editor, open Setup.gs, select runSetup, click Run. Approve the
     OAuth scopes (Sheets, Forms, Drive, email).
     - Creates "MCTF — Site Content (ADMIN ONLY)" with every tab and header.
     - Creates "MCTF — Audit Log (RESTRICTED)" as a SEPARATE file.
     - Creates two Google Forms (Barrier Report, Resource Submission) that
       collect verified email and write into the content sheet.
     - Stores all IDs in Script Properties.
  2. Read the execution log for the two spreadsheet URLs.
     Expected log lines: "AUDIT_DEFERRED action=SETUP" followed by
     "Replayed 1 deferred audit row(s) from bootstrap." That pair is
     normal on a first run — the admin check runs before the audit
     spreadsheet exists, so the entry is buffered and replayed.
     If instead you see "AUDIT_WRITE_FAILED action=SETUP" (older code),
     run backfillSetupAudit('<UTC time from the log>') in Setup.gs once.
  3. Move the Audit Log spreadsheet into a Drive folder that only the
     admin/Privacy Officer can open. Do not share it with anyone else.
  4. Do NOT share the content spreadsheet with task-force members. Admins
     who maintain content get Editor access; nobody else.
  5. runSetup sets the admin list to the account that ran it
     (admin@performancehealthgroup.org). To add county admins later, run
     setAdminEmails in Setup.gs with the FULL list, e.g.
        setAdminEmails('admin@performancehealthgroup.org,healthofficer@cecilcountyhealth.org')
     (Run it from the editor by temporarily adding a wrapper:
        function setAdminsNow(){ setAdminEmails('a@x.org,b@y.org'); }
      then delete the wrapper.)

STEP 3 — Seed starting content
  1. Open Seed.gs, select seedAll, Run. This fills Purpose, Problems,
     Structure, Members, Funding, Data, Meetings, ActionItems, Announcements,
     and Resources (from SeedResources.gs). Seeding skips any tab that
     already has rows.
  2. Open the content sheet and review every row. Set Published to FALSE on
     anything not ready. Member contact columns are blank by default; only
     fill Email/Phone AND set ShareContact = TRUE after the member opts in.
  3. In the Config tab, set TAGLINE, CONTACT_EMAIL, NEXT_MEETING as desired.

STEP 3.5 — Pre-launch content review (REQUIRED)
  1. Run previewPublicSurface in Remediate.gs. It prints exactly what an
     anonymous visitor would see: row counts per tab, every action item with
     its owner, any member whose contact details are published, and how many
     resource rows still claim High confidence.
  2. Read every action item and meeting summary yourself, looking for anything
     that describes ONE person's situation. On a public URL, "case", "the
     patient", or a single-site organization plus a month is enough to
     re-identify someone in a two-county region.
  3. Have each named member confirm their own row.
  4. See docs/AUDIT_2026-09-04.txt for the findings this step exists to catch.

STEP 4 — Deploy the web app
  1. Deploy > New deployment > type: Web app.
     Execute as: Me.
     Who has access: Anyone.
     (The manifest already declares ANYONE_ANONYMOUS. This makes the web
     app URL public — no sign-in — so the embed loads for every member
     regardless of which identity they use.)
  2. Copy the Web app URL. Open it once in a browser to confirm it renders.
  3. Run smokeTest in Api.gs; the log prints published row counts per tab.

STEP 5 — Google Site
  1. sites.google.com > create the site (e.g. "Maternal Care Task Force").
  2. Insert > Embed > By URL > paste the web app URL > Insert. Resize the
     embed to full width; set a height of at least 1400 px (the embed does
     not auto-grow). Alternatively give each section its own Sites page and
     embed the same URL with #problems, #structure, #resources, etc.
  3. Sites sharing: Publish to "Specific people" and add task-force members
     by email. This controls who FINDS the page. It does not restrict the
     web app URL, so never publish a row that must not be public.
  4. Publish.

UPGRADING TO v1.4.3
  Replace Client.html, Styles.html, Config.gs and Remediate.gs, then create a
  new deployment version.

  Then run fixHeaderAndWelcome in Remediate.gs ONCE. Two corrections live in
  the sheet rather than in code, so replacing files does not apply them:
   - The header still reads "Maternal Care Task Force - Cecil & Harford
     Counties" because runSetup's upsertConfig_ never overwrites a Config
     value that is already set. This rewrites SITE_TITLE and SUBTITLE.
   - The "Welcome to the member site" announcement was removed from the seed
     file in v1.2.0, but the row already existed in the sheet. This clears
     the Announcements tab and rebuilds it from the current seed.

  Presentation changes in this version:
   - The trend arrow no longer wraps below its figure.
   - The six indicator tiles are normalised: the unit moved from the
     indicator name to the benchmark line ("Maryland 5.5 per 1,000"), the
     name block reserves two lines, and the benchmark is pinned to the
     bottom of each tile, so all six are the same height with their value
     rows and rules aligned.

UPGRADING TO v1.4.2 (presentation only)
  Replace Client.html, Styles.html and Config.gs; new deployment version.
   - County figures read clearly red when worse than Maryland and green when
     better, on the header tiles and in the Regional Data table. Maryland is
     the benchmark and is never coloured.
   - One comparison function now drives both surfaces, so the tiles and the
     table can never disagree. It inverts correctly for early prenatal care,
     where a higher figure is better.
   - Each figure also carries a small arrow showing whether it sits above or
     below the state, plus hidden text reading, for example, "above Maryland,
     worse" - so the meaning survives for a colourblind reader.

UPGRADING TO v1.4.1 (presentation only)
  Replace Client.html, Styles.html and Config.gs; new deployment version.
   - Every figure on the header tiles is now labelled with the place it
     describes. Each tile carries both counties (Cecil and Harford, each
     coloured against the state) with Maryland stated underneath as the
     benchmark, instead of one unlabelled number.
   - Six tiles fit on one row at the full content width, and collapse to
     one labelled row per indicator on a phone.

UPGRADING TO v1.4.0
  Replace Api.gs, Config.gs, Setup.gs, Client.html and Styles.html, run
  runSetup in Setup.gs (it adds the new Config key without touching rows),
  then create a new deployment version. No re-seed.

  What changed:
   - Barrier Log lists every report, each expandable to its full detail:
     month, county, barrier category, service type, reporting organisation
     and the written description, with filters and Expand all.
   - Each report is labelled Reviewed or Not yet reviewed. Publishing no
     longer waits on review; see COMPLIANCE_NOTES.txt section 7 and the new
     Config key BARRIER_DETAIL_VISIBILITY if the co-chairs want the gate back.
   - Free text passes through an identifier scrub (emails, phone numbers,
     exact dates, long digit strings) before it is served.
   - Every page gains an action bar: quick actions where they apply (Report
     a barrier, Suggest a resource) and On this page jump links that scroll
     to a section, allowing for the sticky tab bar.
   - Fixed a horizontal-overflow bug on phones: filter menus sized
     themselves to their longest option and pushed the page to 525px wide.

UPGRADING TO v1.3.1 (presentation only)
  Replace Styles.html, Client.html and Config.gs; create a new deployment
  version. No sheet change, no re-seed.
   - A newline in a sheet cell no longer becomes a forced mid-sentence line
     break. It starts a new paragraph instead.
   - Meeting Decisions and Next steps render as bulleted lists, one item per
     line, rather than run-on text split by hard breaks.
   - Paragraphs use balanced wrapping so a sentence no longer ends with a
     single word stranded on its own line.
   - Prose is held to about 72 characters per line; summary lines clamp to
     two lines instead of running the full card width.

UPGRADING TO v1.3.0 (presentation only)
  Replace Styles.html, Client.html and Config.gs, then create a new
  deployment version. No sheet change and no re-seed: v1.3.0 changes only
  how the content is presented.

  What changed:
   - Every long page collapses. Sections, problems, teams, funding,
     resources and drivers open on demand, with Expand all / Collapse all
     on each list. The evidence page is about five times shorter closed.
   - Resources are grouped into collapsible categories; a filtered result
     opens its groups automatically.
   - Palette moved from bright purple to a deep navy with a single teal
     accent. Red and green now appear only where a county is compared with
     the state average, so colour always carries meaning.
   - Text selection, focus rings, scrollbars and tabular figures are themed
     rather than left at browser defaults.

UPGRADING AN EXISTING DEPLOYMENT TO v1.2.0
  (Adds the Drivers and CaseStudies tabs, indicator tags on every content
  row, the corrected header, and removes the member-site welcome
  announcement and the public-facing emergency paragraph.)
  1. Replace the changed files (see the release note).
  2. Run runSetup in Setup.gs. It is safe on an existing sheet: it adds the
     new columns (TargetIndicators, Indicators) to the end of existing tabs
     and creates the Drivers and CaseStudies tabs. No rows are touched.
  3. Run remediateV1_2 in Remediate.gs. It corrects the header title and
     subtitle, then clears and rebuilds the tagged content tabs and seeds
     Drivers and CaseStudies. Members, Data, and Meetings are not touched.
  4. Deploy > Manage deployments > Edit > Version: New.

UPDATING CONTENT
  - Edit rows in the content sheet. Changes appear within 5 minutes
    (cache), or immediately after running purgeCache in Api.gs.
  - Never delete a row to remove content; set Published = FALSE. (Rows in
    Meetings/ActionItems are a record of the Task Force's work.)
  - New form submissions land in BarrierLog / ResourceSubmissions with
    Review status blank. A co-chair reviews for identifying details, fills
    Reviewed by, and sets Published = TRUE only if it is safe to show.
    Aggregate counts are shown regardless; free text only when Published.
  - To promote a ResourceSubmission into the directory, copy its fields to
    a new Resources row and set Published = TRUE.

EMBED TROUBLESHOOTING
  - Blank iframe in Sites: confirm the active deployment is set to
    "Anyone" and that a NEW version was created after the last code edit.
  - "Content refreshed" footer shows an old time: run purgeCache.
  - After editing code, create a NEW deployment version (Deploy > Manage
    deployments > Edit > Version: New) or the site keeps the old code.

HANDOFF TO A COUNTY ACCOUNT (when the Task Force is ready)
  1. Drive: transfer ownership of the content spreadsheet, the audit
     spreadsheet, both Forms, and the Apps Script project to the county
     admin account (Share > transfer ownership). Script Properties travel
     with the project.
  2. Apps Script: the new owner must create a NEW deployment, because
     "execute as me" binds to whoever deploys. Update the Sites embed with
     the new URL and archive the old deployment.
  3. Forms: confirm each form's response destination still points at the
     content spreadsheet after transfer.
  4. Run setAdminEmails with the county list; remove the PHG address once
     the county confirms access.
  5. Google Site: transfer ownership or rebuild under the county account.
  6. Record the handoff date in COMPLIANCE_NOTES.txt.

WHERE EACH RUNNABLE FUNCTION LIVES
  verifyProject ....... Setup.gs      (run this first)
  previewPublicSurface  Remediate.gs (what an anonymous visitor would see)
  remediateV1_1 ....... Remediate.gs (applies audit fixes to an already-seeded sheet)
  remediateV1_2 ....... Remediate.gs (indicator tags, Drivers/CaseStudies, header; run runSetup first)
  setConfigValue ...... Remediate.gs
  reseedTabs .......... Remediate.gs
  runSetup ............ Setup.gs
  backfillSetupAudit .. Setup.gs   (only if setup logged AUDIT_WRITE_FAILED)
  fixHeaderAndWelcome . Remediate.gs (corrects the header text and removes the
                                    welcome announcement on an existing sheet)
  setAdminEmails ...... Setup.gs
  seedAll ............. Seed.gs      (calls seedResources in SeedResources.gs)
  purgeCache .......... Api.gs
  smokeTest ........... Api.gs
