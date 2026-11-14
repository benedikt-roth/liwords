# Ubiquitous language

Use the terms below in discussions, tickets and architecture changes. This is a vocabulary derived from code and corroborated where useful by user manuals; it still needs domain-expert agreement. E-identifiers resolve in the [source index](../evidence.md).

## Distinctions to learn first

| Avoid the ambiguous expression | Prefer these precise terms |
| --- | --- |
| “Player” for every identity | **User** for an account/identity; **game player** for a participant in a game; **annotated player** for a person named in a recorded game; **entrant** for a competition participant. |
| “Match” for everything | **Seek/match request** for a proposal; **pairing** for an assigned competition encounter; **game** for one played/recorded board game. |
| “Game type” as a catch-all | Separate **game type**, **game mode**, **variant**, **rating mode**, **lexicon**, **letter distribution** and **board layout**. |
| “Division” without context | **Tournament division** or **league division**. |
| “Rating” for a ranking | **Rating** estimates strength; **standing/rank** describes a competition; **mistake index** measures analysis-derived play quality. |
| “Finished game” for completed editorial work | **Game ended** for the rules outcome; **annotation done** for the annotator's completion flag. |
| “Slot” | **Broadcast slot** belongs to an event; **stream slot** belongs to a user and can follow a broadcast slot. |
| “Private” | Name the actual policy: **private analysis**, **private broadcast**, **unlisted collection**, **rack censorship**, or **verification evidence access**. |
| “Event” | Specify **history event**, **transport message**, **broadcast event**, or a **business fact** such as a game ending. |
| “Language” | Specify **interface locale**, **lexicon**, **letter distribution**, or **tile encoding**. They are different dimensions. |

## People, identity and access

| Term | Meaning | Code equivalent / source |
| --- | --- | --- |
| User | Woogles identity, registered or anonymous; registered accounts have credentials and profile capabilities. Public UUID and internal numeric database ID are distinct. | `entity.User`, `users`; E18 |
| Profile | Public/personal presentation and per-pool ratings/statistics associated with a registered user. Some fields are masked for youth accounts. | `entity.Profile`, `profiles`, profile service; E18 |
| Session | Server-tracked login with expiry/CSRF state; not the same as one WebSocket/tab connection. | `entity.Session`, `sessions`; E18 |
| Connection / realm | A socket connection identifies one client/tab; a realm scopes delivery/presence such as lobby or game. Technical routing terms, not account identities. | socket hub, `RegisterRealmRequest`; E04 |
| Role / permission | A role groups permissions; a permission authorizes a named capability. Director rights can additionally be scoped to a particular event. | RBAC tables, `pkg/auth/rbac`, authorization service; E18 |
| Organization membership / title | A connection to an external player organization and its recognized titles; not a Woogles club membership. | organization service/provider adapters; E18, E19 |
| Verification request | Evidence submitted for organization-related identity/membership review, with reviewer outcome; distinct from email verification. | `pkg/verification`, verification tables and S3 evidence; E18 |
| Provider integration | An account link and stored provider state/credentials used to obtain external data. | `integrations`, OAuth service; E19 |
| Entitlement | An allowance or access decision derived from permissions/subscription data and usage, e.g. BestBot games. | `pkg/entitlements`, paid-tier data; E19 |

## Matching, rules and play

| Term | Meaning | Code equivalent / source |
| --- | --- | --- |
| Sought game | Persisted proposal for a game, before an actual game exists. It can be an open seek or directed request. | `entity.SoughtGame`, `soughtgames`; E10 |
| Seek | A proposal advertised to eligible opponents, with settings and optional rating/follow restrictions. | `ipc.SeekRequest`; E10 |
| Match request | A seek/proposal addressed to a named receiving user. This is a product distinction represented by the same seek structure. | `SeekRequest.receiving_user`, `receiver_is_permanent`; E10 |
| Acceptance / claim | Acceptance validates the proposal; the server atomically consumes/claims it before creating a game. | `acceptSeek`, `SoughtGameStore.Claim`; E10 |
| Rematch / streak | A new game requested from a prior game; original request ID links the series. | `rematch_for`, `original_request_id`, streak metadata; E10, E13 |
| Game settings | Agreed lexical, board, variant, challenge, timing and rated/casual choices. Despite its name, `GameRequest` also represents persisted settings, not just a network request. | `ipc.GameRequest`, `ipc.GameRules`; E13 |
| Native game | A game played on Woogles by two online participants, possibly including a bot. | `GameType.NATIVE`, `entity.Game`; E08, E13 |
| Annotated game | A recorded/edited game with a creator, named players and a GameDocument; commonly used for IRL broadcasts or study. | `GameType.ANNOTATED`, GameDocument/annotation service; E15 |
| Bot-vs-bot game | Generated engine game, including potential puzzle source material. | `GameType.BOT_VS_BOT`; E14 |
| Game mode | Real-time or correspondence scheduling/timing policy. | `GameMode.REAL_TIME`, `CORRESPONDENCE`; E13 |
| Variant | A rule variation within the crossword game family, e.g. classic, super-board variants or WordSmog. Names differ across libraries/UI; use the stored variant when discussing behavior. | `GameRules.variant_name`, engine variant; E13 |
| Rating mode | Whether the game's eligible outcome affects skill rating: rated or casual. | `RatingMode.RATED`, `CASUAL`; E13 |
| Lexicon / dictionary | Versioned accepted-word resource selected for play. UI “dictionary” maps to this concept; it is not an interface translation. | `GameRequest.lexicon`, allowed-new-game lexica; E10, E13 |
| Letter distribution | Tile alphabet, multiplicities and point values, such as English or German. Compatibility with the lexicon is a separate rule. | `GameRules.letter_distribution_name`, word-golib; E10 |
| Board layout | Board dimensions and premium-square arrangement. | `board_layout_name`, crossword/super layouts; E13 |
| Tile / machine letter | A tile is a game resource; a machine letter is its encoded representation. A tile may have multiple visible characters, and a blank assigned a letter is not that ordinary letter tile. | word-golib tile mapping, binary `machine_letters`; E13, protocol reference |
| Position | Current board, racks/bag, scores, player on turn and relevant play state, together with the rules required to interpret it. It is not the full history. | Native engine state, GameDocument, `xwordgame.State`; E08, E15 |
| Rack / bag | Tiles held by a player / tiles available for draws. Opponent racks are sensitive during play. | engine state, rack-censorship helpers; E13 |
| Move / gameplay command | Player intention: place tiles, pass, exchange, challenge or resign. The server validates and derives resulting events. | `ClientGameplayEvent`; E13 |
| Turn | A player's opportunity/action in the sequence of play. One turn may create several history events. | player turn counters; E08, E13 |
| History event | Recorded game fact such as placement, phony return, challenge bonus, lost turn or end-rack adjustment. Distinct from a NATS/WebSocket message. | Macondo/IPC `GameEvent`, staged `game_turns`; E08, E13 |
| Challenge rule / challenge | Policy for contesting words, and the action of contesting a play. Void, single, double, triple and point-penalty policies differ. | `macondo.ChallengeRule`, `CHALLENGE_PLAY`; E13 |
| Phony | A played word unacceptable under the selected lexicon; treatment depends on the challenge rule. | invalid-word/challenge-return logic; E13 |
| Bingo / spread | Playing all rack tiles for the configured bonus / score difference between opponents. Spread is also accumulated in standings. | game statistics and competition standings; E11–E13 |
| Initial time / increment / overtime | Starting clock allowance / time added or refilled under mode policy / permitted clock overrun. Correspondence uses different timer semantics. | `GameRequest`, `Timers`, `FillIncrement`; E13 |
| Time bank | Additional correspondence reserve consumed when turn allowance is exceeded. Not the same as real-time overtime. | `time_bank_minutes`, timer calculations; E10, E13 |
| Game outcome / end reason | Winner or tie plus why play terminated: standard finish, timeout, resignation, abort, cancellation, consecutive zeroes, triple challenge, force forfeit or adjudication. | `GameEndReason`, winner/loser indices; E13 |
| Adjudication | External neutral ruling, such as a league deadline decision; can yield either winner or a tie. Different from penalizing a departed opponent by force forfeit. | `ADJUDICATED` vs `FORCE_FORFEIT`; E13 |
| Rating pool / rating key | Group of comparable rated results determined by normalized lexicon family, variant and time-control category. | `VariantKey`, `ToVariantKey`; E13 |
| Rating / deviation / volatility | Strength estimate, uncertainty and variability parameters, rather than a single permanent rank. | `SingleRating`, Glicko calculation; E13 |

## Organized competition

| Term | Meaning | Code equivalent / source |
| --- | --- | --- |
| Tournament | Competition with directors, divisions, round/pairing controls, entrants and results. | `entity.Tournament`, tournament service; E11 |
| Club / club session | Persistent club space / child competition session spawned from it. The implementation shares the tournament model. | competition types `club`, `child`, parent ID; E11 |
| Director / entrant | User authorized to manage a competition / participant registered or added to it. Neither is necessarily the tournament creator. | directors, tournament persons/entries; E11 |
| Tournament division | Tournament-scoped group with its own players, round controls, pairing manager and standings. | `TournamentDivision`, `DivisionManager`; E11 |
| Round / round control | An organized stage of a tournament / configuration of pairing and game behavior for that stage. | `RoundControl`, classic division; E11 |
| Pairing | Assignment of participants to an encounter in a round, including byes/forfeits and potentially multiple game results. Not an open seek. | `ipc.Pairing`, division manager; E11 |
| Bye / forfeit | Competition outcomes without normal completed play; scoring is controlled by tournament rules. | tournament result enums and configured scores; E11 |
| Check-in / ready | Check-in confirms event participation; ready indicates preparation to start an assigned encounter/game. Scope them explicitly. | tournament check-in fields, readiness tables, `ReadyForGame`; E11, E13 |
| IRL result | Scores/outcome entered for a face-to-face encounter. Does not require native online move history. | IRL mode/result entry; E11 |
| League | Recurring competition with settings and successive seasons. | `leagues`, league service; E12 |
| Season | Dated cycle of a league with registration, division placement, play and final outcomes. | `league_seasons`, `SeasonStatus`; E12 |
| League division | Season-specific field of players, identified independently from a tournament division. | `league_divisions`, division UUID; E12 |
| Season registration / placement | User's participation record for a season / assignment based on history, ratings and field-balancing policy. | `league_registrations`, placement/rebalance code; E12 |
| Standing / crosstable | Ranked competition projection / pairwise results view. Rebuildable from underlying competition/game data. | league/tournament standings and crosstable; E11, E12 |
| Promotion / relegation / champion | End-of-season placement outcomes governing subsequent participation. They are not user roles. | `StandingResult`, registration placement status; E12 |

## Study, analysis and presentation

| Term | Meaning | Code equivalent / source |
| --- | --- | --- |
| Puzzle | Training task drawn from one game position, with a stored answer, tags and difficulty rating. | puzzles table, source game/turn; E14 |
| Puzzle attempt | Per-user interaction with a puzzle: count, first/last timestamps, unresolved/solved/given-up state and rating snapshots. Incorrect guesses can leave it unfinished. | `puzzle_attempts`, nullable `correct`; E14 |
| Puzzle generation job | Batch process that chooses suitable source positions, fills configured buckets and records progress/errors. Separate from analysis jobs. | `PuzzleGenerationJobRequest`, generation logs; E14 |
| Analysis job / claim | Durable server work item for a completed game / temporary worker ownership maintained by heartbeats. | `analysis_jobs`, queue API; E09 |
| Analysis result / mistake index | Versioned computed move-quality output / summary metric used in game and league analysis. Not a player's rating. | Macondo result, S3 key and summary columns; E09 |
| Local analyzer / simulation | Browser-side Wolges computation on a position; may be interactive and stopped without a server queue entry. | WASM loader/lab worker; E06 |
| GameDocument | Versioned editable game representation containing rules, state and events. It is a protocol/data model, not a separate service. | `ipc.GameDocument`, document store; E15 |
| Annotation done | Annotator's completion of entering/editing the game. Independent of its rules end reason. | `annotated_game_metadata.done`; E15 |
| Collection / chapter | Curated game list / ordered member referencing a native or annotated game, with an optional chapter title. | `collections`, `collection_games`; E15 |
| Game comment | Authored discussion attached to a game/turn; collection comment views gather member-game comments. | comment service/tables; E15 |
| GCG / CGP | GCG carries game history for import/export; CGP represents a position for interchange/analysis. Neither name denotes a runtime service. | GCG handlers; [CGP specification](https://github.com/woogles-io/open-protocols/blob/main/cgp/README.md); E15 |
| Broadcast event | A configured presentation of an external/live competition with feed, directors and annotation claims. Different from a history event or bus notification. | `broadcasts`; E16 |
| Feed snapshot | Last successfully fetched external event data, stored with format/hash/poll status so read paths do not need the original feed. | `broadcast_feeds`; E16 |
| Table claim / broadcast game | Binding of broadcast/division/round/table to an annotator and annotated game. | `broadcast_games`, claim operations; E16 |
| Broadcast slot | Event-owned named pointer to the table/game to present. Directors can retarget it as rounds progress. | broadcast slot records; E16 |
| Stream slot | User-owned stable overlay alias following a broadcast slot or the user's annotation activity. | `user_obs_slots`; E16 |
| OBS overlay | HTML/text presentation consumed as an OBS browser source; SSE supplies updates and rebinding. | OBS HTTP handlers; E16 |

## Community and conduct

| Term | Meaning | Code equivalent / source |
| --- | --- | --- |
| Follow / block | Directed social relationship; blocks and followed-only preferences also influence seek eligibility/visibility. | SocializeService and matching checks; E10, E17 |
| Chat channel / message | Conversation scope / authored message, distinct from the socket audience carrying it. | Redis chat store, IPC chat; E17 |
| Presence | Current online/realm/tab-related activity; transient state, not durable account membership. | Redis presence store and socket realm registration; E04, E17 |
| Moderation action | Scoped intervention such as mute, suspend account/rated/all games, remove profile content or reset ratings/statistics. | `ModActionType`, current/history actions; E17 |
| Notoriety / automod | Accumulated conduct signal / automated policy applying sanctions based on game behavior. Not a skill metric. | `pkg/mod`, notorious-game records, completion hooks; E13, E17 |

When a term crosses contexts, qualify it with its owner: “league standing”, “puzzle rating”, “annotation done”, “event broadcast slot”. Preserve public protocol/storage names in code references even where the preferred business term is clearer.
