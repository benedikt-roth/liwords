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
across 23 services. It records the category, caller evidence, translation policy,
and source schema, plus a separate assessment of the text each method returns.

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

## Does the API expose text that may need translation?

Reviewed at `eea0fb279` against the registered handlers, protobuf responses,
and relevant producers and UI call sites. A response body can contain a warning
or rejection even when the RPC succeeds at the transport level. The inventory
assesses those bodies and transport errors independently. A string in a schema
only counts when the reviewed code populates it on a path relevant to people.
This is a source assessment. Deployed consumers and contents of live stores
still need maintainer verification where indicated.

| CSV field | How to use it |
| --- | --- |
| `potential_translation` | `yes` means a response or error has an identified candidate. `review` means a candidate depends on unresolved visibility, provenance, or implementation. `no` means no translation target was identified for the first iteration. |
| `response_text_status`, `response_text`, `response_text_evidence` | `candidate` identifies application-generated prose, a presentation fragment, encoded warning or rendered artifact. `excluded` records known content, protocol fields or diagnostics to preserve. `review` records a concrete uncertainty. `none` means no application-owned presentation text was identified in the current response body. |
| `error_text_status`, `error_text`, `handler` | `candidate` identifies an operation, validation, access, or domain failure with a potential user-facing outcome. It includes relevant errors surfaced by called helpers. `review` covers diagnostic or control-path errors whose user-facing treatment needs checking, and stubbed methods. `excluded` records errors on protected external-only interfaces. `handler` points to the implementation for the row. |

| Consumer class | Yes | Review | No | Total |
| --- | ---: | ---: | ---: | ---: |
| a, provisional UI-only | 183 | 21 | 0 | 204 |
| b, confirmed shared | 5 | 0 | 0 | 5 |
| c, external-only | 0 | 0 | 6 | 6 |
| Unassigned | 31 | 7 | 0 | 38 |
| **Total** | **219** | **28** | **6** | **253** |

**35 RPCs have an identified response-body candidate.** Another 12 have a
response body to review. Of the 219 `yes` methods, 184 are flagged by an error
candidate without a confirmed response-body candidate. A shared authentication,
permission or validation message can account for many endpoints. These are
method counts, with no inference about unique strings or required schema changes.

- **Count text by ownership.** User chat, comments, bios, personal names,
  organizer-supplied names/descriptions, and moderator-authored notes stay as
  submitted. Announcement content, provider titles and lexical data retain their
  existing exclusions. Application-owned wording around them can still be a
  candidate. For example, `GetCollectionComments` and `GetCommentsForAllGames`
  build the `vs` player label, and `ListChatChannels` adds labels such as `Private`
  and `Lobby`. Preserve supplied names as parameters. [Comment label producer](../pkg/comments/service.go#L110) · [Moderation channel labels](../pkg/mod/chat_review.go#L125)
- **Check how each handler uses a shared schema.** `PairRound`, `SetPairing`
  and `SetSingleRoundControls` populate tournament warnings. Other handlers
  that share `TournamentResponse` do not currently populate those warnings.
  `Login` and `Register` leave their declared success `message` empty.
  `VerifyEmail` and `ResendVerificationEmail` fill theirs. [Warning producers](../pkg/tournament/service.go#L154) · [Login return](../pkg/auth/authn_service.go#L202) · [Registration handlers](../pkg/registration/service.go#L33)
- **Check every projection of a stored field.** `GetChatsForChannel` can return
  server-authored challenge lines. `GetChannelChats` filters those lines out.
  The known challenge producer writes only to game and observer channels, while
  the recent-channel index exposes private and competition chat previews.
  Preserve the parsed `display_name` tags and verify any future overlap before
  changing preview storage or projection. [Player chat read](../pkg/user/services/socialize.go#L309) · [Moderation filter](../pkg/mod/chat_review.go#L65) · [Challenge writes](../pkg/gameplay/game.go#L413) · [Recent-channel index](../pkg/stores/redis/add_chat.lua#L88)
- **Record diagnostics where they surface.** Broadcast `feed_error` and puzzle
  `error_status` are displayed in staff UI. They need display-safe failure
  categories while retaining technical details where appropriate. Analysis
  `error_message` is returned, but the checked-in analyzer labels the enum
  and no authored UI display of the raw field was found. `GetSignedCookie`
  has an exact-text control dependency. Preserve the existing string until that
  branch can use a semantic signal. [Broadcast panel](../liwords-ui/src/broadcasts/BroadcastDirectorPanel.tsx#L227) · [Puzzle job logs](../liwords-ui/src/admin/puzzle_generator.tsx#L78) · [Analysis button](../liwords-ui/src/gameroom/analyzer.tsx#L825) · [Cookie handover](../liwords-ui/src/App.tsx#L360)
- **Keep machine fields stable when they are also displayed.** Examples include
  time-control tokens, tier names used for quota decisions, and generated
  censorship aliases that occupy identity fields. Translate a separate display
  label if needed. `ExportTournament` returns TSH, TOU and CSV for downstream
  use. `GetTournamentScorecards` returns a PDF ZIP with application-owned
  captions. Keep exports, QR destinations, and player-supplied data stable.
  [Time-control tokens](../pkg/entity/variants.go#L15) · [Tier consumer](../liwords-ui/src/lobby/bot_selector.tsx#L28) · [Censorship fields](../pkg/gameplay/service.go#L379) · [Exports](../pkg/tournament/export.go#L26) · [PDF captions](../services/tourneypdf/src/generator.py#L410)

Prioritize populated outcomes, warnings and generated labels, then group
reusable errors by meaning. For `review` entries, confirm displayed consumers,
including external ones, and distinguish legacy/generated content from authored
content. Examples are stored puzzle prose, generated parts of moderation notes,
and formatted dates embedded in stored club-session names. The owner still
needs to verify provisional and unassigned callers before transport changes.

## Class b Confirmed shared endpoints

RPC paths use `/api/<package>.<Service>/<Method>`.

| Endpoint | UI evidence | Non-UI evidence | Backend translation target |
| --- | --- | --- | --- |
| `tournament_service.TournamentService/GetTournament` | [Tournament UI](../liwords-ui/src/tournament/utils.ts) | [COP CLI](../cmd/coprunner/main.go), [PDF service](../services/tourneypdf/src/fetch_tourney.py) | Application errors, including the encoded division-manager failure. Preserve tournament data. |
| `tournament_service.TournamentService/GetTournamentMetadata` | [Tournament UI](../liwords-ui/src/tournament/utils.ts), editor and wizard | [PDF service](../services/tourneypdf/src/fetch_tourney.py) | Application errors. Review the generated date in child club-session names under the formatting decision and preserve stored values. |
| `game_service.GameMetadataService/GetGameHistory` | [Embedded viewer](../liwords-ui/src/embed/embed.tsx) | [Image CLI](../cmd/tiles/main.go) | Application errors; preserve history and game notation. |
| `omgwords_service.GameEventService/GetGameDocument` | [Editor](../liwords-ui/src/boardwizard/editor.tsx), game room | [Game-document CLI](../cmd/cwgame/main.go) | Application errors; preserve document data. |
| `user_service.RegistrationService/Register` | [Registration form](../liwords-ui/src/lobby/register.tsx) | [Batch registration](../scripts/batchregister/register.py), [user script](../scripts/utilities/register-user.sh), [bot script](../scripts/utilities/register-bot.sh) | Signup validation and error text. The current success response leaves `message` empty. |

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

| Surface | Current classification | Can expose text that may need translation? |
| --- | --- | --- |
| Browser websocket `/ws` | a candidate, with external consumers unverified | **Yes.** `ErrorMessage.message`, `ServerMessage.message`, and system-origin `ChatMessage.message`. Exclude user chat, routing, presence and control data. [Error producer](../pkg/bus/bus.go#L235) · [Start notice](../pkg/bus/gameplay.go#L502) · [Challenge result](../pkg/gameplay/game.go#L391) |
| `/api/game/eventstream/<gameId>` | c by intended API-client use | **No in this iteration.** Preserve NDJSON event/history fields and existing error bodies. [Stream projection](../pkg/bus/event_api.go#L120) |
| `/api/broadcasts/obs/...`, `/api/annotations/obs/game/...`, `/api/annotations/obs/user/...` | c by OBS-client purpose | **No in this iteration.** Preserve external OBS HTML, plain text, SSE and sentinels. The owned URL builder and local preview use application UI copy. [OBS handler](../pkg/broadcasts/obs_handler.go#L134) · [Python consumer](../tools/obs/woogles_text_sources.py) · [Preview UI](../liwords-ui/src/broadcasts/OBSFieldPreview.tsx) |
| `/embed/<gameId>` | a candidate for the owned page, with other consumers unverified | **Yes.** Initial HTML title and loading text, plus user-facing HTTP errors. Review the separate JS viewer's loading and control copy in its own build. Preserve routes, game data and option values. [Go renderer](../pkg/embed/service.go#L130) · [Standalone viewer](../liwords-ui/src/embed/embed.tsx) |
| `/gameimg/...` | a candidate for the owned image renderer, with other consumers unverified | **No app copy identified in the current endpoint.** The renderer draws the board, tile notation and numeric scores. Render failures are logged and return a generic 404. Review application-owned wording in other image renderers separately. [Image handler](../pkg/memento/service.go#L179) · [Score rendering](../pkg/memento/tiles.go#L1915) |
| `/integrations/patreon/callback`, `/integrations/twitch/callback` | a candidates for browser-facing errors | **Yes.** Browsers can receive application error prose directly. Translate user-facing outcomes and retain provider diagnostics separately. [Patreon callback](../pkg/integrations/patreon.go#L174) · [Twitch callback](../pkg/integrations/twitch.go#L35) |
| `/integrations/csrf`, optional `/api/human-check` | Browser/control endpoints | **Review.** HTTP bodies include English rejection text. Verify whether the browser or edge presents the body to the user or consumes the status/cookie and supplies separate copy. [CSRF handler](../pkg/integrations/oauth.go#L32) · [Human check](../pkg/humancheck/humancheck.go#L105) |
| VDO webhook, health and debug/control endpoints | c or internal infrastructure | **No in this iteration.** Acknowledgments, operational errors and diagnostics are machine or operator surfaces. [VDO webhook](../pkg/vdowebhook/service.go#L39) · [Route registration](../cmd/liwords-api/main.go#L348) |

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
