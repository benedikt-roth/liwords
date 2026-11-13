# ADR 0003 Account locale and formatting

Status: Proposed implementation. Account persistence, signed-out language,
formatting locale and independent timezone behavior are confirmed.

## Context

Current experimental preferences mostly use local storage. The private profile
API and profile model have no language field. Formatting mixes browser defaults,
explicit `en-US`, Moment and Day.js. Some formatter instances and option lists
are created outside component render, so changing only a React provider would
not reliably update their output.

## Confirmed requirements

1. Save UI language privately on the account and use it across devices.
2. Use `en-US` when signed out, including after logout; apply the next account's
   own preference after login.
3. Enable only `en-US` initially and use it when an account has no preference.
4. Date, time and number formats follow selected UI language.
5. Timezone is independent of language, defaulting to the browser/device's local
   timezone for UI display.
6. The selector's visibility mechanism is separate from preference persistence
   and remains open in ADR 0005.

## Proposed approach

Add a private account preference through a database migration and a private
read/write API. Extending personal-info methods or using a dedicated preference
contract are alternatives for owner review. Do not add the preference to public
profiles or brief-profile payloads solely to initialize the user's own UI.

Use one application locale resolver. Existing/missing/unavailable values resolve
to `en-US`; writes validate against the production supported-language registry.
Load the preference on authentication/refetch, reset it on logout, and invalidate
account-dependent caches when switching users. A client cache can improve
startup but is not the authoritative preference and cannot override signed-out
`en-US`. Real-time synchronization of already-open devices is not required.

Centralize human-facing formatting with explicit locale and timezone inputs.
Coordinate Ant Design locale text, current Mantine components and date-library
formatters. Replace browser-default or module-static formatters where they
conflict with selected UI language; avoid a wholesale date-library replacement
as incidental migration work.

Keep serialized timestamps, ISO birthday values and external machine fields
unchanged. Preserve game clock meaning, precision and game notation. Explicit
timezone labels in staff/reporting views need deliberate treatment rather than
being silently changed by a locale switch.

## Backend context and open decisions

Class b rendering needs the selected account locale before sending user-facing
text. Signed-out UI and absent language context resolve to `en-US`. Future
external-caller locale negotiation, cache variation and safe request context
propagation remain owner choices; API-key versus cookie authentication is not
a substitute for an explicit contract.

Email uses the recipient's saved UI language, even without a browser session.
A device timezone is unavailable in some server/report/email contexts. The
owner must select the required timezone context or preserve an explicitly
identified existing timezone there; never infer a US timezone from `en-US`.
This feature does not by itself require a new account timezone setting.

The owner also selects the private storage/API shape and refresh behavior when
tester access is revoked. Resource loading and locale switching must not leave
old translated state or another account's preferences visible.

## Validation

Exercise an existing account without a preference, two accounts in one browser,
logout, another device/login and invalid or unavailable stored values. Verify
that only supported locales can be written and that the selector gate does not
make persistence local-only. Check date/number output with explicit locale and
timezone fixtures, including a daylight-saving boundary; do not change the
represented instant or machine serialization.

## References

- [Clarification decisions 4 and 8](../decisions.md)
- [Private profile contract](../../api/proto/user_service/user_service.proto)
- [Profile model](../../pkg/entity/user.go)
- [Profile persistence queries](../../db/queries/profiles.sql)
- [Date/time helper](../../liwords-ui/src/utils/datetime.ts)
- [Current preferences](../../liwords-ui/src/settings/preferences.tsx)
- [Selector access](0005-experimental-language-selector-access.md)
