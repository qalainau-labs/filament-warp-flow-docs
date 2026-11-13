# Changelog

All notable changes to `qalainau/filament-warp-flow` will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.1.1] - 2026-11-13

### Security

- The request page, its thread and a record's thread check on every request that the person may see them, not only when the page opens. A Livewire request after the first one does not run `mount()`, so someone who got hold of another person's page state could see a request they may not see.

## [1.1.0] - 2026-11-12

### Added

- Dashboard widgets: requests waiting for you, overdue tasks, and your own requests that were returned. Turn them on with `->dashboardWidgets()`. The inbox and *My requests* filters are now kept in the URL, so a widget can link to a filtered list.
- Reminders before a task is due, repeated reminders after it is overdue, and an automatic decision when it is overdue. A step's `remind_before`, `remind_every` and `overdue_action` (`remind`, `escalate`, `approve` or `reject`) are set in the flow editor. An automatic decision is marked as such on the request and in the history. Runs from the hourly `warp-flow:check-due`.
- Flow templates to start from: manager approval, manager then their manager, managers in turn, branch by amount, and purchasing. Shown on the create page and in an empty flow editor. Add your own with `->flowTemplate()`, hide the built-in ones with `->builtInFlowTemplates(false)`.
- Request numbers: `->requestNumber('REQ-{Y}-{seq:5}')`, with the period the sequence restarts in and an optional sequence per flow. Numbers are counted per tenant and never repeat, even when two people submit at the same moment.
- An *All requests* page for auditors and accounting, with filters and a CSV export (values that a spreadsheet would run as a formula are made harmless). Only for the people `canViewAllRequests()` allows, per tenant. Remove it with `->allRequests(false)`.
- Adding an approver part-way (“have this person look at it too”) with `AddApproverAction` and `->addApprovers()`, and reassigning a waiting step to someone else with `ReassignAction` and `->canReassign()`. Permissions and the tenant are checked again when the action runs.
- Line items in request forms: a `table` field with text, number, date and select columns, and a total (the sum of one column, or quantity × unit price). The total is calculated on the server on every save and is available to flow conditions as `data.<key>_total`. The request page and the print view show the table.

### Changed

- Now requires Filament 5.8.3 or later (was 5.0). Older 5.x versions have known security advisories (Composer 2.9 and later refuses to install them), and the plugin cannot be tested on them. The tests now also run on the lowest supported versions before each release.
- The request page explains what you can do when it is not your turn, and the amount column of line items no longer looks editable.
- Clearer wording in all languages, using the same names as the screens (approval route, conversation, flow, step).

### Fixed

- On PostgreSQL, a person key that is not a number (or not a UUID, for UUID keys) sent from the browser ended in a database error instead of “That person cannot be chosen here.” Such keys are now dropped before the users are looked up.
- Submitting no longer loads each flow's published version one by one, which failed in apps that turn on `Model::preventLazyLoading()` when there were two or more flows.
- On MariaDB 11 with Filament tenancy, loading the published version of several flows at once returned none: the flow list showed every flow as unpublished. The published version is now loaded in a way that works on every database.

### Upgrading

- Run `php artisan migrate`. The new columns and the counters table are in a new migration (`extend_warp_flow_tables_with_numbers_and_reminders`); the 1.0.0 migration is unchanged. If you published the 1.0.0 migrations, the new one still runs from the package.

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
- Reminders and automatic handling of overdue steps. A step can remind its approver a number of hours **before the due time**, **repeat the reminder** every N hours after it, and choose what happens **when the due time passes**: only remind (the default, as before), pass it to the manager, **approve it automatically** or **reject it automatically**. An automatic decision is recorded as automatic — no `acted_by`, `is_automatic` on the task, `auto_approved` / `auto_rejected` in the audit log, “Automatic” on the stamp — and each task is handled once, however often `warp-flow:check-due` runs. New notifications `due_soon`, `auto_approved` and `auto_rejected`. New columns `due_soon_at`, `reminded_at`, `reminders` and `is_automatic` on `warp_flow_tasks`.
- Flow templates: a new flow can start from *Manager approval*, *Manager, then the manager’s manager*, *Every manager up to the top*, *Branch on the amount* or *Purchasing* instead of empty — chosen under **Start from** when creating a flow, or with **Start from a template** in the editor of an empty flow. The templates that branch on an amount are offered when the records have a number field. `flowTemplate()` adds your own; `builtInFlowTemplates(false)` hides the bundled ones.
- Request numbers (`requestNumber()`): every new request gets a number such as `REQ-2026-00042` from a format of your choice (`{Y}`, `{y}`, `{m}`, `{d}`, `{seq}`, `{seq:5}`, `{flow}`, or a closure), starting again each year, month or day, per tenant and optionally per flow. No number is given twice, even when two people submit at once. Shown on the request, in the lists and on the printout; the lists find a request by it. New column `number` on `warp_flow_instances` and table `warp_flow_counters`.
- All requests: a page for auditors and accounting that lists every request with filters (status, flow, applicant, department, day of submission, overdue now) and exports the listed requests as CSV (UTF-8 for Excel, formulas made harmless). Only for the people `canViewAllRequests()` allows — managing flows is not enough — and per tenant. `allRequests(false)` hides it.
- Add an approver: the approver whose turn it is can add a person to their step from the request page (“have this person look at it too”). The step keeps its rule, so with *Everyone* the added person has to approve as well. `AddApproverAction`, `addApprovers()` to switch it off or decide per person, notification `approver_added`. New column `added_by` on `warp_flow_tasks`.
- Reassign a stuck step: a flow administrator hands the step of someone who cannot answer to another person, with a required reason, a new due time and a line in the history. `ReassignAction`, `canReassign()` (default: the people `canManage()` allows), notification `reassigned` to the person the step was taken from. Neither action accepts the applicant, a person outside the tenant or a task of another request.
- Request forms: a **Table of rows** field for line items — columns of text, number, date or choice, rows added on the request — with a total (one number column added up, or quantity × unit price per row). The total is a number field of the form (`data.<key>_total`), so a flow can branch on it (“total ≥ 100,000 → the director approves too”); it is worked out on the server at every save. Approvers see the rows and the total on the request and the printout, and what changed in them after a return.
- Dashboard widgets (`dashboardWidgets()`): `ApprovalStats` shows what waits for your approval, how much of it is overdue and how many of your own requests were returned, each number linking to its list; `WaitingApprovals` lists the five requests waiting for you with the nearest due time; `ReturnedRequests` lists your returned requests with who returned them and why. They count the signed-in person’s own work only, per tenant. The inbox and *My requests* keep their filters in the URL.
