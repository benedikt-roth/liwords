# ADR 0004 Secondary renderers and email

Status: Proposed implementation. Surface coverage, recipient email language and
the Discord exclusion are confirmed; renderer adapters remain owner choices.

## Context

The main React root is only one rendering path. The standalone embed has its
own build, initial embed HTML comes from Go, labels occur in CSS pseudo-elements
and canvas, speech helpers create English sentences, and browser notifications
run outside component rendering. The scorecard generator is a Python service.
Emails are rendered in backend/background code without a current UI session.

## Confirmed scope

Cover owned secondary UI, help/legal content, generated images/reports, speech,
browser notifications and transactional emails. Exclude external-only API output,
hidden machine-to-machine text and Discord integration localization. The OBS
builder/sample preview remains in scope while external OBS response formats do
not. Emails use recipient language with `en-US` fallback.

## Proposed approach

Inventory text by rendering surface and assign each message an authoritative
resource and renderer. Share the selected locale context without requiring
every renderer to be a React component. Initialize resources for both main and
embed entry points and for early/fallback UI that appears before normal app
providers are mounted.

Move functional text out of CSS `content` into renderable, accessible markup
where practical. Supply translated captions to canvas/image generators and
speech/notification helpers; update their cached output when locale changes.
Do not translate tile runes, board coordinates, usernames, URLs or stable game
notation. Speech language/voice selection must have a documented fallback and
must not change game-alphabet keyboard behavior.

Use shared language context for library-owned defaults and application-provided
overrides. Coordinate with the ongoing Ant Design/Mantine migration so a newly
introduced translation provider or notification adapter is not tied to a
temporary component facade.

For email, resolve the recipient's account preference or `en-US` before rendering
subject and body. Carry recipient/context through queued work; do not reuse the
triggering moderator's locale or require a live browser. Apply safe interpolation
and preserve links, delivery recipients and email behavior.

For generated reports/PDFs, choose a renderer/resource adapter explicitly and
pass locale/timezone context required by ADR 0003. Reading shared tournament data
does not make every field a translated caption. Preserve report data and source
IDs independently of displayed labels.

## Discord backlog boundary

Do not migrate Discord webhook templates, destination configuration or the
deployment notification script in this iteration. A shared underlying event
may still produce an in-scope email or UI notification; defer only its Discord
projection. Future work is [I18N BL 001](../backlog.md#i18n-bl-001-discord-notification-localization).

## Open decisions and validation

Select the backend/Python resource adapters with ADR 0002 and the server timezone
context with ADR 0003. Confirm which early HTML/static text is build-time `en-US`
and how future locale variants are selected without a dependency on external
translation downloads. Help routing remains in ADR 0001.

Verify main and standalone entry points, error/loading output, CSS replacement,
canvas/images/PDFs, notification text and email subject/body. Check speech text
and language selection with browser APIs stubbed, then perform an appropriate
manual accessibility review. Avoid real notification/email delivery in tests.
Confirm excluded Discord and external integration behavior remains unchanged.

## References

- [Main entry point](../../liwords-ui/src/index.tsx)
- [Embed entry point](../../liwords-ui/src/embed/embed-entry.tsx)
- [Embed build](../../liwords-ui/rsbuild.embed.config.ts)
- [Initial embed HTML](../../pkg/embed/service.go)
- [CSS-generated labels](../../liwords-ui/src/shared/gameLists.scss)
- [Canvas captions](../../liwords-ui/src/gameroom/board3d/scene.ts)
- [Speech helper](../../liwords-ui/src/utils/cwgame/blindfold.ts)
- [Browser notification helper](../../liwords-ui/src/utils/notifications.ts)
- [Moderation email rendering](../../pkg/mod/email.go)
- [League email rendering](../../pkg/league/league_notify.go)
- [PDF generator](../../services/tourneypdf/src/generator.py)
- [UI library migration](../../docs/mantine_migration_plan.md)
