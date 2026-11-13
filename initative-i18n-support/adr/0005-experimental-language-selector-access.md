# ADR 0005 Experimental language selector access

Status: Open. Awaiting the repository owner's decision.

Decision owner: Repository owner. The feature requester has explicitly deferred
this choice to the owner.

## Context

The language selector belongs in the experimental settings section and must be
hidden by default. The language preference itself is stored on the account and
follows the user across devices.

Existing experimental settings use browser-local storage. The codebase also has
a database-backed role-based access control system: roles contain permissions,
admins can assign or remove roles, and clients can fetch their effective
permissions. Existing UI helpers treat `admin_all_access` as a wildcard.

The inspected code does not provide a separate general-purpose per-account
feature-flag service. The existing role and permission infrastructure can
provide account-based access to the selector.

The checked-in role definitions do not include a dedicated i18n tester role.

## Option 1 Dedicated I18n Tester role

Recommended, pending the repository owner's decision.

Add a focused permission, provisionally `can_use_experimental_i18n`, and an
`I18n Tester` role that grants only that permission. Seed their definitions
through a database migration and reuse the existing role assignment UI.

Show the language selector only when the account has that permission, using the
existing permission helper. This would also reveal the selector to admins
through their wildcard permission. Ordinary accounts receive no testing role
by default.

This proposal controls selector visibility. It does not disable the i18n
runtime for other users; the complete application still uses `en-US` catalogs.
Only `en-US` is available in production in the first iteration.

Access follows the account across devices, and admins can manage membership
through the existing role assignment UI.

## Option 2 Developer tools flag

Reveal the selector only when a developer-tools flag is enabled in the current
browser. The flag defaults to off and does not follow the account across
devices. It controls visibility only; the selected language is still saved to
the account as required.

This option needs no new testing role or permission, but testers must enable
the flag separately in each browser they use.

## Consequences and open points

These are the two options to present to the repository owner. Neither has been
accepted. Access and language preference remain separate concepts under both
options, and the selector is hidden by default.

The owner must select an option before the selector gate is implemented. If
option 1 is chosen, specify permission refresh behavior for already-open
sessions and confirm administrator visibility. For either option, specify the
effect of losing selector access when an account has a saved language
preference.

## References

- [Clarification decisions](../decisions.md)
- [Role assignment UI](../../liwords-ui/src/admin/perms_and_roles.tsx)
- [Authorization service](../../pkg/auth/authz_service.go)
- [Role and permission queries](../../db/queries/rbac.sql)
- [Seeded roles and permissions](../../db/migrations/202502031726_rbac.up.sql)
- [Frontend permission helpers](../../liwords-ui/src/mod/perms.ts)
- [Existing experimental settings](../../liwords-ui/src/settings/secret.tsx)
