# ADR 0002 Message contracts and catalog runtimes

Status: Proposed. API rendering rules are confirmed; runtime libraries, catalog
format and exact message schema require repository-owner selection.

## Context

The frontend has no global translation runtime. Existing `WooglesError` values
are serialized as `;numericCode;arg...`, then substituted into English templates
using positional `$1` parameters. Semicolons in arguments cannot round-trip
reliably. Some outcomes already have structured enums; other errors, warnings
and success responses expose English text directly.

React is not the only renderer. Class b APIs require backend translation; email
and generated outputs also run outside the main React root. A frontend-only
catalog migration would leave those requirements unmet. Transport statuses,
application message identities and presentation keys are separate concepts.

## Confirmed constraints

- Class a sends structured identities for UI translation. Class b renders
  user-facing text on the backend. Class c and hidden machine-only text remain
  outside scope. The policy applies per endpoint and per field.
- Only `en-US` is enabled initially; the community can add languages later.
- Existing code/enum identities and machine contracts must not be repurposed.
- Application behavior must use semantic codes or statuses, not English or
  translated sentences.

## Proposed approach

Introduce a shared protobuf application-message descriptor with a stable code
and named parameters. Define the allowed parameters and their types per code;
do not carry preformatted dates, counts or enum labels when the renderer needs
the underlying data. Keep transport failure classification alongside this
descriptor. Preserve existing numeric identities through an explicit mapping.

Use Connect error details for class a failures, an appropriate structured field
for class a success/warning responses and websocket events, and the same
logical identity internally before class b rendering. The pinned Connect Go
v1.20.0 already exposes `NewErrorDetail` and `Error.AddDetail`; no Connect major
upgrade is required to evaluate that approach. Do not copy incompatible v2
signatures into this migration.

Normalize underlying failures at their source or an explicit boundary, including
Lua validation results and WASM errors exposed in UI. Avoid a global error-helper
change that accidentally affects class c or unclassified consumers. Reuse
existing statuses instead of inventing a message code when they already express
the complete outcome.

Recommend `i18next` and `react-i18next` for the React runtime, with typed semantic
keys, feature namespaces, complete sentences, named interpolation and plural
forms. Bundle the complete `en-US` resources, including resources needed by
startup and the embed build, instead of adding a translation download dependency
for the first release. Configure library locale text separately.

Evaluate an established Go renderer such as `go-i18n` for class b and email.
Its template/resource syntax is not the same as i18next's; sharing message IDs
does not make resource files interchangeable. Before the domain migration,
select authoritative resource ownership and prove generation or validation
across the chosen runtimes. Keep frontend-only rich React messages separate
from templates rendered as server text or email HTML where necessary. Do not
build an ad hoc plural or template engine to hide incompatibilities.

## Compatibility slice and decisions still open

Prove one parameterized message, plural variants, safe rich text, missing-key
fallback and a class b response using the actual React/TypeScript, Go and build
versions in this repository. Confirm output consistency for shared messages.

The owner must select the code/parameter schema, frontend/backend libraries,
authoritative catalog layout, supported template features and any generation
tools. Python-generated reports also need a resource/rendering boundary under
ADR 0004. Pin the selected tool versions to make generated output reviewable.

Keep a narrow adapter for existing delimiter-encoded errors during the migration
instead of extending that protocol. Legacy stored sentences have a separate
presentation adapter in ADR 0007. Unknown new codes produce a readable generic
fallback, with technical diagnostics kept separate.

## Alternatives and consequences

Continuing the current positional English map would not provide plural handling,
safe parameter typing or the required backend rendering. Independent catalogs
without shared identity/parameter validation risk drift. A single richer message
format across all runtimes is an alternative only after its renderer support
and build integration are demonstrated.

The proposed approach adds catalog and contract maintenance, but makes message
identity independent of wording and lets each required consumer render at the
correct boundary. No AI translation or runtime translation service is required.

## References

- [Feature specification](../feature-specification.md)
- [API policy](0006-client-reload-and-api-compatibility.md)
- [Existing domain error encoding](../../pkg/entity/error.go)
- [Frontend parser](../../liwords-ui/src/utils/parse_woogles_error.ts)
- [English error map](../../liwords-ui/src/store/constants.ts)
- [Common transport errors](../../pkg/apiserver/errors.go)
- [Code generation](../../api/buf.gen.yaml)
- [React integration](https://react.i18next.com/latest/using-with-hooks)
- [Typed catalog keys](https://www.i18next.com/overview/typescript)
- [Go renderer capabilities and syntax](https://github.com/nicksnyder/go-i18n)
- [Pinned Connect Go API](https://pkg.go.dev/connectrpc.com/connect@v1.20.0)
