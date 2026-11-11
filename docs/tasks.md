# FraudWatch backlog

Source: [project plan](./plan.md). Work starts with the frontend; this document describes future work and does not implement it.

Each task is scoped to one focused session and includes its prerequisites and completion check in its description. Dependencies refer to existing deliverables, not knowledge of another task's text; provide the assignee with the current repository and the linked plan. The numbered order is a suggested implementation sequence, except that the bounded dataset feasibility review should happen early during frontend work.

Frontend fixtures must be deterministic and visibly labeled as fictional preview data. They are not the MVP dataset, model results, or a substitute for server-side authorization. Real operation types, model features, SHAP factors, and risk bands remain provisional until data/model validation; public deployment, queues, advanced observability, and a full E2E suite remain outside scope.

## 1. Set up an empty frontend project with a passing test
Goal: Establish a runnable React, TypeScript, and Tailwind CSS project with one passing test.
Description: Create the minimal frontend package in the monorepo with a test runner and an empty application shell, without business features. Add one smoke test that renders the shell and document installation, local startup, and the test command so a fresh checkout can reproduce the passing result.

## 2. Define provisional frontend contracts
Goal: Give frontend features a consistent typed data boundary before a backend exists.
Description: Using the empty React project, define types and adapter interfaces for sessions, transactions, alerts, reservations, decisions, simulation status, account management, and aggregate metrics. Document loading/error responses, pagination, event envelopes, and provisional dataset-dependent fields, keeping historical labels out of investigation payloads and recording unresolved product choices instead of inventing final rules.

## 3. Add a deterministic preview data adapter
Goal: Let screens run locally without backend services.
Description: Implement the frontend adapter interfaces with a small fixed set of fictional transactions, client histories, accounts, scores, and alert explanations, plus configurable loading and failure responses. Include a visible preview-mode indicator and verify that fixtures are repeatable, contain no real credentials, and never claim to measure a trained model.

## 4. Build the application layout and routes
Goal: Provide consistent navigation for analyst and administrator screens.
Description: Using the React shell and preview adapter, add a responsive layout with routes for dashboard, transactions, investigation, evaluation, and account administration. Include accessible navigation and reusable loading, empty, and error states, and verify that analyst and administrator previews expose the appropriate navigation.

## 5. Build login and password screens
Goal: Preview authentication and mandatory password changes.
Description: Using the session adapter and application routes, implement login, temporary-password replacement, password change, and current-session or all-session logout controls. Cover invalid credentials, the five-failure/15-minute lockout message, restricted first-login navigation, and expired-session feedback with component checks, while clearly treating authentication as simulated.

## 6. Review real dataset feasibility early
Goal: Establish whether a legally usable dataset can support the promised demo.
Description: Independently of frontend completion, inspect a bounded shortlist of public datasets for real anonymized transactions, usable licensing, consistent customer identifiers, sortable timestamps, and fraud labels. Record evidence from actual columns and sample records, supported operation types, and acquisition instructions; if none qualifies within the session, report the blocker and alternatives for an explicit scope decision rather than substituting synthetic data.

## 7. Build the transaction table and query controls
Goal: Preview a filterable transaction list with server-style pagination.
Description: Using the transaction adapter and page layout, render newest-first rows with status, risk, date filters, transaction/customer ID search, and pagination controls backed by adapter queries. Check combined filters, empty results, loading/failure states, and resetting or clamping invalid pages without introducing configurable sorting.

## 8. Build the transaction investigation detail
Goal: Show the information an analyst needs without revealing historical outcomes.
Description: Using the detail and history adapter interfaces, display transaction/customer identifiers, amount, timestamp, score, reservation status, recent customer history, and SHAP factors for alerts. Handle missing explanations and evaluation errors, describe SHAP contributions without causal claims, and verify that historical fraud labels never appear in the view.

## 9. Build reservation and decision interactions
Goal: Preview exclusive alert handling and justified decisions.
Description: Extend the investigation screen and preview adapter with reserve, release, approve, reject, and mark-under-investigation actions, requiring a nonblank justification for each decision. Check ownership conflicts and immutable final decisions, keep under-investigation transactions held, and record the provisional reservation behavior for that nonfinal action for confirmation in the backend design.

## 10. Build shared simulation controls
Goal: Preview start, pause, exhaustion, and confirmed reset behavior.
Description: Using the simulation adapter and analyst layout, implement status and start/pause/reset controls with an explicit warning before deleting a run's work. Make the preview advance chronologically every three seconds, stop at exhaustion, retain investigation access while paused, and preserve accounts on reset; keep administrator controls unavailable pending the explicit permissions decision.

## 11. Build dashboard indicators and charts
Goal: Present the shared operational overview using preview data.
Description: Using aggregate adapter responses, show processed transactions, alerts, high-risk operations, review states, and a compact transaction trend chart. Verify loading, empty, and failed responses, and document provisional risk-band semantics so the interface does not silently define the model's threshold.

## 12. Build the evaluation screen
Goal: Display model-quality aggregates separately from investigation.
Description: Using preview evaluation responses, render precision, recall, false positives, and false negatives for analysts and administrators in read-only mode. Handle zero denominators and small samples explicitly, label the values as fictional in preview mode, and explain that live values will cover only processed historical transactions.

## 13. Build account administration screens
Goal: Preview account creation, deactivation, and password reset.
Description: Using the account adapter and administrator route, add account listing and the forms for creating/managing accounts, assigning temporary passwords, and deactivating users. Verify required fields, mutation failures, confirmation feedback, and analyst route denial, with messaging that deactivation releases reservations and temporary passwords require replacement.

## 14. Add preview live updates and alert notices
Goal: Exercise frontend updates before real WebSockets exist.
Description: Implement a test event source behind the frontend adapter that refreshes dashboard, evaluation, reservation detail, and transaction queries while preserving active filters and page selection when valid. Add temporary visual alert notifications and connection/reconnection states, and verify updates on a nonfirst page, reset handling, and duplicate-notice suppression with focused component tests.

## 15. Define service contracts and state transitions
Goal: Specify how the three services cooperate without ambiguous ownership.
Description: Using the plan and provisional frontend contracts, document REST request/response schemas, call direction, Core API persistence/event ownership, transaction and reservation transitions, run identifiers, idempotency, timeouts, and reset sequencing. Resolve administrator simulator permissions, reservation behavior after marking under investigation, and SHAP-failure behavior, then review the contract against duplicate submissions, stale-run requests, and ML failures without adding queues.

## 16. Define authentication and session contracts
Goal: Specify revocable authentication for browser and service traffic.
Description: Using the three-service contract, document JWT access/refresh lifetimes, secure browser transport/storage, refresh renewal, session revocation checks, WebSocket authentication/disconnection, and service credentials. Include the role matrix, first-login restriction, five-failure/15-minute lockout, current/all-session logout, password-change invalidation, and immediate deactivation semantics so implementation has explicit acceptance rules.

## 17. Define persistence and migration strategy
Goal: Establish a database design that supports concurrency and reset isolation.
Description: Using the service and session contracts, select and document SQL scripts or Alembic and define tables, relationships, constraints, and indexes for accounts, sessions, runs, transactions, restricted historical labels, explanations, alerts, reservations, and decisions. Document atomic reservation and decision boundaries and how reset deletes run data while preserving accounts and configuration, checking the schema against the agreed state transitions.

## 18. Prepare the dataset ingestion and temporal splits
Goal: Produce reproducible training, validation, and replay inputs.
Description: Once a compatible real dataset has been approved, implement documented acquisition/normalization and chronological training, validation, and later replay/evaluation partitions with a manifest of source version and split boundaries. Validate identifiers, timestamps, labels, duplicate handling, and temporal separation, storing data only as the license permits and keeping labels separate from analyst-facing records.

## 19. Implement shared model feature transformations
Goal: Keep training and inference inputs identical and free of future information.
Description: Using the normalized dataset and temporal split manifest, implement versioned feature transformations with a documented schema, missing-value behavior, and any customer-history features limited to information available at transaction time. Check the same sample through training and inference transformations and verify that labels, future transactions, and validation/replay data cannot influence fitted preprocessing.

## 20. Train and evaluate a baseline model
Goal: Produce a reproducible candidate model with temporal validation results.
Description: Using the shared feature pipeline and training/validation partitions, train one baseline classifier offline with a recorded seed, dependency versions, and dataset/feature versions. Report precision, recall, confusion counts, and score distribution on validation data and save the candidate artifact without tuning against the later replay/evaluation partition.

## 21. Select the fixed threshold and validate SHAP
Goal: Freeze an inference artifact whose alert decisions and explanations are practical.
Description: Using the trained candidate and validation results, choose and document a fixed threshold and any display risk bands, then check SHAP compatibility and measured inference/explanation cost on representative samples. Export the model, preprocessing, threshold, and version metadata together, and record a blocker if explanations are not viable rather than reporting unsupported factors.

## 22. Reconcile frontend contracts with real data
Goal: Replace provisional field assumptions with validated dataset and model semantics.
Description: Using the approved dataset schema and frozen model metadata, update the documented API/frontend contracts and preview adapter mappings for actual operation types, available attributes, risk levels, and SHAP features. Check each existing screen against representative normalized records, retain preview labeling, and remove any UI promise that the source cannot support.

## 23. Scaffold the three backend services
Goal: Make Core API, Simulator Service, and ML Service independently runnable.
Description: Using the agreed REST contracts, create minimal FastAPI packages with environment-based configuration, health endpoints, and generated OpenAPI/Swagger UI. Add a focused health/configuration smoke test per service and document local commands without implementing business workflows in this task.

## 24. Add local Docker Compose orchestration
Goal: Start the frontend, three service shells, and PostgreSQL locally.
Description: Using the runnable frontend and FastAPI shells, add container builds, Compose networking, health checks, local volumes, and a documented environment-variable example without committed secrets. Verify a clean local startup and frontend/API reachability, keeping ML artifact/data setup explicit and avoiding any public deployment configuration.

## 25. Implement account and session persistence
Goal: Store authentication state using the selected migration mechanism.
Description: Using PostgreSQL and the approved schema, add account/session migrations, secure password hashing, and an explicit local bootstrap procedure for the first administrator. Verify fresh database migration, hash verification, uniqueness constraints, and that bootstrap does not silently overwrite an existing account or ship a fixed password.

## 26. Implement login, refresh, and protected access
Goal: Authenticate active users and enforce roles on backend requests.
Description: Using account/session persistence and the authentication contract, implement access/refresh token issuance, renewal, active-session checks, role guards, and restricted sessions for temporary passwords. Test invalid/expired/revoked tokens, unauthorized roles, refresh renewal behavior, and the rule that users awaiting password replacement cannot access business APIs.

## 27. Implement password replacement and session logout
Goal: Enforce session invalidation when credentials change or users sign out.
Description: Using authenticated Core API sessions, add mandatory first-login replacement, ordinary password change, and logout of either the current session or all sessions. Verify that password changes revoke every prior session and logout preserves alert reservations, with other sessions surviving only a current-session logout.

## 28. Implement login lockout
Goal: Block password guessing after five incorrect attempts for fifteen minutes.
Description: Using the Core API login endpoint and account persistence, store failed-attempt and lockout state with explicit reset/expiry behavior. Add clock-controlled tests for the fifth failure, attempts during lockout, expiry, and concurrent failures so the rule does not depend on a single server process's memory.

## 29. Implement account administration APIs
Goal: Let administrators create and manage accounts and temporary credentials.
Description: Using role-protected Core API endpoints, implement account listing, approved account-field management, creation, and administrator password reset with mandatory replacement on next login. Test analyst denial, duplicate accounts, password hashing, and revocation of existing sessions after a reset, leaving deactivation's reservation cleanup to the dedicated lifecycle task.

## 30. Implement simulation-domain persistence
Goal: Persist run-scoped transactions, alerts, reservations, and decision history.
Description: Using the agreed PostgreSQL design and existing migration setup, add the run and transaction-domain tables, restricted label storage, explanation storage, and repositories with uniqueness and foreign-key constraints. Verify that duplicate transaction identifiers are scoped to a run and that simulation records can be removed without deleting accounts or configuration.

## 31. Secure REST calls between services
Goal: Limit internal simulation and prediction APIs to trusted services.
Description: Using the service authentication contract and FastAPI shells, implement credential validation and reusable REST clients with configured timeouts and consistent error mapping. Check missing/invalid credentials, unreachable services, and malformed responses, keeping service secrets outside source control and browser payloads.

## 32. Implement ML prediction and alert explanations
Goal: Serve versioned model scores and SHAP explanations through REST.
Description: Using the frozen model artifact, shared feature transformations, and secured ML Service, load the model and expose validated prediction responses with model version and fixed-threshold metadata. Test below/equal/above-threshold outcomes, invalid input, unavailable artifacts, and SHAP generation only for alerts, following the agreed explanation-failure contract.

## 33. Implement transaction ingestion and evaluation
Goal: Persist each replayed transaction once and apply safe ML outcomes.
Description: Using Core API domain persistence and the ML REST client, accept run-scoped transaction submissions, score them, auto-approve below threshold, and retain threshold-reaching transactions with an alert and explanation. Test duplicate delivery, stale-run rejection, threshold equality, and timeout/invalid-response paths that retain an evaluation-error transaction rather than approving without a score.

## 34. Implement manual evaluation retry
Goal: Allow analysts to recover retained transactions after ML failure.
Description: Using transaction ingestion and analyst authorization, add an explicit retry endpoint for evaluation-error transactions that reuses the normal scoring/classification rules. Test repeated failure, successful recovery, concurrent retries, duplicate alert prevention, and denial for administrators or already finalized transactions.

## 35. Implement chronological replay and pause
Goal: Emit one historical transaction every three seconds into the shared run.
Description: Using normalized replay data, secured Core API ingestion, and persisted run contracts, implement Simulator Service start/pause/status behavior with a single shared cursor and exhaustion state. Use a controllable clock to verify chronological order, pacing, duplicate start requests, pause/resume, and the agreed retry behavior after delivery failure without disabling investigation of existing alerts.

## 36. Implement atomic alert reservation
Goal: Ensure only one analyst owns an alert at a time.
Description: Using Core API alert persistence and active-user guards, add atomic take/release operations with ownership checks and no inactivity expiry. Test two analysts taking the same alert concurrently, unauthorized release, and reservation persistence after logout, returning an explicit conflict to the losing request.

## 37. Implement justified investigation decisions
Goal: Record traceable decisions while preventing conflicting final outcomes.
Description: Using reserved alerts and the agreed transition rules, implement approve, reject, and under-investigation actions requiring the current owner and a nonblank justification, storing analyst identity and timestamp. Test concurrent or duplicate final decisions, administrator denial, and continued retention after under-investigation, applying the documented reservation behavior for each transition.

## 38. Implement account deactivation and reservation cleanup
Goal: Immediately prevent a deactivated analyst from operating.
Description: Using account/session persistence and reservation APIs, implement administrator deactivation that revokes sessions, blocks renewal and protected requests, and releases the user's reservations consistently. Test deactivation racing with an alert decision and expose the revocation signal needed for active WebSocket disconnection, with connection closure verified when the WebSocket layer is available.

## 39. Implement consistent global reset
Goal: Clear a simulation run without allowing stale work to reappear.
Description: Using run persistence, simulator controls, and decision APIs, implement the reset contract that stops replay, invalidates the previous run, removes its transactions/alerts/decisions/explanations, and resets the cursor while retaining accounts and configuration. Test reset racing with ingestion, an in-flight ML response, and a decision, plus repeated reset requests and a partial service failure, before allowing the next run to start.

## 40. Implement transaction and history query APIs
Goal: Supply paginated investigation data without historical-label leakage.
Description: Using Core API domain persistence and role guards, expose newest-first paginated queries with status/risk/date filters and transaction/customer ID search, plus transaction detail and recent customer history. Test stable tie ordering, filter combinations, invalid page inputs, and response serialization that excludes historical labels and future customer activity.

## 41. Implement operational aggregate APIs
Goal: Supply dashboard counts and chart series from persisted run state.
Description: Using transaction, alert, and decision persistence, expose processed totals, alert/high-risk counts, review-state totals, and the dashboard trend series for authorized read-only access. Check empty runs, state transitions, retry deduplication, and reset behavior so aggregate values match the underlying records.

## 42. Implement model evaluation aggregates
Goal: Calculate valid live metrics from scored historical transactions.
Description: Using restricted labels and recorded model predictions, compute precision, recall, false positives, and false negatives only for processed, successfully scored transactions in the current run, independently of analyst decisions. Test zero denominators, evaluation errors, successful retries, resets, and role access, returning aggregates without exposing per-transaction labels in investigation APIs.

## 43. Implement authenticated WebSocket events
Goal: Notify active clients about committed state changes and revoke invalid connections.
Description: Using Core API session checks and committed domain mutations, publish the agreed run-aware events for transactions, alerts, reservations, decisions, metrics, simulation status, and reset. Test connection authentication, session expiry/logout/password-change/deactivation disconnection, and event ordering around commits and resets, treating REST state as authoritative after missed messages.

## 44. Connect frontend authentication and account management
Goal: Replace preview identity and account operations with real APIs.
Description: Using the completed session/account endpoints and browser authentication contract, implement the production frontend adapter for login, refresh, password flows, logout, and account administration. Verify expired/revoked sessions, first-login restrictions, lockout feedback, and role-specific navigation with adapter/component integration tests, retaining an explicit separate preview mode.

## 45. Connect frontend transaction reads and metrics
Goal: Render actual persisted transactions, histories, and aggregate results.
Description: Using the real transaction, operational, and evaluation APIs, replace preview reads for the table, investigation detail, customer history, dashboard, and evaluation screens. Verify query serialization, pagination/filter preservation, loading/errors, score/SHAP mapping, and the absence of historical labels in investigation responses.

## 46. Connect frontend workflow mutations
Goal: Operate real reservations, decisions, retries, and simulator controls.
Description: Using the implemented mutation APIs and frontend interaction screens, wire reserve/release, justified decisions, manual reevaluation, start/pause, and confirmed reset to the real backend. Test conflict feedback, duplicate-click handling, permission failures, and authoritative refresh after mutation, including keeping investigation usable while replay is paused.

## 47. Connect live updates and reconnection recovery
Goal: Keep every open screen consistent with backend state after interruptions.
Description: Using authenticated WebSocket events and real frontend query adapters, invalidate/refetch affected views, show temporary alert notices, and reconnect automatically with a REST resynchronization before declaring the view current. Test a nonfirst filtered table page, missed events, duplicate alerts, reset during disconnection, and session revocation so stale run data or unauthorized reconnection cannot survive.

## 48. Verify cross-service scoring and recovery
Goal: Demonstrate the main transaction paths across real service boundaries.
Description: Using the composed services and deterministic permitted test fixtures, add integration checks for below-threshold approval, threshold-reaching retention with SHAP, ML unavailability, and successful manual retry. Assert persisted outcomes and aggregate counts through REST/PostgreSQL without building a full browser E2E suite, and document the reproducible test command.

## 49. Verify multiuser concurrency and lifecycle behavior
Goal: Confirm invariants across sessions, reset, and account deactivation.
Description: Using the completed APIs and WebSockets, add focused integration tests for competing reservations, competing final decisions, logout-preserved ownership, deactivation release/disconnection, and reset against in-flight ingestion or decisions. Assert that exactly one permitted outcome commits and that old-run records cannot return, reusing existing unit checks rather than duplicating every implementation detail.

## 50. Document and rehearse the local demo
Goal: Let another person run and explain the completed project from a fresh checkout.
Description: Using the working Compose stack and validated data/model artifacts, document setup, legal data acquisition, offline training/artifact preparation, administrator bootstrap, test commands, service Swagger URLs, architecture, and preview-versus-real mode. Rehearse the plan's demo with two analyst sessions, failure/retry, pause, evaluation, and confirmed reset, recording actual limitations and verifying that accounts survive reset and no public deployment is required.
