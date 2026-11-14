# ADR 0009 — One central language file

**Status.** One central language file is a stakeholder requirement. The assessment recommends the interpretation and packaging approach below. The requester should confirm the file boundary and the maintainers should approve the implementation after the build proof.

## Context

Woogles renders its own wording in the main React application, a separately built embed, Go handlers and email jobs, early HTML, and a Python scorecard service. The draft already calls for feature namespaces and leaves catalog ownership and runtimes open. Different build and deployment paths need to consume the same authored wording. [Feature specification, FR 10–11](../feature-specification.md#catalogs-and-formatting) · [ADR 0002](../adr/0002-message-contracts-and-catalog-runtimes.md) · [ADR 0004](../adr/0004-secondary-renderers-and-email.md)

## Decision

1. Maintain **one version-controlled, human-edited language file for all in-scope application-owned copy**. Use stable semantic keys and logical feature sections within it. The proposal keeps future locale entries in that same file. Only `en-US` is enabled in production during this initiative. Build test variants from fixtures.
2. Bundle the file directly where the build and renderer support it. Where packaging or runtime syntax differs, derive the required resource from the same file at build time. Derived content is read-only and reproducible. Review edits at the source. The proposal permits generated assets and bytes embedded in independently shipped artifacts.
3. Include migrated help and legal copy as coherent text blocks and include the wording of app-owned emails and reports. Existing code/templates own structure, stable IDs, link destinations, formatting and context-appropriate escaping. Use named placeholders, whole messages and developer-supplied components for rich content. Keep game data, external protocol values, user content and third-party resources in their current ownership boundaries.
4. Review persisted wording as part of scope. Existing system chat text stays in its original records as compatibility data. Current badge descriptions come from an admin write path and a DB-generated frontend snapshot. Use the existing description for compatibility while the maintainers decide when the badge workflow can obtain its wording from the catalog. Treat this as an explicit migration dependency for badge coverage. [Historical chat ADR](../adr/0007-historical-system-message-rendering.md) · [Badge write path](../../pkg/config/service.go#L155) · [Badge generation workflow](../../cmd/gen-badges/main.go#L1)

## Options considered

| Option | Assessment |
| --- | --- |
| Same file imported unchanged by every renderer | Try direct consumption first for each build. Browser and Python can read a common resource syntax. The Go embed path is package-relative, the API image excludes `liwords-ui`, the Compose frontend mounts only `liwords-ui`, and SAM packages `src/`. A direct import in every path requires build layout changes and still leaves template semantics to solve. |
| One authored file, resources bundled from it | **Recommended.** A small deterministic build step can select relevant keys and adapt supported syntax where needed. Builds must package the source revision they were built with. This incurs a build/test cost and keeps renderer dependencies local. |
| Independently maintained frontend, backend, report, help or per-language files | Conflicts with the single-file requirement and makes shared wording and parameters drift. Logical sections in the one file still let reviewers navigate by feature. |
| One physical file fetched at runtime by all consumers | Introduces availability, caching and version coordination across the web app, standalone embed, jobs and reports. A strict one-physical-file interpretation needs a revised feasibility decision for these entry points. |

[Frontend mount](../../docker-compose.yml#L242) · [Docker context exclusions](../../.dockerignore) · [Embed packaging](../../liwords-ui/rsbuild.embed.config.ts) · [Scorecard package](../../services/tourneypdf/template.yaml#L15) · [Go embed package documentation](https://pkg.go.dev/embed#hdr-Directives)

## Implementation criteria

- Use a diffable format with stable ordering and a duplicate-aware check at source ingestion. Validate reference coverage, parameters and plural forms with the selected renderers. A small, explicit supported subset is sufficient for a pilot. Use established rendering libraries for plural rules and interpolation. Choose the runtime, syntax and whether any conversion is worth maintaining after the pilot. i18next and go-i18n provide different plural/resource and interpolation conventions. [i18next plural conventions](https://www.i18next.com/translation-function/plurals) · [go-i18n formats and template data](https://github.com/nicksnyder/go-i18n)
- Validate one plain label, a count with singular/plural output, a named argument containing punctuation and markup characters, a React help block with a controlled link, a Go HTML email, and a Python PDF label. A candidate renderer let a catalog slot override a code-owned link in the isolated fixture. Reject catalog-authored slot attributes and arbitrary markup or use a renderer that enforces code-owned props. Verify missing-key and malformed-input fallback and baseline output parity. Prove the existing main and embed builds, Go API/job builds, and PDF packaging. Verify a catalog-only change triggers every affected check. [Engineering experiment](SOFTWARE-ENGINEERING.md#isolated-proof-run) · [Catalog CI ADR](../adr/0008-catalog-checks-and-copy-review.md) · [Build path filters](../../.github/workflows/build-and-deploy.yml#L61)
- Measure initial browser and embed transfer size against the baseline. Package only the messages a consumer needs when size or context justifies it. Whole paragraphs, stable key order and reviewable batches make a central file easier to maintain as coverage grows.
- Keep semantic outcome IDs and typed data in the application contract. The catalog maps presentation keys to text. For public clients, keep existing fields, codes and English wire values by default. Catalog lookup belongs at the renderer or the verified service boundary.

## Open decisions

- **Requester.** Does “one file” allow mechanically packaged copies and subsets of the sole authored file? Should all later languages live in that one file as proposed? Does the constraint cover legally reviewed long-form copy and future badge authoring in full?
- **Maintainers.** Choose the source location and build integration after the pilot, and decide when to migrate badge authoring. Confirm reviewer ownership and the process for enabling future locales.

[Joint report](REPORT.md) · [Engineering assessment](SOFTWARE-ENGINEERING.md) · [Architecture assessment](SOFTWARE-ARCHITECTURE.md) · [Product assessment](PRODUCT-MANAGEMENT.md)
