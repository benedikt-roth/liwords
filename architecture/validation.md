# Validation and acceptance report

Reviewed on **2026-11-13** against baseline **`739d748c26e82067424f2ea5c5c7958bcd197485`**. The acceptance criteria were defined in [the analysis plan](00-analysis-plan.md) before modelling. The result meets those criteria for a code-derived architecture overview; the operational and domain-expert limitations below remain explicit.

## Requirement-by-requirement audit

| Requirement | Evidence in the delivered artefacts | Result |
| --- | --- | --- |
| Structured approach and expected artefacts for each phase | The plan defines separate container discovery, DDD discovery and quality-gate steps, their outputs and acceptance criteria. | Met |
| C4 level 2 container diagram in Mermaid | `containers.md` contains people, the Woogles system boundary, application/data-store containers, external applications, technologies, responsibilities and labelled protocols. Supporting executable jobs/functions and external integrations are inventoried separately. | Met |
| Thorough, pragmatic domain model | Twelve inferred contexts; two entity/relationship diagrams; aggregate candidates, value objects, identity/reference distinctions, persistence, rules, services and consistency boundaries in `domain/`. | Met |
| Shared language and quick transparency | `domain/language.md` contains 76 glossary entries plus ten ambiguity distinctions. README gives a short reading path; context and aggregate tables provide an overview before detailed workflows. | Met |
| Functionality and important behavior covered | All 23 registered RPC services mapped below; socket commands, non-RPC routes, background jobs and eight principal workflows checked. | Met |
| Codebase is the primary source | Twenty evidence groups link entry points, handlers, stores, SQL, protocols, frontend and selected existing tests. Manuals/protocol documentation are secondary and their limitations are recorded. | Met |
| Results stored under `architecture` with README | Nine Markdown documents are present in the repository's `architecture/` directory; README links all eight supporting documents. | Met |
| Verify achieved results and adequate quality | Mermaid parsing/rendering, screenshot inspection, link/structure checks, capability reconciliation and source-based challenge review performed as described below. | Met |

## Container coverage

Reconciled against `docker-compose.yml`, the API/socket entry points, store construction, the deployment workflow, executable directories and service invocation code:

- Interactive/runtime core: browser application, application API, socket server, NATS, PostgreSQL, Redis, S3/MinIO and Macondo bot are in the main diagram and responsibility table.
- External execution: analysis workers and OBS/event consumers are identified as external applications. Browser WASM workers are correctly inside the frontend container.
- Supporting execution: maintenance, puzzle generation, COP pairing Lambda, PDF Lambda and the migration task are identified with their callers/stores. Offline administrative utilities are distinguished from long-running applications.
- Delivery/operations: local Traefik/Rsbuild, static S3 delivery and Jaeger/OpenTelemetry are described without inventing a complete production topology.
- Integrations: email, notifications, subscription/OAuth, organizations, tournament feeds, monitoring, human check and donation checkout are traced to their call sites.

The diagram is a **C4 container view expressed with Mermaid flowchart primitives**, not a component diagram and not a Docker deployment diagram. An initial native `C4Container` render had overlapping relationship labels; the final notation preserves C4 semantics and renders legibly with standard Mermaid layout.

## Application capability reconciliation

The 23 `New…ServiceHandler` registrations in `cmd/liwords-api/main.go` were compared with all `service` declarations in local protobuf sources. Counts match; no contract service is unregistered and no registered service lacks a contract. Each is accounted for here:

| Registered services | Domain / documented capability |
| --- | --- |
| `AuthenticationService`, `RegistrationService`, `ProfileService`, `OrganizationService`, `AutocompleteService`, `AuthorizationService` | Identity and Access: accounts, sessions, profiles, membership/verification, user lookup and permissions. |
| `SocializeService`, `ModService` | Community and Trust: social links, chat/presence access, conduct actions and notoriety. |
| `GameMetadataService` | Gameplay/history, exports, active/recent/correspondence game views. |
| `GameEventService`, `GameCommentService`, `CollectionsService` | Study and Annotation: editing/import, comments and ordered collections. |
| `TournamentService`, `PairService` | Tournaments and Clubs: management/results, scorecards and remote COP pairing. |
| `LeagueService` | Leagues: recurrence, registration, placement, play, standings and administration. |
| `PuzzleService` | Puzzles: selection, attempts, solutions, voting and generation. |
| `AnalysisService`, `AnalysisQueueService`, `AnalysisAdminService` | Analysis: requester, worker and administrative capabilities. |
| `BroadcastService` | Broadcast event/feed/claim/slot operations and presentation. |
| `IntegrationService` | Integrations and Entitlements: connected provider state and removal. |
| `WordService` | Shared lexical support: word validity, definitions and anagram queries. |
| `ConfigService` | Shared application configuration and administrative settings. |

**Additional interfaces checked:** `/ws` plus API bus routing cover matchmaking, gameplay, readiness and chat commands that are not exposed as independent RPC services. Image/memento, embed, OAuth callbacks, VDO webhook, optional human check, OBS event/game/user endpoints and the NDJSON game-event stream are accounted for in the container/workflow descriptions. Health/profiling/debug endpoints are operational interfaces, not extra business contexts.

**Frontend cross-check:** `App.tsx` routes corroborate lobby/play, tournaments/clubs, leagues, puzzles, annotation/editor/embed, broadcasts, collections, registration/profile/settings, documentation/admin and donations. The WASM loader/lab implementation corroborates local computation distinct from queued server analysis.

## Source-based challenge review

The following potentially misleading interpretations were specifically checked against implementations and corrected or qualified:

| Concern | Verified conclusion |
| --- | --- |
| Gameplay package names imply independent services | `omgwords`, `cwgame`, `xwordgame` and bus logic are packages inside the API; the future-service README does not establish a deployed service. |
| History log implies current event-sourced position reads | Active served native reads still use serialized history. Turn reconstruction is commented out; dual writes, shadows and ongoing-game paths are conditional migration mechanisms. |
| `game.Cache` owns cached live positions | Its mutable-game cache has been removed; the remaining listing cache is a different responsibility. |
| Every game waits for both people online | The bus auto-starts newly created correspondence games; real-time readiness follows a separate path. |
| Game completion is one transaction | Game save/staged-turn atomicity is narrower than ratings, statistics, competition updates, notifications, listings, analysis and S3 archival. |
| Analysis requester must be a player | The handler allows any authenticated requester of a completed supported game; the retained `NOT_A_PLAYER` protocol enum does not prove otherwise. |
| A wrong puzzle answer is always terminal | Attempt completion is nullable; wrong guesses can remain unresolved, while revealing the solution resolves as given up. |
| Annotation ownership provides concurrency locking | Document read/modify/write is not locked; schema versioning is not optimistic concurrency. |
| All slots share one owner/lifecycle | Broadcast slots are event-owned; personal stream slots belong to users and can follow those event slots. |
| “Private” means one universal access rule | Collection discovery, rack censorship, annotation visibility, tournament analysis and verification-image access are distinct policies. |

Existing concurrency, atomicity, archival-contract, shadow-load, league-lifecycle and OBS-slot tests were inspected as corroborating specifications (E20). No claim is made that those application tests were executed.

## Diagram and documentation checks

- Extracted **all nine Mermaid blocks** from the actual delivered Markdown and called both `mermaid.parse` and `mermaid.render` with **Mermaid 11.17.2** in existing Chromium through the installed Playwright CLI. **9/9 passed**, producing nonempty SVG view boxes.
- Inspected rendered screenshots of the container view, context map, both class models, gameplay sequence, and game/season/puzzle/analysis lifecycles. Simplified the context map and shortened/reoriented lifecycle labels to remove overlaps.
- Resolved every relative Markdown file/directory link from its containing document; no missing targets remain. External references were consulted during analysis; this is not a continuous availability check of third-party URLs.
- Checked expected files, balanced code fences, one document title per file, absence of trailing whitespace and complete RPC coverage. Checked the final worktree scope: only `architecture/` was added; application code and dependency manifests are unchanged.
- Mermaid and browser-validation dependencies, screenshots and scripts were isolated under `/tmp`; they are not application dependencies or required repository artefacts.

For a future refresh, repeat extraction of every fenced `mermaid` block, parse **and** render them in a browser, inspect screenshots, resolve relative Markdown targets, and compare protobuf service declarations with API registrations. Useful inventory commands from the repository root are:

```sh
rg -n 'New.*ServiceHandler' cmd/liwords-api/main.go
rg -n '^service ' api/proto
rg -n 'router.Handle|go .*Start|StartPoller' cmd/liwords-api/main.go
git diff --check -- architecture
```

`git diff --check` alone does not inspect untracked files; the initial review additionally inspected the new documents directly for whitespace/structure.

## Limits and follow-up questions

The analysis is complete as repository documentation, with these boundaries: no production deployment, availability, performance, security audit or live end-to-end exercise was performed. Bounded contexts and aggregate candidates need maintainer/domain-expert agreement before being used as refactoring requirements. Production topology, schedules, enabled migration flags and external worker internals remain unverified. The [uncertainty register](evidence.md) captures these questions without presenting them as implementation facts.
