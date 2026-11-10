# DRAFT - ojuny-fleet-manager

Status: draft scope. Nothing is built. This file describes what the repository
is supposed to contain, not what it contains today.

## Purpose

Ojuny Fleet Manager (OFM), the web interface for `ojuny-fleet-controller`. It is
the one place an administrator manages users, groups, endpoints, servers,
applications, policies, templates, deployments and logs.

OFM is the enterprise-geared version of `nix-web-gui` (NWG). Both reuse the
`nix-settings-gui` backend directly, so OFM inherits the same comment-annotated
Nix model, scanner, metadata lookup, span-patch write path, formatter and
descriptor sidecars. OFM adds fleet-specific pages, role-based access and remote
actions on top of that shared backend.

## What the repository will contain

```text
ojuny-fleet-manager/
  flake.nix                  package, module and app outputs
  server/                    HTTP API over the shared nix-settings-gui backend
    fleet/                   fleet entities, policies and controller client
    deploy/                  deployment queue and controller calls
    auth/                    Nix-declared accounts, roles and audit
  frontend/                  browser interface
    pages/                   users, groups, endpoints, servers, applications
    components/              editors, tables, diffs and dialogs
  docs/                      API, roles, page reference, workflows
  tests/                     backend, frontend and end-to-end checks
```

### Pages

- Users: list, add, edit details, group membership, user-bound policies and
  per-user settings.
- Groups: list, add, edit details, settings and group-bound policies.
- Endpoints: list, add, edit the NixOS declaration, hardware data and
  device-bound policies; deploy, live statistics and remote SSH.
- Servers: the same model with a server-oriented default structure and
  container configuration.
- Applications: list, add, package lists and application-bound policies.
- Policies: Nix files that define rules for system, user, device or
  application settings; binding and precedence views.
- Templates: template Nix files that can be filled in for a new endpoint.
- Settings: controller settings, flake inputs, cache, VPN and certificate
  state.
- Logs: controller, deployment, access and audit logs.
- Deploy: build and download pre-addressed live ISOs, queue deployments and
  watch progress.

### Editing model

- Every entity is a Nix file or a set of Nix files. The UI edits the file, not
  a database record that is later exported.
- `@menu` and `@option` annotations drive pages and controls exactly as in
  `nix-settings-gui`.
- `@option` comments attach rendering metadata to an assignment. They do not
  declare the option.
- Options must be declared at the top level of a module's `options` attribute.
  Options inside `lib.mkMerge`, a list of attribute sets, `map`, `forEach` or
  `lib.optionalAttrs` are not reliably visible and must not be used.
- A freeform value that is not a module option uses a descriptor sidecar.
- Every page offers "edit raw Nix" for the whole file.
- Saves are all-or-nothing: parse check with `rnix`, format with `nixfmt`,
  write atomically, then commit.
- A diff preview is shown before save and before deploy.
- The scanner is not modified to descend into lists or apply nodes. The fix for
  an invisible option is always to change the module shape.

### Deployment

- A deploy action builds the endpoint's configuration, shows the build result
  and queues a push through the controller.
- Deploy-rs is used for an existing NixOS target; nixos-anywhere is used for a
  machine that boots the prepared live ISO.
- The deploy page can generate install media for several targets at once. Each
  ISO carries a unique identifier and a controller address. Reusing the same
  USB for a different target is supported by retargeting, not reflashing.
- Live statistics and a remote SSH action are available per endpoint when the
  endpoint and controller allow it.
- Failed builds and failed deployments are recorded with logs and the previous
  generation remains bootable.

### Authentication and audit

- Account and role declarations are shared with the controller through Nix.
  There is no SSO, LDAP or Kerberos dependency.
- Endpoint actions require an explicit role and are audited.
- Secrets are never rendered into the browser, logs or diffs.
- Read-only mode can be used for review and audit.

### Documentation

- The page reference and each entity's file shape.
- Roles and what each role can read and change.
- The edit, save, diff and deploy flow.
- Descriptor sidecar usage for freeform settings.
- The module shape rules and why they are binding.
- The controller API contract and versioning.

### Tests

- Backend scanner and parser tests against annotated fixtures.
- Metadata lookup with nixd and the eval fallback.
- Span-patch write, parse check and format tests.
- Role and permission tests for every action.
- A full flow: add a user, bind a policy, edit an endpoint, save, build and
  queue a deployment in a test controller.
- A rejected save leaves the working tree unchanged.
- A list-nested option is reported as unsupported rather than silently edited.

## Limits and non-goals

- Not a general-purpose MDM and not a Microsoft Intune clone.
- No Windows endpoint management.
- No database as the source of truth. Git and Nix files are authoritative.
- No editing outside the managed configuration root.
- No automatic merge of conflicting changes. Conflicts are shown to a person.
- No AI-written policies and no automatic remediation.
- No public multi-tenant service. One controller manages one fleet.
- No SSO, LDAP or Kerberos integration. Accounts and roles are declared in Nix.
- The same option-shape limitations as `nix-settings-gui`.

## Open questions

- How OFM's fleet pages extend the shared NWG frontend without forking it.
- Frontend stack and whether server-side rendering is required.
- How conflicts between two administrators are resolved.
- Whether policies are evaluated on the controller or on the endpoint.
- How live statistics are collected: push agent, SSH, or controller poll.
- Whether remote SSH opens a browser terminal or hands off to a local client.

## Sources

- `nixos-configs/notes/ojuny pro - early draft.md`, web panel section.
- `nix-settings-gui/docs/architecture.md`, `docs/comment-syntax.md`,
  `docs/descriptor-schema.md`.
- `ojuny-template/docs/05-settings-gui-limitations.md`.
