# API consumer classification and translation boundaries

API translation depends on both the endpoint's consumers and the visibility of
each message. The consumer classes determine where user-facing application
messages are translated. Machine-to-machine text that no user sees is excluded
regardless of class. Only `en-US` ships in this iteration.

## Policy

| Class | Consumers | User-facing application text | Machine-only text |
| --- | --- | --- | --- |
| a | Internal UI only | Use structured codes and resolve the translation in the UI. | Out of scope. |
| b | Internal UI and external clients | Resolve the translation on the backend before sending the response or message. The UI displays that text without translating it again. | Out of scope; preserve the protocol. |
| c | External clients only | Out of scope for this iteration. | Out of scope. |

For this inventory, internal means the application-owned UI, including admin,
director and embedded UI. External means a client outside that UI, such as a
CLI, integration or worker consuming a network API; it does not imply that a
different organization owns the code. A backend implementation or shared
protobuf type alone is not evidence of an external API consumer.

Apply the classes per endpoint and the visibility restriction per field or
message. For example, a class b game-history response contains machine-readable
game data that must stay unchanged; only application-owned errors or other
human-facing text need backend translation. Dictionary content, user-authored
text, identifiers, enum values and game notation do not become UI translations.
Announcement content and organization titles retain their explicit exclusions.

## Method inventory and evidence limits

[The CSV inventory](api-classification.csv) covers all 253 declared RPC methods
across 23 services. It records the category, evidence status, translation policy,
source schema, and identified UI/non-UI call sites.

| Assignment | Methods | Evidence status |
| --- | --- | --- |
| a | 204 | UI source references found; no non-UI callers found by the inventory. Provisional until the owner confirms external usage. |
| b | 5 | Both UI and non-UI call sites found in this repository. |
| c | 6 | Non-UI caller found, or explicitly worker-facing contract; no UI caller found. |
| Not yet assigned | 38 | No sufficient caller evidence found. Owner review required before changing the contract. |

Unassigned is a review status, not a fourth consumer class. The inventory does
not assign unused, indirect or unknown endpoints to class c merely because no
UI reference was found. Caller detection uses source patterns for generated
query descriptors and client method calls, plus manual checks for dynamic
method selection; it is not a runtime traffic audit or a complete call graph.
Generated client definitions and test fixtures are not treated as usage.

Class a candidates span the account/settings UI, roles and permissions UI,
organization operations, social/chat UI, collection/comment tools, player
analysis, broadcast management, league management, puzzles and tournament
management. The CSV identifies individual methods; these groups are not
blanket classifications of every method in a service.

## Class b Confirmed shared endpoints

RPC paths use `/api/<package>.<Service>/<Method>`.

| Endpoint | UI evidence | Non-UI evidence | Backend translation target |
| --- | --- | --- | --- |
| `tournament_service.TournamentService/GetTournament` | [Tournament UI](../liwords-ui/src/tournament/utils.ts) | [COP CLI](../cmd/coprunner/main.go), [PDF service](../services/tourneypdf/src/fetch_tourney.py) | Application errors or human-facing system messages; preserve tournament data. |
| `tournament_service.TournamentService/GetTournamentMetadata` | [Tournament UI](../liwords-ui/src/tournament/utils.ts), editor and wizard | [PDF service](../services/tourneypdf/src/fetch_tourney.py) | Application errors; preserve metadata, names and other content. |
| `game_service.GameMetadataService/GetGameHistory` | [Embedded viewer](../liwords-ui/src/embed/embed.tsx) | [Image CLI](../cmd/tiles/main.go) | Application errors; preserve history and game notation. |
| `omgwords_service.GameEventService/GetGameDocument` | [Editor](../liwords-ui/src/boardwizard/editor.tsx), game room | [Game-document CLI](../cmd/cwgame/main.go) | Application errors; preserve document data. |
| `user_service.RegistrationService/Register` | [Registration form](../liwords-ui/src/lobby/register.tsx) | [Batch registration](../scripts/batchregister/register.py), [user script](../scripts/utilities/register-user.sh), [bot script](../scripts/utilities/register-bot.sh) | Human-facing validation, error and outcome text where emitted; preserve fields and machine semantics. |

The backend must use a defined language context for class b rendering. Missing
language context defaults to `en-US`; signed-out UI uses `en-US` by confirmed
requirement. How future external callers select a language remains an owner
decision. API authentication method alone does not establish caller purpose.

Use existing stable status/error fields for control flow. If further semantic
identity is needed, design an additive mechanism without replacing an external
text field with a raw code. Translated text must not become a condition in UI
or backend logic. Language-dependent caching also needs an explicit policy.

## Class c RPCs

| Endpoint or group | Evidence | Scope |
| --- | --- | --- |
| `analysis_service.AnalysisQueueService/ClaimJob`, `Heartbeat`, `SubmitResult`, `FailJob` | [Worker-facing contract](../api/proto/analysis_service/analysis_service.proto); no authored UI calls found | Worker protocol and hidden failure diagnostics are excluded. Translate a user-visible outcome later at the in-scope player/admin boundary if it is shown there. |
| `pair_service.PairService/HandlePairRequest` | [COP CLI caller](../cmd/coprunner/main.go); no authored UI call found | External pairing RPC and its existing output are excluded. This does not exclude labels or error presentation in the director UI. |
| `config_service.ConfigService/SetFEHash` | [Deployment caller](../.github/workflows/build-and-deploy.yml); no authored UI call found | Deployment protocol is excluded. Its frontend version value can still support the separate browser reload requirement. |

`TournamentService.RunCOP` shares `ipc.PairResponse`, but no frontend or external
network caller was found in this inventory. It remains unassigned rather than
being inferred to share another method's class.

## Non-RPC surfaces

| Surface | Current classification | Translation scope |
| --- | --- | --- |
| Browser websocket `/ws` | a candidate; no external websocket client established by this inventory | Translate user-facing errors and system events in the UI. Hidden routing, presence and control messages remain machine data. |
| `/api/game/eventstream/<gameId>` | c by intended API-client use; no main-UI consumer found | Preserve the external NDJSON surface. Changes to shared event schemas must not inadvertently change its rendering behavior. |
| `/api/broadcasts/obs/...`, `/api/annotations/obs/game/...`, `/api/annotations/obs/user/...` | c by OBS-client purpose; broadcast SSE also has a [Python consumer](../tools/obs/woogles_text_sources.py) | External OBS outputs are excluded. The [URL builder](../liwords-ui/src/broadcasts/OBSPanel.tsx) and [local sample preview](../liwords-ui/src/broadcasts/OBSFieldPreview.tsx) are application UI and remain in scope. |
| Application-owned embed HTML and `/gameimg/...` | a candidates for their owned rendering UI; owner to verify other API consumers | Help/loading text and generated-image labels remain within the previously agreed UI scope; game data stays unchanged. |
| OAuth browser callback/error pages | a candidates for their human-facing rendering | Translate what the person sees. External OAuth protocol fields, webhook acknowledgements and hidden integration errors are excluded. |
| Machine-only webhooks and health/control responses | c or internal system plumbing, depending on caller | Excluded by the user-visibility restriction in either case. |

Serving the application's embedded viewer inside another website does not alone
make its own UI text an external API contract. Conversely, a web UI that builds
an OBS URL does not establish that the main frontend consumes the external OBS
stream.

## Implications for implementation

Create application message identities at the source of user-facing outcomes,
then choose the renderer at the endpoint boundary. Class a transmits codes to
the UI. Class b renders text on the backend before transmission. Class c and
hidden machine-only messages remain unchanged.

Do not translate shared event-bus objects globally or persist a viewer-specific
translation as the sole form of an event. A single event may later feed a class
a UI path, a class b response and an excluded class c integration. Its projections
must respect those different boundaries. Existing records stay unchanged, while
known historical system messages are normalized for translated display; new
records store structured codes. See
[ADR 0007](adr/0007-historical-system-message-rendering.md).

This policy supersedes the earlier possibility of excluding every endpoint
that has any external consumer. Class b is now explicitly included for backend
translation of user-facing text. [ADR 0006](adr/0006-client-reload-and-api-compatibility.md)
records the policy and the remaining implementation decisions.
