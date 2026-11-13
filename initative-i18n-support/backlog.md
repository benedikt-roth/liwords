# Internationalization backlog

These items are deferred from the first iteration. Listing an item does not
schedule it or approve an implementation.

## I18N BL 001 Discord notification localization

Status: Backlog. Not scheduled for the first iteration.

Owner: To be assigned by the maintaining team.

### Goal

Prepare human-readable Discord notifications for translation and define how a
shared destination selects its language without depending on the UI language
of the person who triggers an event.

### Current integration

- Moderation and tournament notifications use the deployment-provided
  `DiscordToken` webhook destination.
- League announcements use `LeagueDiscordToken`.
- Build and deployment notifications use the GitHub Actions secret
  `DISCORD_WEBHOOK` and the notification shell script.
- The actual destinations are deployment configuration, not checked-in channel
  choices. No per-user Discord subscription or destination selection was found
  in the application UI, API or database schema.

### Future work

Inventory the moderation, tournament, league and build/deployment messages.
Give their human-facing templates stable identities and named parameters, and
render text before posting it to Discord. Preserve webhook behavior, formatting,
links, mentions and delivery semantics.

Decide whether language is configured per destination or remains fixed for a
given notification stream. A deployment-level setting defaulting to `en-US` is
an option for review, not an accepted design. Determine ownership, fallback,
formatting/timezone policy and configuration of future translated destinations.
Per-user subscriptions would be a separate product capability and are not
implied by this item.

Generic machine-to-machine acknowledgements and hidden diagnostic messages
remain outside the translation scope. Do not put webhook URLs or secrets into
catalogs or documentation.

### Completion criteria for a future implementation

1. Every targeted human-readable Discord template is cataloged and validated.
2. The maintaining team has chosen and documented the destination language and
   fallback policy.
3. Template parameters and formatting are verified without changing recipients
   or delivering real notifications as a side effect of tests.
4. Delivery behavior and existing `en-US` output are preserved except for
   explicitly reviewed wording corrections.

### References

- [First-iteration scope](feature-specification.md)
- [Clarification decision 11a](decisions.md#11a-discord-integration-deferred)
- [Configuration](../pkg/config/config.go)
- [Discord delivery helper](../pkg/notify/notify.go)
- [Moderation messages](../pkg/mod/notify.go)
- [Tournament messages](../pkg/tournament/service.go)
- [League messages](../pkg/league/league_notify.go)
- [Deployment workflow](../.github/workflows/build-and-deploy.yml)
- [Deployment notification script](../scripts/utilities/notify-discord.sh)
