# Analysis approach and acceptance criteria

Prepared before modelling on 2026-11-13. This is a reverse-engineering exercise, not a proposed redesign. The code at `739d748c26e82067424f2ea5c5c7958bcd197485` is the primary source. Working tree was clean at the start.

## Phase 1 — C4 level 2: runtime composition

1. Inventory entry points, frontend bootstrapping, process wiring, build/deployment definitions and configuration.
2. Trace HTTP, WebSocket, messaging, database and object-storage connections from callers to implementations.
3. Separate executable applications and stores from in-process packages, libraries and external systems. Distinguish local configuration, deployment intent and unverified production details.
4. Model the main runtime in Mermaid, then inventory optional workers, integrations and operational concerns in supporting tables.

Expected artefacts: `containers.md`, containing a C4 container view, container responsibilities, protocols, significant runtime flows and deployment caveats; source references collected in `evidence.md`.

Acceptance: every principal runnable application and store is accounted for; the boundary, people, technologies and relationships are explicit; relationships have evidence; libraries are not presented as independently deployed services; optional or externally implemented capabilities are labelled.

## Phase 2 — pragmatic DDD domain model

1. Inventory the user-facing capabilities from routes and RPC contracts, then inspect handlers, domain logic, persistence and relevant tests.
2. Propose bounded contexts according to language, rules and lifecycle ownership. Identify where the implementation shares models, stores or orchestration across these conceptual boundaries.
3. Derive a ubiquitous language, including overloaded terms and mappings to code. Distinguish domain entities, value objects, aggregate candidates, services and read models.
4. Trace representative commands, state transitions and cross-context effects. Record enforced invariants, consistency limits and unresolved questions.
5. Cross-check optional public documentation only where it clarifies terminology or functionality; source code wins when versions disagree.

Expected artefacts:

- `domain/contexts.md`: capability inventory, bounded contexts, context map and ownership.
- `domain/model.md`: entity relationships, aggregate candidates, important rules and persistence mapping.
- `domain/language.md`: ubiquitous language with code mappings and ambiguity guidance.
- `domain/workflows.md`: principal scenarios, state transitions and cross-context interactions.
- `evidence.md`: traceable source index and uncertainty register.

Acceptance: live and correspondence play, matchmaking, rules, tournaments, leagues, puzzles, analysis, annotation/collections, broadcasts, identity, social activity, moderation and integrations are covered; key terms have unambiguous definitions; aggregate candidates are distinguished from proven transaction boundaries; important invariants and lifecycle transitions have implementation evidence; conceptual domain events are distinguished from actual messages.

## Phase 3 — quality gate

1. Reconcile the container inventory against executable wiring and deployment files.
2. Reconcile capability coverage against all registered RPC services and significant non-RPC routes.
3. Trace representative invariants and workflows to implementation and existing tests. Review statements for unsupported guarantees, particularly persistence, event delivery and transactionality.
4. Parse/render all Mermaid blocks, check repository links and review diagram legibility. Correct defects and record exactly which checks ran.
5. Provide `README.md` as the entry point and `validation.md` as the completed acceptance report, including limitations and open questions.

Documentation-only work does not require running application integration tests or deploying services. Validation focuses on evidence, coverage and the usability of the artefacts.
