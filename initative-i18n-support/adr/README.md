# Internationalization architectural decisions

Product requirements are confirmed in [the decision log](../decisions.md) and
assembled in [the draft specification](../feature-specification.md). The records
below distinguish those requirements from proposed implementation choices.
No proposed runtime, route, role or rollout mechanism is approved merely by its
inclusion here.

| ADR | Subject | Status and owner action |
| --- | --- | --- |
| [0001](0001-localized-help-and-stable-urls.md) | Help locale resolution and stable URLs | Requirements confirmed; owner selects route mechanics and precedence. |
| [0002](0002-message-contracts-and-catalog-runtimes.md) | Message contracts and catalogs across runtimes | Proposed; owner chooses libraries, schema and resource generation after a compatibility slice. |
| [0003](0003-account-locale-and-formatting.md) | Account preference, formats and timezone | Requirements confirmed; private API/storage and backend context details proposed. |
| [0004](0004-secondary-renderers-and-email.md) | Secondary rendering surfaces and email | Scope and recipient policy confirmed; renderer adapters proposed. |
| [0005](0005-experimental-language-selector-access.md) | Hidden selector access | Open, explicitly for repository owner: dedicated `I18n Tester` role recommended, or developer-tools flag. |
| [0006](0006-client-reload-and-api-compatibility.md) | API classes and required client reload | Rendering policy confirmed; owner verifies classifications and chooses rollout enforcement. |
| [0007](0007-historical-system-message-rendering.md) | Unchanged history with translated display | Requirements and English fallback confirmed; adapter/schema proposed. |
| [0008](0008-catalog-checks-and-copy-review.md) | Automated checks and copy audit | Checks and maintaining-team copy review required; tooling and future language governance open. |

Deferred integration work is tracked in [the i18n backlog](../backlog.md).
