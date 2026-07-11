# Copilot Instructions

This repository is an Ansible-based SSL/TLS certificate lifecycle system.

Use this file for execution rules. Use [README.md](../README.md) for operator setup and full runbook details.

## Immediate Working Mode

- Keep changes minimal, idempotent, and inside existing roles unless the user explicitly asks for new structure.
- Default to Vault-based flows. Use `openssl` only for legacy/specific requests.
- Do not add features, suffixes, or behavior variants without explicit user confirmation.
- Keep task names in gerund form and variable names in snake_case.
- Use English in task names, comments, and docs.

## Critical Safety Rules

- Treat secrets as sensitive by default and use conditional masking:

  ```yaml
  no_log: "{{ not debug | default(true) }}"
  ```

- Do not introduce Python for automation logic. Prefer native Ansible modules.
- Do not use `shell`/`command` when a native module exists.
- Exception: macOS Keychain `security` command is allowed.

## Architecture Boundaries

- Preserve modular role boundaries: `private_key`, `csr`, `certificate`, `pkcs12`, `verify`, `import`.
- Keep certificate verification scoped to file validation (not endpoint probing) unless explicitly requested.
- Avoid hardcoded certificate paths. Respect dynamic path resolution from the PKI orchestration.
- Maintain expected certificate path pattern:

  `/opt/certificates/{year}/{domain}/{environment}/{certificate_name}`

## Configuration Integrity

- When changing any variable in `vars/config.yml`, mirror the same structural change in:
1. `extra-vars.yml`
2. `roles/pki/templates/config.yml.j2`
3. `roles/pki/tasks/convert_config_from_older_version.yml`
- Keep these files synchronized to prevent config drift.
- Preserve namespaced merge model:
1. Context merge to `context_vars`
2. Config merge to `config_vars`
- In source config/context files loaded with `name:`, reference same-file variables using that namespace.

## Include Pattern Enforcement

- For cross-role task calls, use `ansible.builtin.include_role` with `tasks_from`.
- Do not use relative `include_tasks` to reach other roles.

Accepted pattern:

```yaml
ansible.builtin.include_role:
  name: "metadata"
  tasks_from: "read_metadata.yml"
```

## Command Quickstart For Agents

- Prepare controller: `ansible-playbook prepare_controller.yml -i inventory/hosts -K`
- Full certificate pipeline: `ansible-playbook create_all.yml -e @extra-vars.yml`
- Verification run: `ansible-playbook verify_certificate.yml -e @extra-vars.yml`
- Import run: `ansible-playbook import_certificate.yml -e @extra-vars.yml`

Use [README.md](../README.md) for prerequisites, environment variables, and extended execution examples.

## Project-Specific Pitfalls

- Certificate directories prefixed with `_` are intentionally excluded.
- Password semantics must remain stable:
1. `"?"` means auto-generate and store.
2. `""` means no password and do not store.
3. Explicit value means use as provided and store.
- For config fallback behavior, avoid broad `default(omit, true)` on standard fields.

## Related Docs

- Operational runbook: [README.md](../README.md)
- Shared role helpers: [roles/common/README.md](../roles/common/README.md)
