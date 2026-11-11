# FraudWatch backlog

Source: [project plan](./plan.md). Work starts with the frontend; implementation is tracked in GitHub issues.

Each issue is scoped to one focused session and includes its prerequisites and completion check in its description on GitHub. Dependencies refer to existing deliverables, not knowledge of another task's text; provide the assignee with the current repository and the linked plan. The numbered order is a suggested implementation sequence, except that the bounded dataset feasibility review should happen early during frontend work.

Frontend fixtures must be deterministic and visibly labeled as fictional preview data. They are not the MVP dataset, model results, or a substitute for server-side authorization. Real operation types, model features, SHAP factors, and risk bands remain provisional until data/model validation; public deployment, queues, advanced observability, and a full E2E suite remain outside scope.

## GitHub issues

All 50 original task descriptions have moved to [GitHub Issues](https://github.com/lohengrin-01/fraudwatch/issues). PM grooming added two documentation follow-ups, bringing the backlog to 52 issues. Each issue includes Goal, Acceptance criteria, Out of scope, and Constraints; GitHub is the source of truth for task details and status.

The links below preserve the original task numbering, not a strict dependency order. Complete the testing guide (#52) before writing tests, including task #1's smoke test, and the design guide (#51) before styled UI work. Task #1 remains the first implementation task; both guides are documentation-only prerequisites.

| Task | Issue |
|---|---|
| 1 | [Set up an empty frontend project with a passing test](https://github.com/lohengrin-01/fraudwatch/issues/1) |
| 2 | [Define provisional frontend contracts](https://github.com/lohengrin-01/fraudwatch/issues/2) |
| 3 | [Add a deterministic preview data adapter](https://github.com/lohengrin-01/fraudwatch/issues/3) |
| 4 | [Build the application layout and routes](https://github.com/lohengrin-01/fraudwatch/issues/4) |
| 5 | [Build login and password screens](https://github.com/lohengrin-01/fraudwatch/issues/5) |
| 6 | [Review real dataset feasibility early](https://github.com/lohengrin-01/fraudwatch/issues/6) |
| 7 | [Build the transaction table and query controls](https://github.com/lohengrin-01/fraudwatch/issues/7) |
| 8 | [Build the transaction investigation detail](https://github.com/lohengrin-01/fraudwatch/issues/8) |
| 9 | [Build reservation and decision interactions](https://github.com/lohengrin-01/fraudwatch/issues/9) |
| 10 | [Build shared simulation controls](https://github.com/lohengrin-01/fraudwatch/issues/10) |
| 11 | [Build dashboard indicators and charts](https://github.com/lohengrin-01/fraudwatch/issues/11) |
| 12 | [Build the evaluation screen](https://github.com/lohengrin-01/fraudwatch/issues/12) |
| 13 | [Build account administration screens](https://github.com/lohengrin-01/fraudwatch/issues/13) |
| 14 | [Add preview live updates and alert notices](https://github.com/lohengrin-01/fraudwatch/issues/14) |
| 15 | [Define service contracts and state transitions](https://github.com/lohengrin-01/fraudwatch/issues/15) |
| 16 | [Define authentication and session contracts](https://github.com/lohengrin-01/fraudwatch/issues/16) |
| 17 | [Define persistence and migration strategy](https://github.com/lohengrin-01/fraudwatch/issues/17) |
| 18 | [Prepare the dataset ingestion and temporal splits](https://github.com/lohengrin-01/fraudwatch/issues/18) |
| 19 | [Implement shared model feature transformations](https://github.com/lohengrin-01/fraudwatch/issues/19) |
| 20 | [Train and evaluate a baseline model](https://github.com/lohengrin-01/fraudwatch/issues/20) |
| 21 | [Select the fixed threshold and validate SHAP](https://github.com/lohengrin-01/fraudwatch/issues/21) |
| 22 | [Reconcile frontend contracts with real data](https://github.com/lohengrin-01/fraudwatch/issues/22) |
| 23 | [Scaffold the three backend services](https://github.com/lohengrin-01/fraudwatch/issues/23) |
| 24 | [Add local Docker Compose orchestration](https://github.com/lohengrin-01/fraudwatch/issues/24) |
| 25 | [Implement account and session persistence](https://github.com/lohengrin-01/fraudwatch/issues/25) |
| 26 | [Implement login, refresh, and protected access](https://github.com/lohengrin-01/fraudwatch/issues/26) |
| 27 | [Implement password replacement and session logout](https://github.com/lohengrin-01/fraudwatch/issues/27) |
| 28 | [Implement login lockout](https://github.com/lohengrin-01/fraudwatch/issues/28) |
| 29 | [Implement account administration APIs](https://github.com/lohengrin-01/fraudwatch/issues/29) |
| 30 | [Implement simulation-domain persistence](https://github.com/lohengrin-01/fraudwatch/issues/30) |
| 31 | [Secure REST calls between services](https://github.com/lohengrin-01/fraudwatch/issues/31) |
| 32 | [Implement ML prediction and alert explanations](https://github.com/lohengrin-01/fraudwatch/issues/32) |
| 33 | [Implement transaction ingestion and evaluation](https://github.com/lohengrin-01/fraudwatch/issues/33) |
| 34 | [Implement manual evaluation retry](https://github.com/lohengrin-01/fraudwatch/issues/34) |
| 35 | [Implement chronological replay and pause](https://github.com/lohengrin-01/fraudwatch/issues/35) |
| 36 | [Implement atomic alert reservation](https://github.com/lohengrin-01/fraudwatch/issues/36) |
| 37 | [Implement justified investigation decisions](https://github.com/lohengrin-01/fraudwatch/issues/37) |
| 38 | [Implement account deactivation and reservation cleanup](https://github.com/lohengrin-01/fraudwatch/issues/38) |
| 39 | [Implement consistent global reset](https://github.com/lohengrin-01/fraudwatch/issues/39) |
| 40 | [Implement transaction and history query APIs](https://github.com/lohengrin-01/fraudwatch/issues/40) |
| 41 | [Implement operational aggregate APIs](https://github.com/lohengrin-01/fraudwatch/issues/41) |
| 42 | [Implement model evaluation aggregates](https://github.com/lohengrin-01/fraudwatch/issues/42) |
| 43 | [Implement authenticated WebSocket events](https://github.com/lohengrin-01/fraudwatch/issues/43) |
| 44 | [Connect frontend authentication and account management](https://github.com/lohengrin-01/fraudwatch/issues/44) |
| 45 | [Connect frontend transaction reads and metrics](https://github.com/lohengrin-01/fraudwatch/issues/45) |
| 46 | [Connect frontend workflow mutations](https://github.com/lohengrin-01/fraudwatch/issues/46) |
| 47 | [Connect live updates and reconnection recovery](https://github.com/lohengrin-01/fraudwatch/issues/47) |
| 48 | [Verify cross-service scoring and recovery](https://github.com/lohengrin-01/fraudwatch/issues/48) |
| 49 | [Verify multiuser concurrency and lifecycle behavior](https://github.com/lohengrin-01/fraudwatch/issues/49) |
| 50 | [Document and rehearse the local demo](https://github.com/lohengrin-01/fraudwatch/issues/50) |
| 51 | [Document the frontend design system](https://github.com/lohengrin-01/fraudwatch/issues/51) |
| 52 | [Document testing guidelines](https://github.com/lohengrin-01/fraudwatch/issues/52) |
