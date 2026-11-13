# Internationalization support initiative

Status: Initial draft for maintaining-team review. This folder contains the
feature specification, codebase analysis, clarification decisions and proposed
architecture for the first iteration. It does not implement the migration.

The initiative prepares application-owned, user-facing text for translation
while shipping only US English (`en-US`). The structure must allow community
translations later. Start with the [feature specification](feature-specification.md)
for the requirements, delivery plan and acceptance criteria.

## First-iteration overview

- Cover player, admin and director UI; help and legal content; badge descriptions;
  owned embeds and generated outputs; speech, browser notifications and
  transactional emails, subject to the documented content and API exclusions.
- Give user-facing backend outcomes stable message codes and named parameters.
  Internal UI-only APIs send codes for UI translation; shared UI/external APIs
  render user-facing text on the backend. External-only APIs and hidden
  machine-to-machine text are outside this iteration.
- Save language as a private account preference across devices. Signed-out
  users use `en-US`. Add a hidden language selector in experimental settings;
  the repository owner still chooses how testers reveal it.
- Base human-facing date, time and number formats on UI language while keeping
  timezone independent. Preserve existing help URLs and deep links. Keep
  historical system-message records unchanged and translate known legacy
  formats for display; store structured codes for new messages.
- Require automated catalog checks and maintaining-team review of any obvious
  English wording corrections. Older browser tabs must reload during rollout.

Additional production languages, user-authored content translation, announcement
content, organization titles and Discord localization are excluded. Discord work
is recorded as [I18N BL 001](backlog.md#i18n-bl-001-discord-notification-localization).
The specification defines the full scope and its exceptions.

## Files and folders

| Path | Purpose |
| --- | --- |
| [feature-specification.md](feature-specification.md) | Draft requirements, scope, delivery steps, acceptance criteria and maintaining-team decisions. |
| [decisions.md](decisions.md) | Confirmed answers from the clarification discussion, including scope and compatibility requirements. |
| [api-impact.md](api-impact.md) | Technical analysis of potentially affected RPCs, websocket events, stored messages and shared integration boundaries. |
| [api-classification.md](api-classification.md) | Consumer classes, translation policies, caller evidence and limits of the inventory. |
| [api-classification.csv](api-classification.csv) | Method-level inventory of 253 RPCs, including classifications and source/caller references. |
| [adr/](adr/README.md) | Eight architectural decision records covering help URLs, message contracts and runtimes, account locale, secondary renderers and email, selector access, rollout, historical messages and catalog checks. The index records each ADR's status and owner action. |
| [english-copy-audit.md](english-copy-audit.md) | Audit template for explicit before/after wording corrections and maintaining-team review. No corrections have been proposed or applied during planning. |
| [backlog.md](backlog.md) | Deferred work, currently Discord notification localization. |

## Review and next steps

1. Review the [specification](feature-specification.md) alongside the confirmed
   [clarification decisions](decisions.md). Architectural recommendations remain
   proposals wherever an owner decision is listed.
2. Review the [API inventory](api-classification.md) and [ADRs](adr/README.md).
   API assignments include provisional and unassigned methods; verify consumers
   before changing each contract. [ADR 0005](adr/0005-experimental-language-selector-access.md)
   stays open for the repository owner: a dedicated `I18n Tester` role
   (recommended) or a developer-tools flag.
3. Follow the [first-iteration delivery plan](feature-specification.md#first-iteration-delivery-plan):
   resolve contracts and inventory, build the foundation, validate an end-to-end
   slice, migrate the remaining domains, then complete the coverage and rollout
   review. The slice alone does not complete the iteration.

Keep confirmed requirements in `decisions.md` and `feature-specification.md`,
implementation choices and their status in the ADRs, deferred scope in
`backlog.md`, and English wording corrections in `english-copy-audit.md`.
