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
     HTML files).

STEP 2 — Provision the spreadsheets and forms
  1. In the editor, open Setup.gs, select runSetup, click Run. Approve the
     OAuth scopes (Sheets, Forms, Drive, email).
     - Creates "MCTF — Site Content (ADMIN ONLY)" with every tab and header.
     - Creates "MCTF — Audit Log (RESTRICTED)" as a SEPARATE file.
     - Creates two Google Forms (Barrier Report, Resource Submission) that
       collect verified email and write into the content sheet.
     - Stores all IDs in Script Properties.
  2. Read the execution log for the two spreadsheet URLs.
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
  runSetup ............ Setup.gs
  setAdminEmails ...... Setup.gs
  seedAll ............. Seed.gs      (calls seedResources in SeedResources.gs)
  purgeCache .......... Api.gs
  smokeTest ........... Api.gs
