# Jira–Xray Sync-Up Interface

**Automating Test Execution Management & Real-Time CI/CD Result Sync**

*Ashok Kakade | Principal QA Automation Engineer |QA Automation Architect | NGDATA*

---

## 1. Problem Statement

At NGDATA, setting up test executions in Jira-Xray for release regression cycles across multiple projects (LP, SC, SI) was entirely manual. There was no automated bridge connecting CI/CD execution results to Xray test management beyond Bamboo's native result sync — everything upstream (creating executions, associating tests, keeping fields consistent) and everything downstream (failure triage and re-runs) required manual effort.

## 2. Constraints

- Manually creating and configuring ~100 test executions per release, with correct test associations, took a Manual Lead roughly **2 days** — often longer, since human error in test-to-execution associations meant rework.
- Each execution required consistently updating multiple interdependent fields (Summary, Description, test associations, Release, Assignee, Time Estimation, Timesheet Account, Test Plan ID) — high-friction at volume, and easy to get wrong.
- Bamboo already synced CI/CD results into test executions automatically, but nothing existed to help Test Engineers act on those results — they still manually identified failed tests and re-triggered them individually or in batches.
- Needed to work consistently across three active Jira/Xray Cloud projects without turning into three separate tools.

## 3. Approach

Instead of scripting a one-off fix for execution creation, the tool was designed from the outset as a CLI-driven interface with multi-project switching built in, so LP, SC, and SI could share one codebase. The scope was set to close the loop end-to-end — from bulk execution setup on one side, to real-time failure visibility in Teams on the other — rather than stopping at the most visible pain point.

## 4. What Was Built

- CLI tool for bulk Jira-Xray test execution creation, test association, and test plan linking, with execution-type selection (smoke / sanity / regression) and multi-project switching.
- Bulk manual test repository export script for cross-referencing test inventories.
- Cookie-based session management for Robot Framework suites authenticating via Microsoft SSO — CDP-based cookie injection, LAPSID polling, post-injection URL validation as the true session-validity signal, and user-mismatch detection.
- Robot Framework `output.xml` parser posting formatted failure reports to Microsoft Teams via Incoming Webhook and Adaptive Cards, built with only Python's standard library, with a root-cause extraction heuristic that strips assertion boilerplate down to the actual failure reason.
- GitHub Actions integration with sparse checkout for CI-triggered runs.
- Resolved along the way: Bearer vs. Basic Auth mismatches, GraphQL numeric `issueId` resolution, mandatory custom-field formatting quirks, and handling of Xray's 202 Accepted async responses.

## 5. Outcome

| Metric | Before | After |
|---|---|---|
| Test execution creation + association (~100 executions, 2000+ tests) | ~2 days (manual) | ~10 minutes |
| CI/CD result sync to Jira-Xray | Manual result checking | Real-time, per suite completion |
| Association errors | Recurring (human error) | Eliminated |

**Example scale:** for Regression 10.2.xx, 100+ test executions were created feature/module-wise, with 2000+ tests associated across them. CI/CD batches trigger automatically on successful build deployment; results post to Jira-Xray as each test suite (10–15 tests) completes, whether suites run sequentially or in parallel.

**Net effect:** a multi-day, error-prone manual process became a ~10-minute automated one, with human association errors eliminated entirely.

## 6. What I'd Do Differently

- **SSO/cookie-based session handling** — Cookie-based auth via CDP injection works, but it's inherently fragile since it depends on Microsoft's login flow staying stable. A service-account or app-registration based auth flow (e.g., OAuth client credentials) would remove that dependency on browser-session mimicry entirely.
- **Multi-project support** — This was retrofitted after the tool proved out on one project first, which meant rework to generalize field mappings and project-specific config. Designing the project-abstraction layer upfront — even before a second project needed it — would have saved that rework cycle.
- **Scaling beyond current volume** — At 2000+ tests the tool performs well, but wasn't built with explicit rate-limiting or retry/backoff handling for Xray's API. At significantly higher volume, I'd add batched requests with backoff and resumable state, so a partial failure doesn't require re-running the entire batch from scratch.
- **Observability on the tool itself** — The tool automates a critical release-gating process but has no monitoring/alerting on itself — a silent sync failure might only be noticed when someone checks Jira. A lightweight health-check or failure alert, reusing the existing Teams webhook, would close that gap.

---

*Full implementation available on request during interviews.*
