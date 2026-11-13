# ADR 0006 Client reload and API compatibility

Status: Open for implementation details. The browser reload requirement and
API translation policy are confirmed; classification verification, backend
language context and the rollout mechanism remain open.

## Context

The migration introduces stable message codes and parameters for user-facing
application text. Internal UI-only endpoints can send codes to the UI; endpoints
shared with external clients must render their user-facing text on the backend
before sending it. Existing clients can depend on text in RPC responses, errors,
websocket events and stored system messages.

Older browser tabs must reload. The existing version check retrieves the
frontend hash with `AuthenticationService.GetSocketToken`, offers an optional
refresh if it differs, and still returns a websocket connection URL. It does
not enforce the required upgrade for every already-open tab or RPC client.

Analysis workers, the COP Lambda, OBS sources, scripts and third-party API
clients cannot all be upgraded by refreshing the web application. Their
potential contracts are listed in [the API impact assessment](../api-impact.md).

## Confirmed requirement

Require older browser tabs to reload for the migration. Maintaining continued
operation of old tabs is not a release requirement. This does not by itself
approve breaking non-browser consumers or deleting historical data.

Apply this consumer policy to individual endpoints:

| Class | Consumers | Policy for user-facing text |
| --- | --- | --- |
| a | Internal UI only | Translate codes in the UI when used. |
| b | Internal UI and external clients | Translate on the backend before sending the response or message. |
| c | External clients only | Out of scope for this iteration. |

Generic machine-to-machine messages from integrations that users do not see are
out of scope in every class. Preserve machine-readable data, identifiers,
transport statuses and control semantics. Translate only the human-facing
portions of an in-scope response.

Confirmed class b endpoints cover tournament data and metadata, game history,
game documents, and registration. See [the classification](../api-classification.md)
and [method inventory](../api-classification.csv). Source-based class a
assignments need owner verification because unknown external callers are not
visible in this repository. Methods without sufficient caller evidence remain
unassigned pending review.

Question 6 is clarified at the requirement level; the repository owner must
resolve the implementation details below. Historical storage and display
requirements are recorded separately in
[ADR 0007](0007-historical-system-message-rendering.md).

## Proposed direction

Use explicit client compatibility/version handling and a coordinated rollout.
Keep application message reasons separate from transport statuses. Evolve
protobuf contracts without reusing existing field or enum numbers.

Class b backend rendering is the selected boundary for shared endpoints; it is
an explicit exception to frontend message resolution. The UI displays the
rendered text without translating it again. Preserve the external contract's
shape and machine semantics. Keep existing codes/status fields available for
logic; if additional application identity is needed, use a compatible additive
mechanism rather than changing a text field into a raw code.

Do not infer endpoint purpose solely from cookies, API keys, or browser headers.
Use an explicit per-endpoint classification and a defined language context.
Hidden integration diagnostics remain unchanged even when an associated
user-visible error is rendered at an in-scope boundary.

The implementation must explain how an old bundle that lacks the new reload
handling is reached, how current connections and RPC calls are handled, and
how reload loops or mixed frontend/backend versions are avoided. The existing
optional refresh notice alone does not meet the confirmed requirement.

## Decisions still needed

- Minimum compatible client identification and enforcement mechanism.
- Deployment order and any required preparatory release.
- Reload behavior during an active game or with unsent edits.
- Rollback behavior when one component has already changed contracts.
- Verification of provisional classifications and assignment of methods with
  insufficient caller evidence.
- Backend language context for class b, including future external callers and
  language-dependent caching. Only `en-US` ships initially.
- Compatible structured identity for any class b outcome where existing status
  fields are insufficient for UI control flow.
- Treatment of legacy transport fields in conjunction with ADR 0007's confirmed
  historical storage and display requirements.

These mechanisms remain proposals until the repository owner selects the
release strategy. Requiring a browser reload does not require repurposing old
string fields or changing successful machine-data responses unnecessarily.

## Validation

Exercise an already-open pre-migration tab, a newly loaded tab, RPC calls made
without a websocket, reconnects, installed PWAs, embedded views, and rollback.
Confirm that incompatible tabs cannot continue consuming messages they cannot
interpret and can recover by loading a compatible frontend. Validate affected
class b contracts independently to ensure backend translation preserves their
shape and machine semantics. Confirm class c contracts and hidden machine-only
messages remain unchanged, and verify that class b text is not translated again
in the UI.

## References

- [Clarification decisions](../decisions.md)
- [API impact assessment](../api-impact.md)
- [Consumer classification](../api-classification.md)
- [Method inventory](../api-classification.csv)
- [Frontend version comparison](../../liwords-ui/src/socket/socket.ts)
- [Current refresh action](../../liwords-ui/src/socket/reload.tsx)
- [Socket token response](../../api/proto/user_service/user_service.proto)
- [Published frontend hash](../../pkg/config/service.go)
- [Protobuf evolution guidance](https://protobuf.dev/best-practices/dos-donts/)
