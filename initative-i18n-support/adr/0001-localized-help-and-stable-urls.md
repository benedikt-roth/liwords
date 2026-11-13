# ADR 0001 Localized help and stable URLs

Status: Proposed. The requirements below are confirmed; the routing mechanism
and future locale-link precedence remain open for the repository owner.

## Context

Help manuals are part of the application and must support internationalization.
The current routes include `/docs`, `/docs/:manualId`, and
`/docs/:manualId/:sectionId`. Manuals have stable IDs and section IDs, and the
streaming manual currently contains US English content.

Moving manuals into locale-specific files or introducing locale-prefixed URLs
can break existing links. A user's future language preference also needs to
select the appropriate language version without changing the identity of a
manual or section.

## Confirmed requirements

1. Mark existing help content as `en-US`.
2. Support additional language versions and routing based on the user's i18n
   language setting.
3. Keep existing URLs and deep links working after migration. If localized
   paths are introduced, the old paths must still reach the corresponding
   US English content and sections.
4. Ship only US English in the first iteration.

The example `en_us` prefix describes a possible path convention. It does not
require underscores or prescribe a filesystem layout. Language metadata uses
`en-US`.

## Proposed content model

Resolve help content through a locale-aware registry keyed by stable manual and
section IDs, with the existing content explicitly registered as `en-US`. Keep
translated titles and physical source-file paths separate from URL identities.
The same manual bodies are used on pages and in help modals, so both consumers
need the selected content language rather than routing alone determining it.

An `en-US` fallback for an unavailable future help translation is recommended;
the owner must confirm fallback and explicit-locale-link precedence alongside
the route strategy. Only `en-US` is available in the first iteration.

## Options to resolve

- Keep current routes and resolve the content language from the user setting.
- Add locale-prefixed routes while retaining legacy routes as aliases.
- Add locale-prefixed routes and redirect legacy routes while preserving
  manual IDs, section IDs, query parameters, and fragments.

The specification must distinguish language selection on navigation from the
compatibility behavior of an existing unprefixed link. It must also define
explicit locale-link behavior and fallback when a translation is unavailable.

## Consequences and validation

Manual and section identity must remain independent of translated titles and
content organization. Moving a source file alone must not change a public URL.

Verification must cover existing index, manual, and section links; direct page
loads; bookmarks containing query parameters or fragments; and navigation with
a language preference. A development-only test locale can exercise language
routing without adding another supported production language.

## References

- [Confirmed clarification decisions](../decisions.md)
- [Application routes](../../liwords-ui/src/App.tsx)
- [Manual registry](../../liwords-ui/src/docs/registry.ts)
- [Streaming manual](../../liwords-ui/src/docs/manuals/streaming.tsx)
