# US English wording correction audit

Only corrections of obvious mistakes are permitted during the i18n migration.
Existing wording otherwise stays unchanged. Every correction must be explicit
here and reviewed by the maintaining team; a proposed correction does not
count as reviewed.

No application wording corrections have been proposed or applied during this
planning task.

## Correction entries

Add one entry per wording correction. Preserve the exact before and after
text, including relevant punctuation and placeholders. If one corrected
catalog entry is used in several places, list the affected locations or uses.

| ID | Source location or message key | Original wording | Corrected wording | Obvious mistake and reason | Review status | Maintaining-team reviewer and review reference |
| --- | --- | --- | --- | --- | --- | --- |

Use `Pending review`, `Reviewed` or `Rejected` as the review status. Start new
entries as `Pending review`. Mark an entry `Reviewed` only after the maintaining
team has reviewed it and a review reference is recorded. Keep rejected entries
and their review outcome rather than removing the audit trail.

## Review requirements

- Verify that the change corrects an obvious mistake and preserves the intended
  meaning.
- Keep stylistic rewrites and broader terminology changes outside this migration.
- Check placeholders, plural forms, links and rich-text markup where affected.
- Record all source-language wording corrections, including corrections in UI
  catalogs, help pages, badges, user-facing backend messages, emails and other
  in-scope rendering surfaces.

Moving unchanged wording into a catalog is not a copy correction. Track
structural changes such as named interpolation and plural handling in the
implementation review; add an audit entry whenever the resulting English
wording changes beyond the required locale-formatting behavior.
