# Basic internationalization support

Status: Draft for maintaining-team review.

The first iteration makes application-owned, user-facing text translatable while
shipping only US English (`en-US`). It covers the application UI, owned rendering
surfaces and transactional emails. Community languages can be added later.

Confirmed product requirements come from [the clarification decisions](decisions.md).
Implementation recommendations are recorded in [the ADRs](adr/README.md). An open
ADR is not an approved architectural choice; in particular, the repository owner
must decide how testers reveal the language selector.

## Outcomes

1. All in-scope application text has a defined translation source and renderer.
2. Existing US English wording is preserved except for obvious mistakes listed
   in a correction audit reviewed by the maintaining team.
3. User-facing backend outcomes have stable identities and parameters; their
   rendering boundary follows the API consumer classification.
4. Language is an account preference used across devices. Signed-out users use
   `en-US`. A hidden experimental selector exposes only supported languages.
5. Existing help URLs and historical messages remain usable after migration.
6. Automated catalog checks prevent invalid resources and unmapped messages.

## Scope

| Surface | First-iteration requirement |
| --- | --- |
| Main player UI | Navigation, account forms, lobby, game room, chat, puzzles, analysis, leagues, tournaments, broadcasts and collections are covered. |
| Staff UI | Administration, moderation and director tools are covered. |
| Application text | Buttons, headings, form validation, tooltips, placeholders, empty/loading/error states, confirmations, notifications, accessibility text and document titles are covered. |
| Static and help content | Existing legal, about and help content is prepared for translation. Help content has `en-US` metadata and locale-aware resolution. |
| Other owned rendering | Standalone embeds, generated images/reports, canvas captions, speech and browser notifications are covered where they are application-owned UI rather than an excluded external-only API contract. |
| Badge content | Badge descriptions and surrounding UI/accessibility copy are covered; badge codes and image identities stay stable. |
| Transactional email | Verification, password reset, league and moderation emails use the recipient's language with `en-US` fallback. |
| Historical system messages | Existing records stay unchanged but known system formats are translated for display. |

Only user-facing text is a translation target. Generic integration messages and
machine-to-machine text that users do not see are excluded in every category.

Explicit exclusions are:

- Additional supported production languages in this iteration.
- Translation of announcement content and organization titles; their surrounding
  application UI remains covered.
- Automatic translation of user-authored chat, biographies, names, comments or
  other user content, and translation of dictionary/lexicon data as UI copy.
- Translation of external-only API responses and protocols, including external
  OBS output. The application OBS URL builder and its local preview remain UI.
- Discord integration localization, notification-template changes and destination
  language settings. See [I18N BL 001](backlog.md#i18n-bl-001-discord-notification-localization).
- Stylistic rewriting and general terminology cleanup.
- Changes to game rules, alphabets, tile notation, persisted timestamps or
  machine-readable number/date formats caused solely by UI language.

These explicit exclusions govern narrower cases within the general UI scope.
Serving an owned embed on another website does not itself exclude its UI text.

## Language preference and selector

FR 01: Use `en-US` as the supported language identifier and fallback. The
production supported-language registry contains only `en-US` after migration.
Do not enable languages from browser detection or from the mere presence of a
catalog file.

FR 02: Persist the preferred UI language privately on the account. Existing
accounts without a value resolve to `en-US`. An unavailable or invalid stored
value falls back safely to `en-US`; writes accept only supported values.

FR 03: After login, apply the current account's saved language on that device.
After logout, return to `en-US`. Switching accounts must not carry the previous
account's language into the next account. Cross-device use requires persistence
and loading on login/refetch; continuous real-time synchronization is not a
requirement of this iteration.

FR 04: Add a language selector to the existing experimental/Secret features
settings section, hidden by default. When revealed, it lists only English
(United States) in production. Hiding the selector does not disable the i18n
runtime or make the language preference browser-only.

The repository owner must choose between a dedicated `I18n Tester` role
(recommended) and a developer-tools flag. Neither is selected by this spec.
See [ADR 0005](adr/0005-experimental-language-selector-access.md).

## Messages and rendering boundaries

FR 05: Use stable application message identities and named, validated parameters.
Retain numbers, timestamps, units and enums as data until formatting is required.
Preserve existing error-code identities; do not reuse protobuf field or enum
numbers. A localized sentence must never determine application control flow.

FR 06: Apply the API policy per endpoint, and the user-visibility rule per message
or field:

| Class | Consumers | Required behavior |
| --- | --- | --- |
| a | Internal UI only | Backend produces a code and parameters for user-facing outcomes; the UI resolves the translation when used. |
| b | Internal UI and external clients | Backend translates user-facing response/message text before transmission. The UI displays that text directly. Preserve existing contract shapes and machine semantics. |
| c | External clients only | Translation is out of scope. Preserve the existing protocol. |

Class b is an explicit exception to frontend code resolution. Machine-only
fields are not translated even on an in-scope endpoint. Shared internal events
must support appropriate projections without globally mutating messages that
also feed excluded integrations.

The [inventory](api-classification.csv) covers 253 RPC methods: 204 provisional
class a, five confirmed shared methods, six class c, and 38 unassigned pending
caller review. Source inspection is not proof that an external caller does not
exist. Verify classifications before changing each affected contract.

FR 07: Cover failures, warnings and conditional success messages. Reuse an
existing structured status/count where it already fully identifies an outcome.
Normalize lower-level errors, including Lua/WASM failures, at an in-scope
application boundary without making hidden diagnostics a translation target.

FR 08: Replace existing English-text conditions in login, email verification,
cookie handover and puzzle behavior with stable semantic conditions. Resolve
codes at display time, including callbacks and code outside React components.
Do not translate rendered class b text a second time.

FR 09: Unknown or malformed new application messages produce a readable generic
`en-US` fallback, not a raw key/code, missing placeholder or internal diagnostic.
The historical-message exception below may retain original English content.

## Catalogs and formatting

FR 10: Maintain complete `en-US` resources, separated into practical feature
namespaces. Use semantic keys, complete sentences, named parameters and proper
plural forms. Preserve React/link structure safely without arbitrary translated
HTML. Keep translators' context separate from public code identities.

FR 11: Supply resources to all in-scope runtimes and entry points. UI catalogs
alone are insufficient for Go-rendered class b messages and emails, generated
outputs or the separate embed build. Define an authoritative source for each
message and validate any resource generation between runtimes.

FR 12: Format human-facing dates, times and numbers according to the selected UI
language, which is `en-US` in this iteration. Do not independently derive those
formats from browser regional settings. Coordinate component-library locale
text and date-library formatting with the same selection.

FR 13: Keep timezone independent of UI language. The browser/device's local
timezone is the default for UI display. A language change must not change the
underlying instant, persisted timestamp or timezone. Preserve explicitly
identified timezone contexts; server-rendered output without a device context
needs a documented timezone policy rather than an inferred US timezone.

Game lexicon selection, alphabet behavior, board coordinates and machine-readable
exports remain independent. Human-facing game descriptions still use catalogs.

## Help content and URL compatibility

FR 14: Mark existing help pages as `en-US` and support resolution of future
language variants based on the user's i18n setting. Preserve stable manual and
section identities independently of translated titles or source-file layout.

FR 15: Existing `/docs`, manual URLs and section deep links continue working after
migration. If locale prefixes such as `en_us` are introduced, retain the former
unprefixed URLs and access to the corresponding US English content. Do not drop
manual IDs, section IDs, meaningful query parameters or fragments in redirects.
The example prefix does not prescribe a final path spelling.

The owner chooses routing mechanics and future explicit-locale/preference
precedence in [ADR 0001](adr/0001-localized-help-and-stable-urls.md). Only the
`en-US` version is shipped now; development fixtures can verify locale resolution
without adding another supported production language.

## Historical messages and retrieval

FR 16: Leave existing datastore records unchanged. Store structured identities
and parameters for new user-facing system messages after migration. Do not store
only the current viewer's rendered language as the canonical new system record.

FR 17: Recognize known legacy system-message formats at read time and normalize
them for translation at the class a/class b boundary. Use system-origin metadata;
a player's message that copies a system sentence remains user-authored content.
If the legacy format cannot be recognized reliably, display its original English.

FR 18: Preserve message IDs, order, timestamps, retention, deletion handling,
previews and reconnect catch-up. Current chat has channel-based retrieval and
moderation channel discovery, without message-body full-text search. Preserve
those operations and moderation's existing exclusion of senderless system lines.
No new full-text search capability is required. See
[ADR 0007](adr/0007-historical-system-message-rendering.md).

## Email and deferred integration

FR 19: Render transactional email using the recipient's saved UI language,
falling back to `en-US` when no preference is available. Do not inherit the
sender's or triggering user's locale. Queue/background rendering must resolve
the recipient context without depending on a live browser session.

FR 20: Keep Discord localization outside this iteration. Retain its existing
destinations, templates and delivery behavior; future destination language
selection belongs to [the backlog](backlog.md). Human-facing UI and emails that
share an underlying event remain within their own scope.

## English copy and catalog checks

FR 21: Preserve the existing English except for obvious mistakes. Each wording
correction must include the exact before/after, location/key and reason in
[the copy audit](english-copy-audit.md), and be reviewed by the maintaining team.
Moving unchanged text into catalogs is not a correction; broader rewriting is
outside scope.

FR 22: Run automated catalog checks in CI. Validate syntax, duplicate keys,
referenced-key coverage, code-to-message coverage, named parameters, plurals and
supported rich-text structure. Validate each supported rendering resource and
any generated equivalents. Source changes must not silently leave `en-US`
incomplete or ship development-only locales.

Detailed community workflow, translation review ownership and future language
release criteria remain owner decisions. Passing automated checks alone does
not enable another production language. See
[ADR 0008](adr/0008-catalog-checks-and-copy-review.md).

## First-iteration delivery plan

| Step | Reviewable result |
| --- | --- |
| 1. Contracts and inventory | Owner-reviewed API classifications; chosen message schema, runtime/catalog approach, private preference API and rollout strategy. |
| 2. Foundation | Bundled `en-US` resources, typed message resolution, backend renderer, formatting adapters, preference persistence and catalog CI. |
| 3. End-to-end slice | Verification flow, a gameplay rejection, a conditional analysis outcome, shared registration backend rendering, old/new challenge chat, and the gated selector. |
| 4. Domain migration | Player UI, staff/director UI, secondary entry points, help/static text, badge descriptions, generated outputs and transactional emails. |
| 5. Completion review | Coverage inventory closed, copy corrections reviewed, legacy URLs/history verified and browser reload rollout rehearsed. |

The slice is a checkpoint, not completion of the iteration. Track extraction and
review by surface, including CSS-generated text, canvas, speech and notification
helpers that ordinary JSX string scanning misses. Coordinate provider and UI
changes with the repository's ongoing Ant Design/Mantine migration.

## Acceptance criteria

| ID | Required evidence |
| --- | --- |
| AC 01 | Production lists, loads and accepts only `en-US`; development fixtures are not shipped as supported languages. |
| AC 02 | Existing and new accounts resolve safely, a saved preference is loaded across devices, and logout/account switching do not leak another account's locale. |
| AC 03 | The selector is hidden for default users and revealable through the owner's chosen mechanism; account persistence is independent of that gate. |
| AC 04 | Every in-scope rendering surface has a reviewed translation source or a documented content/protocol exclusion. Hidden integration text, class c APIs, announcement content, organization titles and Discord are excluded. |
| AC 05 | Representative class a outcomes display from codes, class b outcomes arrive as backend-rendered text, and neither path uses translated wording for behavior. |
| AC 06 | Invalid/unknown new codes and missing parameters degrade to readable output; approved legacy fallback retains original English. |
| AC 07 | Dates, times and numbers use `en-US` formatting while device timezone remains independent. Stored timestamps, game notation and external machine data are unchanged. |
| AC 08 | Existing help index/manual/section URLs, bookmarks and relevant query/fragment links remain usable after source-file or route changes. |
| AC 09 | Mixed historical/new system messages display correctly without rewriting old records; user-authored lookalikes, previews, catch-up and moderation discovery remain correct. |
| AC 10 | Emails use recipient language/fallback without a live UI session; Discord templates and delivery are not migrated. |
| AC 11 | CI rejects representative invalid catalogs, missing keys/mappings, parameter/plural/markup errors and unsupported production locale registration. |
| AC 12 | Every source-English wording correction has a reviewed audit entry; remaining copy changes are extraction/formatting work with unchanged meaning. |
| AC 13 | A development-only pseudolocale or equivalent fixture demonstrates interpolation, plural behavior, text expansion and alternate entry-point coverage without claiming support for another production language. |
| AC 14 | The rollout handles an already-open old tab, reconnect, PWA/embed and rollback according to the owner's reload strategy, without breaking protected external machine contracts. |

Use existing TypeScript/build/lint, Vitest and relevant Go checks alongside
targeted contract and browser tests. Prefer tests that expose boundary failures,
old records or state leakage over snapshots that merely repeat the English
catalog. Include a visual review of dense game/director UI and non-React output.

## Decisions needed from the maintaining team

| Decision | Record |
| --- | --- |
| Help route convention, explicit-locale links and future fallback resolution | [ADR 0001](adr/0001-localized-help-and-stable-urls.md) |
| Runtime libraries, source catalogs, generation and structured message schema | [ADR 0002](adr/0002-message-contracts-and-catalog-runtimes.md) |
| Private preference contract and server-side locale/timezone context | [ADR 0003](adr/0003-account-locale-and-formatting.md) |
| Secondary renderer adapters and email/background language context | [ADR 0004](adr/0004-secondary-renderers-and-email.md) |
| Dedicated i18n tester role versus developer-tools flag; explicitly owner-only | [ADR 0005](adr/0005-experimental-language-selector-access.md) |
| Classification verification, browser reload enforcement and rollout order | [ADR 0006](adr/0006-client-reload-and-api-compatibility.md) |
| Exact legacy adapter/storage schema and coverage of deployed old formats | [ADR 0007](adr/0007-historical-system-message-rendering.md) |
| Catalog tooling and future contributor/language-release governance | [ADR 0008](adr/0008-catalog-checks-and-copy-review.md) |

These decisions do not reopen the confirmed user requirements. Resolve the
relevant owner choice before implementing that boundary; keep unrelated work
reviewable while a decision remains open.
