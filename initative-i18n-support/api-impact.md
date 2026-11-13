# Internationalization API impact

Internationalization affects user-facing RPC errors, several successful RPC
responses, websocket events and stored system messages. Internal UI-only APIs
(class a) send codes for the UI to translate. Shared UI/external APIs (class b)
translate user-facing text on the backend before transmission. External-only
APIs (class c) and machine-to-machine messages that users do not see are out of
scope. Older browser tabs must reload.

The accepted policy and the evidence-based endpoint assignments are documented
in [the classification](api-classification.md) and
[the method inventory](api-classification.csv). Class a assignments are
provisional until external usage is confirmed; 38 methods lack enough caller
evidence to assign a class. This assessment describes potential text-bearing
contracts, not permission to migrate unclassified methods.

The inspected protobuf definitions declare 23 services and 253 RPC methods,
registered under `/api/`. This is an inventory of possible impact, not a list of
approved API changes. Shared external contracts constrain the error-boundary
audit as well as successful responses.
The concrete payload candidates below are identified from the current schemas
and rendering code. Unknown external consumers cannot be enumerated from this
repository.

## Confirmed UI and non-browser overlap

The following endpoints have both UI callers and network clients outside the
main frontend in this repository. The non-browser callers are repository tools
or services; their presence proves shared use, not third-party ownership or
current production usage. There is no complete external-consumer registry in
the inspected source.

| Shared endpoint | UI caller | Non-browser caller | Relevance to i18n |
| --- | --- | --- | --- |
| `tournament_service.TournamentService.GetTournament` | `liwords-ui/src/tournament/utils.ts` | `cmd/coprunner/main.go`; `services/tourneypdf/src/fetch_tourney.py` | Class b: backend-render user-facing errors or messages; preserve tournament data. |
| `tournament_service.TournamentService.GetTournamentMetadata` | `liwords-ui/src/tournament/utils.ts`, admin editor and tournament wizard | `services/tourneypdf/src/fetch_tourney.py` | Class b: backend-render user-facing errors; preserve metadata and content. |
| `game_service.GameMetadataService.GetGameHistory` | `liwords-ui/src/embed/embed.tsx` | `cmd/tiles/main.go` | Class b: backend-render user-facing errors; preserve machine-readable history. |
| `omgwords_service.GameEventService.GetGameDocument` | `liwords-ui/src/boardwizard/gcg_process_form.tsx`, editor and game room | `cmd/cwgame/main.go` | Class b: backend-render user-facing errors; preserve document schema and game notation. |
| `user_service.RegistrationService.Register` | `liwords-ui/src/lobby/register.tsx` | `scripts/utilities/register-user.sh`, `register-bot.sh`, and `scripts/batchregister/register.py` | Class b: backend-render human-facing validation and outcome messages; preserve the external contract shape. |

Additional authenticated RPCs can be called using session cookies or an API key
through the same `AuthUser` helper. Public read RPCs and registration may need no
API key. Authentication method therefore is not a reliable way to classify all
external versus UI callers. Do not use cookie presence or a browser user agent
as implicit authorization to change the response contract.

### Shared schemas and linked endpoints are different cases

- `AnalysisQueueService` is a worker-facing API. No direct caller was found in
  the authored frontend source. The UI uses `AnalysisService`; they still share
  persisted analysis state, so adapt failures for UI display without requiring
  a worker API migration.
- `PairService.HandlePairRequest` is called by `cmd/coprunner`. It and the COP
  Lambda use `ipc.PairResponse`. No direct call to `HandlePairRequest` or
  `TournamentService.RunCOP` was found in the authored frontend source. Shared
  protobuf or pairing-library use must not be described as a confirmed shared
  UI endpoint.
- The OBS Python script consumes `/api/broadcasts/obs/<slug>/<slot>/events`.
  The main web UI constructs OBS URLs and renders a local sample preview; the
  inspected preview does not fetch that SSE endpoint. Translate the builder
  and preview independently. Changes to external OBS output require an explicit
  scope decision and are not authorized by the UI migration alone.
- The game event-stream API is a separate API-key-authenticated NDJSON surface.
  No main-frontend consumer was found. It can still be affected indirectly if
  common event messages are changed.

### Translation boundary and remaining implementation decisions

Class b backend rendering is the accepted boundary for shared endpoints. The
UI displays their translated text directly; it does not translate it again.
Class a uses codes resolved by the UI. Class c and hidden machine-only text
remain unchanged. These rules supersede the earlier possibility of excluding
all endpoints with any external consumers.

Preserve shared contract shapes and machine semantics. Existing status enums
and codes remain the basis for control flow. Where those are insufficient,
choose a compatible additive application identity rather than parsing English
or changing a text field into an encoded code.

The owner still needs to verify consumer classifications, determine backend
language context for shared responses and future external callers, and define
caching behavior for language-dependent output. A global change to common
error helpers must respect the per-endpoint policy.

## RPC endpoints with message-bearing responses

RPC paths follow `/api/<package>.<Service>/<Method>`. Method names below use the
exact names in the checked-in contracts.

| Service and methods | Potential impact |
| --- | --- |
| `user_service.AuthenticationService.Login` | `LoginResponse.message` is an English-message field in the contract, even when a particular success path leaves it empty. Authentication errors also need stable application reasons. |
| `user_service.RegistrationService.Register`, `VerifyEmail`, `ResendVerificationEmail` | Response `message` fields; verification and resend paths currently populate English success messages. Token expiry and email-verification requirements must become semantic error reasons. |
| `user_service.OrganizationService.ConnectOrganization`, `RefreshTitles`, `SubmitVerification`, `ApproveVerification`, `RejectVerification`, `ManuallySetOrgMembership`, `AdminRefreshUserTitles` | English outcome `message` fields. These operation messages are in scope even though organization title content is excluded from translation. |
| `analysis_service.AnalysisService.RequestAnalysis`, `GetAnalysisStatus` | `message` and `error_message`; existing status enums and queue positions already provide part of the structured result. Persisted worker failures need a display-safe application reason separate from diagnostics. |
| `league_service.LeagueService.InviteUserToLeagues`, `RevokeUserFromLeagues`, `MovePlayerToDivision`, `CreateDivision`, `DeleteDivision`, `RecalculateSeasonExtendedStats`, `AddSeasonTimeBank`, `CancelPlayerResults` | English `message` fields, including dynamic player names, minutes and counts. Some operations need explicit outcome codes in addition to a success boolean. |
| `tournament_service.TournamentService` methods returning `TournamentResponse`; `SetSingleRoundControls` | `warnings` currently contains delimiter-encoded `WooglesError` strings. Structured warnings affect every method sharing these response types; preserve the distinction between a failed request and successful work with warnings. |
| `tournament_service.TournamentService.PreviewPairings` | Nested `SchemePreview.name`, `unavailable_reason`, `notes`, and `RoundsAlternative.reason` are generated English explanations displayed in the director UI. These require message identities and parameters, not just a generic error adapter. |
| `pair_service.PairService.HandlePairRequest`; `tournament_service.TournamentService.RunCOP` | Shared `ipc.PairResponse` already has `error_code`, plus `error_message` and a free-form `log`. No direct UI call to these RPCs was found. Treat them as protected external/backend contracts rather than approved UI migrations; adapt any UI-visible pairing failures at the in-scope boundary. |
| `user_service.SocializeService.GetChatsForChannel`, `GetActiveChatChannels`; `mod_service.ModService.GetChannelChats`, `ListChatChannels` | Player chat history and previews can contain server-authored messages. Existing moderation review excludes senderless system lines; preserve that behavior. Translate moderation's generated channel labels while retaining user-authored chat. No message-body search is present; channel/entity discovery and history retrieval must keep working. |
| `user_service.ProfileService.GetBadgesMetadata`; `config_service.ConfigService.AddBadge` | Badge descriptions and their authoring/catalog pipeline need review. Most profile rendering currently uses the generated badge-description map; the metadata API remains used by admin UI. Badge codes and image identities stay stable. |
| `broadcast_service.BroadcastService.GetBroadcast` | `feed_error` exposes a persisted feed failure in the director UI. Separate the translatable failure category from upstream technical details. |
| `puzzle_service.PuzzleService.GetPuzzleJobLogs` | Nested `PuzzleJobLog.error_status` is displayed in admin UI. Review how diagnostic text is presented alongside a translated failure category. |

The shared `TournamentResponse` covers 29 declared methods: `UnfinishTournament`,
`FinishTournament`, `SetTournamentMetadata`, `PairRound`, `SetRoundControls`,
`SetDivisionControls`, `AddDirectors`, `RemoveDirectors`, `AddDivision`,
`RenameDivision`, `RemoveDivision`, `AddPlayers`, `RemovePlayers`, `MovePlayer`,
`SetPairing`, `SetResult`, `StartRoundCountdown`, `UnstartTournament`,
`OpenRegistration`, `CloseRegistration`, `OpenCheckins`, `CloseCheckins`,
`UncheckAllIn`, `RemoveAllPlayersNotCheckedIn`, `CheckIn`, `Register`,
`InitializeMonitoringKeys`, `RequestMonitoringStream`, and `ResetMonitoringStream`.
Whether a particular method emits warnings today must be checked at its producer.

## Error responses across the API

The common error helpers currently accept strings or underlying errors. Replacing
their output affects consumers of `ConnectError.rawMessage` and any callers
matching English text, even if successful response schemas stay unchanged.

Besides the methods above, assess errors across `AuthenticationService`,
`ProfileService`, `AuthorizationService`, `AutocompleteService`,
`SocializeService`, `IntegrationService`, `OrganizationService`,
`GameMetadataService`, `GameEventService`, `WordService`, `ConfigService`,
`TournamentService`, `LeagueService`, `ModService`, `PuzzleService`,
`GameCommentService`, `CollectionsService`, `BroadcastService`, `PairService`,
`AnalysisQueueService`, `AnalysisService`, and `AnalysisAdminService`.
`RegistrationService` is included as well.

Assessment does not authorize changing class c or unclassified APIs. Common
error helpers must apply the class a/class b rendering policy rather than
changing every error response globally. Hidden machine-only messages remain
excluded regardless of the endpoint's class.

Concrete behavior dependencies include login's email-verification detection,
verification-link expiry handling, the signed-cookie domain handover, and the
demo puzzle's exact English error comparison. These must use application codes
before the text is removed or changed.

Keep Connect transport statuses separate from application reasons. Structured
protobuf error details can carry application codes and parameters without
changing the RPC's normal success response. Adding such details is distinct
from repurposing an existing string field to contain an encoded object or code.

## Account preference and reload contracts

- Account language requires a private read/write preference contract. Extending
  `ProfileService.GetPersonalInfo` and `UpdatePersonalInfo` or adding dedicated
  preference methods are candidates; the final API design is still open.
- `AuthenticationService.GetSocketToken` already returns `front_end_version`.
  `ConfigService.SetFEHash` publishes the deployed frontend version. The
  frontend compares versions during token retrieval and currently offers an
  optional refresh while proceeding to connect.
- Requiring a reload needs a rollout mechanism for tabs already executing the
  old JavaScript. New frontend code alone cannot force an already-running old
  bundle to understand new messages. The owner must select version enforcement,
  deployment order, and recovery behavior before an incompatible cutover.
- A language-selection permission, if chosen in ADR 0005, can use
  `AuthorizationService.GetSelfPermissions` and existing role-management RPCs.
  A developer-tools flag would not require a new account access permission.

## Websocket and streaming contracts

The browser websocket uses `ipc.ErrorMessage.message`,
`ipc.ServerMessage.message`, and `ipc.ChatMessage.message`. Application errors,
announcements such as a game starting, and server-authored challenge results
need structured message identities. User-authored chat remains content.

The messages travel through the API, event bus and socket service, so producer
and consumer deployment order matters. The Redis chat writer and reader also
need corresponding updates for system-message persistence. A tab reload does
not migrate stored messages.

The agreed history policy preserves existing datastore values and stores codes
for new system messages. Translate old system lines through read-time
normalization of known legacy formats, not a bulk rewrite. Keep player-authored
text unchanged. See [ADR 0007](adr/0007-historical-system-message-rendering.md).

`/api/game/eventstream/<gameId>` serves an initial history and subsequent events
as NDJSON. It marshals event payloads from the event bus, so consumers must be
reviewed if the shared event types gain or change message fields. Do not assume
that every websocket message type appears on this stream.

## Direct rendering endpoints

| Endpoint or operation | Potential impact |
| --- | --- |
| `/embed/<gameId>` | HTML title, loading text and HTTP errors, plus a separately built embed bundle. Existing embed URLs must remain usable. |
| `/api/broadcasts/obs/...`, `/api/annotations/obs/game/...`, `/api/annotations/obs/user/...` | Class c OBS-client surfaces: HTML, plain-text fields and SSE streams remain out of scope. The application URL builder and its local preview remain in scope as UI. |
| `/gameimg/...` | Generated images can contain application labels. Preserve image URLs and game notation while preparing captions for translation. |
| `TournamentService.GetTournamentScorecards`, `ExportTournament` | Scorecard/PDF and export outputs need classification of application labels versus stable data formats. The PDF service is a separate Python consumer of tournament metadata and data RPCs. |
| `/integrations/csrf`, `/integrations/patreon/callback`, `/integrations/twitch/callback` | HTTP errors currently contain English or underlying error text. A directly opened callback needs a user-facing renderer rather than a bare translation code. External OAuth protocol fields are not translation targets. |

For in-scope rendering consumers, resolve a code where the final user-facing
message is produced. This includes transactional emails and human-readable
operational notifications outside the deferred Discord integration. Emails use
the recipient's saved UI language with `en-US` fallback. Discord notification
localization and destination-language configuration are out of scope and tracked
in [I18N BL 001](backlog.md#i18n-bl-001-discord-notification-localization).
Hidden integration acknowledgements and generic machine-only messages are
excluded. Raw codes are not a usable replacement for rendered HTML, images,
PDFs or email.

## Non-browser consumers and compatibility limits

`AnalysisQueueService.SubmitResult` returns a free-form `error`, while `FailJob`
accepts a worker-provided `error_message` that can reach persisted job status.
Workers cannot be updated by reloading a browser, and their APIs are outside
the agreed migration scope. Preserve worker contracts and derive display-safe
UI outcomes at an in-scope boundary; keep technical diagnostics separate.

The COP Lambda serializes the same `ipc.PairResponse` used by the API. The PDF
service reads tournament RPC JSON. Repository scripts, Bruno requests, OBS
clients and unknown third-party API consumers may also rely on current shapes
or English messages. Reloading the web UI does not establish permission to
break these consumers.

Preserve machine identities, enum numbers, field numbers, game rules, lexicon
keys, and user-authored content. An additive message field may be wire-compatible
while removing English from an old field is still a behavior change. Reserve
retired protobuf field numbers and names rather than reusing them.

## Evidence

- [RPC registration](../cmd/liwords-api/main.go)
- [Consumer classification](api-classification.md)
- [Method inventory](api-classification.csv)
- [User, profile, organization and authorization contracts](../api/proto/user_service/user_service.proto)
- [Analysis contracts](../api/proto/analysis_service/analysis_service.proto)
- [League contracts](../api/proto/league_service/league_service.proto)
- [Tournament contracts](../api/proto/tournament_service/tournament_service.proto)
- [Pairing response](../api/proto/ipc/pair.proto)
- [Moderation contracts](../api/proto/mod_service/mod_service.proto)
- [Broadcast contracts](../api/proto/broadcast_service/broadcast_service.proto)
- [Puzzle contracts](../api/proto/puzzle_service/puzzle_service.proto)
- [Common error helpers](../pkg/apiserver/errors.go)
- [Frontend version check](../liwords-ui/src/socket/socket.ts)
- [Current refresh action](../liwords-ui/src/socket/reload.tsx)
- [Stored chat and previews](../pkg/stores/redis/redis_chat.go)
- [Game event stream](../pkg/bus/event_api.go)
- [OBS rendering](../pkg/broadcasts/obs.go)
- [COP Lambda](../services/cop-lambda/main.go)
- [PDF API consumer](../services/tourneypdf/src/fetch_tourney.py)
- [Tournament UI caller](../liwords-ui/src/tournament/utils.ts)
- [Pairing CLI caller](../cmd/coprunner/main.go)
- [Game history CLI caller](../cmd/tiles/main.go)
- [Game document CLI caller](../cmd/cwgame/main.go)
- [Registration UI caller](../liwords-ui/src/lobby/register.tsx)
- [Batch registration caller](../scripts/batchregister/register.py)
- [OBS Python consumer](../tools/obs/woogles_text_sources.py)
- [OBS URL builder](../liwords-ui/src/broadcasts/OBSPanel.tsx)
- [Local OBS preview](../liwords-ui/src/broadcasts/OBSFieldPreview.tsx)
- [Cookie and API-key authentication](../pkg/apiserver/auth_user.go)
- [Connect error details](https://connectrpc.com/docs/web/errors/)
- [Protobuf evolution guidance](https://protobuf.dev/best-practices/dos-donts/)
