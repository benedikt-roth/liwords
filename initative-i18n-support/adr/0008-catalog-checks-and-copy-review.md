# ADR 0008 Catalog checks and copy review

Status: Required checks and copy review confirmed. Tool selection and future
contributor/language-release governance remain open for the repository owner.

## Context

Shipping only English can conceal missed extraction, missing keys, incorrect
plural handling and untested alternate rendering paths. Multiple renderers also
create a risk of mismatched message IDs or parameters. A source-English wording
change needs human review even when a catalog compiles successfully.

## Confirmed requirements

- Automated catalog checks run in the first iteration.
- Production supports only `en-US` after migration; future community languages
  must be possible without another extraction migration.
- Only obvious mistakes may be corrected in existing English. Every wording
  correction must be explicit in an audit reviewed by the maintaining team.

## Proposed CI checks

1. Parse catalog source with duplicate-key detection and compile it for the
   selected renderer. Do not rely on a JSON parser that silently drops duplicate
   entries to detect them after the fact.
2. Validate referenced keys and registered backend code mappings against complete
   `en-US` resources. Require documented exceptions for dynamic key construction
   or content/protocol literals that are intentionally not translations.
3. Validate named placeholders, types required by message contracts, plural forms
   and supported markup/component structure. Reject parameters that disappear
   or unsupported markup introduced by a translation.
4. Validate generated resources and shared identities across chosen runtimes;
   detect stale generation or inconsistent output from a shared source.
5. Check the production supported-language registry and bundles so a development
   pseudolocale or unfinished catalog does not become a shipped language.
6. Report source-English wording changes for audit review. Automation can check
   references and entry completeness; it cannot impersonate maintaining-team
   review or decide whether every wording change is an obvious mistake.

The owner selects tooling after ADR 0002 chooses catalog formats. Fit checks
into existing frontend/backend CI without requiring unrelated deployments or
external message delivery. Apply checks to the in-scope resources; do not make
excluded Discord templates or hidden integration logs extraction targets.

## Copy review

Use [the audit log](../english-copy-audit.md). Each correction records exact
before/after text, location/key, reason, status, reviewer and review reference.
Start corrections pending review and preserve rejected entries. A generated
catalog diff or automated test pass does not constitute team review.

Distinguish structural interpolation/plural changes from wording corrections,
while still reviewing the resulting English. Preserve legal/help wording and
other existing text except for the permitted corrections. Locale-driven date
or number formatting follows the separate confirmed formatting requirement.

## Community workflow and open governance

A documented GitHub PR contribution flow is a reasonable proposal, but only the
automated checks were explicitly confirmed during clarification. The maintaining
team must select review ownership, coverage thresholds, incomplete-catalog
fallback and the criteria for enabling a future language. Passing catalog checks
alone must not change the production language registry.

No additional language or hosted translation platform is needed for this
iteration. A development-only fixture can exercise plural rules, placeholders,
language changes and text expansion without becoming a supported locale.

## Validation

Seed representative invalid-resource fixtures and show that CI reports syntax,
duplicate keys, missing mappings, placeholder/plural errors, unsupported markup
and accidental locale registration. Use actual renderer validation rather than
a second hand-written template parser. Review the full surface inventory for
text outside JSX; catalog checks cannot prove extraction coverage on their own.

## References

- [Feature specification](../feature-specification.md)
- [Clarification decisions 9 and 10](../decisions.md)
- [Copy audit](../english-copy-audit.md)
- [Message/catalog runtime decision](0002-message-contracts-and-catalog-runtimes.md)
- [Current frontend build and tests](../../liwords-ui/package.json)
- [Current CI workflow](../../.github/workflows/build-and-deploy.yml)
