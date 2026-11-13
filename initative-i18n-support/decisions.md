# Internationalization clarification decisions

These requirements guide the first iteration of internationalization. The
clarification answers are incorporated into [the draft feature specification](feature-specification.md).
Architectural recommendations remain proposals wherever an owner decision is
listed; completing clarification does not approve those implementation choices.

Only US English (`en-US`) is supported in production after the migration. The
structure must allow the community to add languages later.

Translation applies only to user-facing text. Generic machine-to-machine
messages from integrations that are not shown to a user are out of scope.

Store all files created for this task under `initative-i18n-support/` at the
repository root, including the feature specification and ADRs.

## 1. Application UI scope

All application-owned UI surfaces are in scope, including player-facing UI,
administration and director tools, legal and help pages, standalone embeds,
speech output, browser notifications, and generated images and reports.

### Help pages and existing links

- Mark existing help pages with the US English language code, `en-US`.
- Provide a basic structure for additional language versions and routing based
  on the user's i18n language setting.
- Preserve existing URLs and deep links after the migration. Existing links
  must continue to reach the corresponding US English content, including its
  sections, if locale-specific paths are introduced.
- A locale prefix such as `en_us` must not make the former unprefixed URL
  inaccessible. The example prefix does not fix the final URL convention.

The routing mechanism, redirect policy, and behavior when a requested language
version is missing remain to be specified. See
[the help routing ADR](adr/0001-localized-help-and-stable-urls.md).

## 2. Email and operational notifications

Transactional emails remain in scope, including verification, password reset,
league and moderation emails. Human-facing operational messages outside the
deferred Discord integration remain within the user-facing scope.

Discord integration localization is out of scope for this iteration. This
includes moderation, tournament, league and build/deployment notifications
posted through Discord webhooks. Track the future work in
[I18N BL 001](backlog.md#i18n-bl-001-discord-notification-localization).

Prepare in-scope messages for translation while shipping only `en-US` in the
first iteration. Transactional email follows the recipient policy in question
11 below.

This scope covers messages rendered for people. Integration acknowledgements,
worker protocol messages and other machine-to-machine text that users do not
see are excluded.

## 3. Managed content

Badge descriptions are in scope for translation. Keep stored badge codes and
image identities stable while providing translatable display text.

Announcements and organization titles are out of scope for content translation
in this iteration. Keep their existing content in its original language. The
surrounding application UI remains in scope for translation.

## 4. Account language preference

Save the preferred UI language to the user's account so it follows them across
devices. Browser-only storage is not sufficient. Only `en-US` is available in
production in the first iteration.

Use `en-US` whenever the user is not logged in, including before login and after
logout. A remembered browser preference must not override this signed-out
default. When a user logs in, use that account's saved language preference.

## 5. Experimental selector access

Leave this architectural decision open for the repository owner. The two
options are:

1. A dedicated `I18n Tester` role with an i18n-specific permission
   (recommended).
2. A developer-tools flag that reveals the selector in the current browser.

The selector remains hidden by default under either option. The recommendation
is not an accepted implementation choice. Do not ask the feature requester to
select the mechanism. See
[ADR 0005](adr/0005-experimental-language-selector-access.md).

## 6. Client reload and API compatibility

Older browser tabs must reload as part of the migration. Keeping those tabs
working indefinitely is not a release requirement. The current frontend-version
check only offers an optional refresh, so the rollout must address how the
required reload is enforced.

Classify APIs by their consumers and apply the following policy only to
user-facing text:

| Class | Consumers | Translation policy |
| --- | --- | --- |
| a | Internal UI only | Return structured message codes and translate when the UI uses them. |
| b | Internal UI and external clients | Translate on the backend before sending the user-facing response or message. Preserve machine-readable data and existing external contract shapes. |
| c | External clients only | Out of scope for this iteration. |

The class b backend rendering rule is an explicit exception to the general
frontend code-resolution approach. Machine-to-machine text that no user sees
remains out of scope in every class. Classify individual endpoints rather than
assuming every method in a service has the same consumers.

See [the classification](api-classification.md),
[the method inventory](api-classification.csv), and
[the API impact assessment](api-impact.md). Class a assignments based on source
inspection are provisional: lack of an external call site in this repository
does not prove the absence of deployed external clients. Methods without enough
caller evidence remain unclassified pending repository-owner review.

The exact browser reload mechanism, unverified classifications and backend
language-context details remain open in
[ADR 0006](adr/0006-client-reload-and-api-compatibility.md).

Question 6 is clarified at the requirement level. Classification verification,
backend language context and reload mechanics remain open for the repository
owner. Historical-message requirements are recorded in question 7 below.

## 7. Historical system messages

Keep existing message records unchanged in the datastore. Store structured
codes and parameters for new user-facing system messages after the migration.
Historical system messages must nevertheless be translated when displayed.
This does not authorize translating user-authored chat or hidden machine-only
messages.

Use a read-time compatibility adapter to recognize known historical system
message formats and obtain a message identity and parameters without rewriting
the stored record. Apply question 6's rendering policy: class a resolves the
translation in the UI; class b resolves it on the backend before transmission.
If a legacy system-message format cannot be recognized reliably, display its
original English text as the fallback. Do not discard the message or infer a
translation from arbitrary user-authored text.

The current code retrieves chat history by channel; it does not provide
full-text search over message bodies. Moderation searches/discovers channels
by user, channel type, IDs, addresses, and tournament/club/league names or slugs,
then retrieves the selected channel's messages. These operations must continue
to work. Existing moderation review excludes senderless system messages; keep
that behavior unless separately requested.

See [ADR 0007](adr/0007-historical-system-message-rendering.md) for the adapter,
retrieval/search evidence, and confirmed fallback policy.

## 8. Date time and number formatting

Format user-facing dates, times and numbers according to the selected UI
language. Do not independently choose formatting rules from the browser or
device's regional settings. In the first iteration, the only supported UI
language and formatting locale is `en-US`.

This decision selects display formatting rules. It does not change stored
timestamps, machine-readable date/number formats, game notation or external
protocol data.

Use the browser/device's local timezone as the default. Timezone selection is
independent of the UI language: changing the language changes display formats
without changing the timezone or the instant represented by a timestamp.

## 9. English wording corrections and audit

Preserve existing US English wording except for corrections of obvious
mistakes. Do not use the i18n migration for stylistic rewriting or broader
terminology changes.

Make every wording correction explicit in an audit log that must be reviewed
by the maintaining team. Record the original and corrected wording, the
location or message key, and the reason for the correction. Track review status
and the reviewer separately from the proposed correction; do not mark an entry
reviewed without a real maintaining-team review.

Use [the English copy audit log](english-copy-audit.md). No application wording
has been changed during this planning task.

## 10. Automated catalog checks

Automated catalog checks are required in the first iteration. Run them in CI
so changes to translation resources are validated before they are merged.

Checks must cover catalog syntax and duplicate keys, referenced message-key
coverage, interpolation parameters, plural requirements, and supported rich-text
structure. Validate the shipped `en-US` resources and the backend code-to-message
mapping. Prepare the checks to validate additional language catalogs when they
are introduced; production still supports only `en-US` after this migration.

The detailed contributor workflow and authority to enable a new language have
not been explicitly settled by this answer. Those governance details can be
documented as repository-owner decisions rather than assuming that a passing
catalog check alone authorizes a new production language.

## 11. Transactional email language

Render transactional emails in the recipient's saved UI language. Use `en-US`
when no recipient language preference is available. The recipient's preference,
not the sender's or triggering user's preference, selects the email language.
Only `en-US` is supported in production in the first iteration.

## 11a. Discord integration deferred

Discord integration localization is deferred to the i18n backlog. Do not add
destination language settings, change Discord notification templates or create
per-user Discord subscriptions as part of the first iteration. See
[I18N BL 001](backlog.md#i18n-bl-001-discord-notification-localization).

Current code uses deployment-configured Discord webhooks. The application
configuration supplies `DiscordToken` for moderation and tournament messages,
and `LeagueDiscordToken` for league messages. Build/deployment notifications
use the GitHub Actions secret `DISCORD_WEBHOOK`. The actual webhook destination
is configured outside the checked-in source.

No application-level per-user Discord destination selection or notification
subscription mechanism was found in the UI, API contracts or database schema.
The community Discord invite link in the UI is a separate link, not a
subscription or notification-routing setting.

These findings inform the backlog item. Future language selection for Discord
destinations remains a backlog decision and does not block this iteration.

Sources: [application configuration](../pkg/config/config.go),
[moderation notifications](../pkg/mod/notify.go),
[tournament notifications](../pkg/tournament/service.go),
[league notifications](../pkg/league/league_notify.go), and
[deployment workflow](../.github/workflows/build-and-deploy.yml).

## Open owner decisions

Clarification is complete for drafting the first iteration. Contributor
workflow details, language-release governance, help route mechanics, runtime
and catalog tooling, and selector access remain repository-owner decisions.
Classification verification, backend language context and the reload mechanism
also require implementation decisions. See [the ADR index](adr/README.md).
Discord destination language belongs to the backlog. No open choice is an
accepted architectural decision.
