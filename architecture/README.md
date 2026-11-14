# Woogles architecture

Woogles is a crossword board-game platform for online play, organized competition, study and broadcasting. Its main backend is a Go application with many domain responsibilities; a separate Go socket server carries real-time traffic. PostgreSQL, Redis and NATS have distinct roles. The React application also performs analysis locally through WebAssembly.

This overview describes the code at **`739d748c26e82067424f2ea5c5c7958bcd197485`**, reviewed on **2026-11-13**. It is an account of the implementation and an inferred domain vocabulary, not a production infrastructure audit or a microservices proposal.

## Start here

| Document | What it answers |
| --- | --- |
| [Container architecture](containers.md) | What runs, what stores data, and how the parts communicate. Includes a Mermaid C4 level 2 diagram. |
| [Domain contexts](domain/contexts.md) | What the application does, where domain responsibilities belong, and how they interact. |
| [Domain model](domain/model.md) | Important entities, relationships, aggregate candidates, invariants and consistency boundaries. |
| [Ubiquitous language](domain/language.md) | Shared business terms, code equivalents and terms that must not be confused. |
| [Workflows and lifecycles](domain/workflows.md) | How matching, play, competitions, puzzles, analysis and broadcasting work end to end. |
| [Evidence and uncertainties](evidence.md) | Source index, secondary references, implementation caveats and unanswered questions. |
| [Analysis plan](00-analysis-plan.md) | The approach, expected artefacts and acceptance criteria defined before modelling. |
| [Validation report](validation.md) | Coverage checks, diagram/link verification and the limits of the analysis. |

For a quick orientation, read the container diagram, the context table and the terminology distinctions at the top of the language document. Use the detailed model and workflows when changing behavior.

## Reading conventions

- **Observed** means supported by executable code, configuration or schema. Evidence identifiers such as E03 refer to the [source index](evidence.md).
- **Inferred** means an interpretation of that code, especially bounded contexts, aggregate candidates and business-event names. These should be validated with maintainers and domain experts.
- A DDD context is a language and responsibility boundary. It does not imply a separate process, database, team or transaction.
- Diagrams are embedded Mermaid source. The C4 container view uses explicit C4 labels and a system boundary in Mermaid flowchart syntax; other views use flowcharts, class diagrams, sequence diagrams or state diagrams. Rendering details and validation are recorded in the validation report.

Important distinctions: a **user** is not always a **player**; a **seek** is not a **pairing**; a tournament **division** is not a league **division**; a game's **end** is not an annotator's **done** state; a **broadcast slot** is not a personal **stream slot**. These distinctions determine ownership and behavior throughout the model.

## Keeping this overview current

When changing entry points, protocols, store wiring, game persistence, RPC capabilities or lifecycle rules, update the affected document and its evidence references. Recheck links and Mermaid blocks, and record the reviewed revision. Keep deployment facts separate from assumptions and preserve the distinction between current behavior and migration scaffolding.
