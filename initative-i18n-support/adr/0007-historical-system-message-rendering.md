# ADR 0007 Historical system message rendering

Status: Proposed implementation. Storage, historical-display requirements and
the original-English fallback for unrecognized legacy formats are confirmed.

## Context

Historical game chat stores English system lines alongside user-authored chat
in Redis streams. The current server-authored challenge-result lines have no
username or user ID. Their text has three forms: a valid challenged play, a
play challenged off with no returned tiles, and a play challenged off with a
returned-tiles suffix. Player and observer channels each receive a stored line.

`SocializeService.GetChatsForChannel` requests the latest 100 records of a
channel. `RedisChatStore.OldChats` uses `XREVRANGE`, then returns them oldest
first. It does not query message content. `GetChat` retrieves a specific stream
record by its ID. The UI also consumes history for reconnect catch-up and
displays last-message previews.

## Confirmed requirements

1. Keep existing message records unchanged in the datastore.
2. Store structured codes and parameters for new user-facing system messages
   after migration, rather than only a rendered English sentence.
3. Translate historical system messages when displaying them, despite leaving
   their stored representation unchanged.
4. Preserve user-authored chat and hidden machine-to-machine messages as content
   or protocol data outside the translation scope.
5. Respect the API classification: class a sends identities to the UI; class b
   renders text on the backend; class c remains outside the i18n migration.
6. If a legacy system-message format cannot be recognized reliably, display
   the original English text as the fallback.

No bulk rewrite of existing history is required. Existing expiry, stream limits,
message IDs and timestamps must not change as a side effect of localization.

## Proposed implementation

Normalize records at the read boundary into a logical representation of user
content, a structured system message, or an unrecognized legacy system message.
New system records already provide a code and parameters. A bounded legacy
adapter maps known old English formats to those same identities and parameters.
This normalization does not write back to storage.

Use trusted system-origin metadata together with the known template; never
infer that a player-authored message is a system message because it happens to
contain the same English words. The current senderless convention supplies
evidence for existing challenge lines. Use an explicit message kind for new
records so identity does not depend on absent sender fields indefinitely.

Preserve returned tile strings and other game notation as parameters. Match
known fixed text, not arbitrary English, and keep the adapter separate from
application control flow. This is a presentation compatibility layer for legacy
records, not a general parser for transport errors or user content.

Send the normalized code and parameters on class a paths and translate on
render. On class b paths, render the normalized identity using the request's
language context before sending. Shared storage or event changes must continue
to support the existing projections used by excluded class c consumers without
introducing a translation requirement for those clients.

Apply the same policy to channel views, last-message previews, reconnect
catch-up, and application-owned summaries or notifications that display a
stored system line. Some preview paths currently flatten a message to text;
retain system-origin and identity information before flattening so user text
cannot be mistaken for a known legacy format.

## Search and retrieval findings

No full-text search over historical message bodies was found in the current
chat APIs, Redis implementation or UI.

Moderation's `ListChatChannels` supports channel-prefix and username filtering,
explicit channels, and a `find` lookup by a channel or game/tournament/club/league
ID, address, name or slug. Channel scanning and entity-name search are not
message-body search. `GetChannelChats` then retrieves messages from the chosen
channel.

Moderation's `reviewChats` filters out senderless system lines before channel
summaries, message views and exports. Preserve that exclusion; translating
historical game chat does not require adding system messages to moderation.
Usernames, IDs, slugs and stored message text must not be rewritten to support
localized display. Existing channel discovery, counts and retrieval remain
valid independently of the rendered language; no message-text search index
migration is needed for the functionality found here.

## Unrecognized legacy formats

The repository shows current templates, but does not prove which older formats
may still exist in a deployed datastore. An unrecognized historical sentence
cannot reliably supply a message identity and parameters.

Preserve and display the original English as the confirmed fallback. Record
the missing mapping for developer follow-up without collecting unrelated
message content. Do not silently discard the line, show a raw translation key,
or machine-translate arbitrary user content.

Retain compatibility as long as legacy records can be retrieved. Current game
chat expiration is not proof that every old record or other channel has
expired, and this feature must not alter retention to avoid supporting history.

## Validation

Verify the known legacy templates and their parameters, structured new records,
mixed old/new history, player and observer channels, last-message previews and
reconnect catch-up. Check that storage values, IDs, timestamps and retrieval
order remain unchanged. A user message identical to a legacy system sentence
must remain user content. Preserve moderation's system-line exclusion and its
channel discovery behavior. Exercise the agreed unknown-format fallback and
both class a and class b rendering boundaries.

## References

- [Clarification decisions](../decisions.md)
- [API rendering policy](0006-client-reload-and-api-compatibility.md)
- [API classification](../api-classification.md)
- [Historical system message producer](../../pkg/gameplay/game.go)
- [Redis history and previews](../../pkg/stores/redis/redis_chat.go)
- [Chat history RPC](../../pkg/user/services/socialize.go)
- [Moderation API contract](../../api/proto/mod_service/mod_service.proto)
- [Moderation channel lookup and system-message exclusion](../../pkg/mod/chat_review.go)
- [Moderation UI](../../liwords-ui/src/admin/chat_review.tsx)
- [Chat message display](../../liwords-ui/src/chat/chat_entity.tsx)
- [Channel preview display](../../liwords-ui/src/chat/chat_channels.tsx)
- [History catch-up](../../liwords-ui/src/chat/chat.tsx)
