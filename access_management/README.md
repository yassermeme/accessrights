# Access Management (Odoo 19)

`access_management` is a **restrictive overlay** for Odoo security. It is
intentionally conservative: it never grants an operation that native ACLs or
record rules reject, and it does not alter existing groups, ACLs, record rules,
menus, views, or data.

## Policy

* Only active profiles assigned to the current user apply. Recovery
  administrators and the Odoo superuser are exempt, so they can deactivate a
  profile without deleting it.
* An unchecked Model Access permission is an explicit denial when **Enforce**
  is checked. Checked permissions never grant native access.
* Matching active profiles are combined with **deny-wins** semantics. Each
  enabled domain rule is an additional AND restriction for its selected
  operation.
* Read-only and Global restrictions are server-enforced only for models named
  in **Global Model Scope**; an empty scope is deliberately a no-op. This avoids
  an accidental installation-wide lockout.
* Field rules remove invisible/non-readable fields from metadata and reject
  explicit `read()` requests; read-only/non-writable fields reject writes.
  This covers normal ORM/RPC calls but cannot guarantee confidentiality when a
  separate custom controller reads data using `sudo()`.

## Safe enforcement boundary

The module uses supported `base` model inheritance to call native
`check_access_rights`/`check_access_rule` first and then raise an extra
`AccessError`. It enforces model create/write/delete, field reads/writes, and
record access for browsed/read/written/deleted records. Native record rules
remain authoritative.

Odoo does not expose one stable generic server authorization hook for arbitrary
object-button methods, arbitrary controllers, imports/exports, mail routes, or
all menu/action frontend paths. Therefore button/tab, search-panel, chatter,
import/export, developer-mode, and menu rules are **configuration inventory
and UI integration targets**, not claimed as generic security controls in this
foundation. Secure a specific custom button/controller by calling
`record.check_access_rights(operation)` and `record.check_access_rule(operation)`
before executing business logic, or by adding a small model-specific hook.
Do not use UI hiding to protect sensitive data.

## Installation / recovery

1. Place this directory on `addons_path` and update the Apps list.
2. Install **Access Management**. Assign configuration access only to trusted
   users through **Access Management Manager**.
3. Assign at least one trusted user **Access Management Recovery
   Administrator** before activating restrictions. The profile constraint
   prevents assigning that group to scoped global/read-only profiles.
4. To recover, log in as that recovery administrator, archive (`active=False`)
   the profile, then adjust it. Configuration is retained and native permissions
   resume immediately after cache invalidation.

## Limitations

Domain validation deliberately accepts only literal list domains (`ast.literal_eval`);
context variables and Python expressions are refused. Domain constraints are
validated during record access. Search aggregation and custom `sudo()` code may
need model-specific integration. Test restrictions in a disposable database
with representative workflows before production use.
