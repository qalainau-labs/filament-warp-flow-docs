# Changelog

All notable changes to `qalainau/filament-warp-flow` will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2026-11-11

### Added

- First release: approval workflows for Filament 5 that know your organization.
- Organization: departments in a tree, positions with a rank, members with concurrent posts and dated transfers, and a manager override. `OrgDirectory` to use an app's own organization data instead.
- Flows drawn in an editor: steps from top to bottom, conditions with branches side by side, undo and redo, drafts saved as you go, versions (running requests keep theirs), and a route simulator drawn on the flow.
- Approvers relative to the applicant (manager, N levels up, managers in turn up to a position, position holders, department heads), specific people, a user stored on the record, and custom resolvers. Resolved at submission.
- Conditions on the fields a model declares with `ConditionField`, and on the applicant's department and rank. Flows chosen by model (named after the panel’s resource for it), priority and conditions, or by key.
- Step types: approval, consultation, circulation and post-review; everyone, any one or N people per step; skip the applicant and people who already approved.
- Return to the applicant or to any earlier step, resubmit (resume or start over), reject, withdraw, re-resolve the route after a transfer.
- Due times with reminders and hand-over to the manager (`warp-flow:check-due`, scheduled hourly), delegation, an append-only audit log, events.
- Approval inbox with unread and bulk approve, My requests, Delegations, and a page per request with hanko-style stamps, the route and a thread with @mentions and attachments.
- Record actions (`ApprovalActions`; *Submit for approval* is offered for a new, returned, rejected or withdrawn record, not an approved one), `ApprovalStamps` and `ApprovalTimeline` schema components, `ApprovalStatusColumn` and `ApprovalStatusFilter`.
- Database and mail notifications, `notifyUsing()` for other channels. Dark mode.
- Multi-tenancy with Filament’s tenancy (tenant by id or slug): everything is kept per tenant, including the inbox of people in several tenants, user pickers and @mentions (`canAccessTenant()`, or `scopeUsersUsing()` for one query), flow keys, and the scheduled due-time check. `forTenant()` for jobs and seeders, `tenantKeyUsing()` for your own tenancy. A panel without tenancy in the same app sees only rows without a tenant. The tenant is stored as `tenant_type` and `tenant_id`, so panels with different tenant models (`Team` and `Company`) never mix rows with the same ID; flow keys are unique per tenant in the database. People outside the tenant returned by `approverResolver()` or a record’s user field are not given tasks. A tenant panel shows nothing before its tenant is known.
- Record threads: `HasThread` and `RecordThread::make()` put a discussion on any record, with or without an approval — comments with @mentions and attachments, followers who are notified of new comments, and the record's requests as they happen (submitted, returned, rejected, withdrawn, approved). A *Threads* page (`threads()`) lists the threads you follow with the unread ones counted. `canViewThread()` decides who can read them (by default the model's policy). Notifications `thread_mentioned` and `thread_commented`; `FlowMessage::$thread`.
- Foreign keys between the `warp_flow_*` tables (deleting a request or thread removes what belongs to it; a flow with requests cannot be deleted) and tenant indexes that include the column each list filters or sorts by.
- Plugin options are kept per panel (a panel without tenancy and a team panel can have different `canManage()`, resolvers and notifications).
- Changes since the last submission: when a returned request is resubmitted (or a rejected record is submitted again), the request page marks the fields that changed with their old values, the timeline lists them, and the approvers’ notification says what changed. `approvalSnapshot()` on the model decides what is compared.
- Due times in business days: a step can be due within N business days instead of calendar hours. `businessCalendar()` sets the working days, Japanese holidays (`japaneseHolidays()`, from the Cabinet Office list, with the holiday law for later years), your own days off and the time zone.
- Out of office: people (or managers for them) enter absences on a new *Out of office* page, and `isAbsentUsing()` reads them from your own data (leave, shifts). An approval or consultation that reaches someone away goes to their manager — or further up if the manager is away too — unless they have a delegate. Requests already waiting are passed on by the hourly `warp-flow:check-due`. New table `warp_flow_absences`.
- Answer from the email (`emailApproval()`): the email that brings a task carries a signed link, for that person only and valid for 7 days by default, to a page where they approve or return (agree or disagree, mark as read) without signing in. Opening the link does nothing by itself; delegates get their own link.
- Print / PDF: *Print / PDF* on the request page opens an A4 approval record (a ringi-sho) with the hanko stamps, the summary and its changes, every person’s result and comment, and the history. Save it as a PDF from the browser’s print dialog — no PDF library on the server.
- Org chart: a new *Org chart* page draws the departments top-down with their heads and members, concurrent posts, manager overrides and dated transfers, as of any day. Find a person by name. Managers see it by default; `canViewOrgChart()` opens it to others.
- Organization import: *Import* on the members page reads a CSV file or rows pasted from Excel (UTF-8 or Shift_JIS, English or Japanese headers) with one person and department per row. Missing departments and positions are created and existing memberships updated, so the same file can be imported again; nothing is written if any row has a problem. `importUserColumn()` matches people on another column (an employee code).
- Approval analytics: a new page for managers and auditors with the requests of the last 30, 90 or 365 days (per flow or all), the approval rate and median time to approval, the steps where requests wait longest, who they wait on now (with overdue counts) and the oldest open requests. `analytics(false)` hides it.
- Request forms (`forms()`): admins build forms — expense claims, leave, purchases — from fields (text, number, date, choice, checkbox, person, files) under *Request forms*, and everyone files requests with them under *Requests*, without a model in the app. A form is a flow target like a model: its number, choice, text, checkbox and person fields work in conditions, the simulator and as approvers. *Create its flow* starts one. New tables `warp_flow_forms` and `warp_flow_form_entries`.
- Translations: English, Japanese, German, French, Spanish, Italian, Brazilian Portuguese, Korean and Chinese (simplified and traditional).
- Hanko-style stamps with a key under them (whose turn it is, overdue, later, stamped; red for approved or agreed, black for rejected, orange for returned), smaller on phones.
- An approval step whose approvers resolve to nobody stops the submission with an error by default (**If nobody is found**); a step skipped on purpose is marked with a warning on the request page, in the route, on the timeline and in the route simulator.
- `navigationGroup()`, `settingsNavigationGroup()` and the label of `approverResolver()` accept a closure, called when the label is shown, so it follows the user's language.
