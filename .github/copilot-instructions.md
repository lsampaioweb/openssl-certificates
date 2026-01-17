# Copilot Instructions

This is an Ansible-based certificate management system for automated SSL/TLS certificate lifecycle management. Your primary role is to help maintain project consistency and prevent deviations from established architecture.

## Mentoring Philosophy

**Ruthless Code Review**: Focus on semantic clarity, idempotency, consistency, error handling, and security (no_log usage, file permissions).

**Incremental Testing**: When implementing new roles, start with variable setup, comment out complex operations, test each section independently, then verify end-to-end integration.

**Variable Contamination Prevention**: Each role MUST reset its output variables at the start:

```yaml
- name: "Resetting role-specific variables"
  ansible.builtin.set_fact:
    role_creation_output: {}
    role_specific_path: ""
```

**Review Focus**: Task names in gerund form, proper no_log for sensitive data, meaningful variable names, idempotent operations.

**Question Assumptions**: Challenge design decisions if:
- New variables duplicate existing functionality.
- Tasks could fit in existing roles instead of creating new ones.
- Hardcoded values could be configurable.
- Complex conditionals could be simplified.
- Error handling is missing or inadequate.
- **NEVER add features, suffixes, or variations without explicit user confirmation**. Ask first, implement after approval.

## Project Discipline

**VAULT** and **OPENSSL** support: All new development should default to Vault. However, the `openssl` provider can be used for legacy systems or specific use cases.

**Variable/Role Consistency**: Use existing variable names and patterns. Fit new tasks into existing roles rather than creating new ones.

**Configuration File Synchronization**: CRITICAL - When ANY variable is added, updated, or removed from `vars/config.yml`, the SAME change MUST be applied to:
1. `extra-vars.yml` - Runtime override template (commented out)
2. `roles/pki/templates/config.yml.j2` - Certificate-specific config template
3. `roles/pki/tasks/convert_config_from_older_version.yml` - Legacy format migration mappings

These files must NEVER drift! All four define the same variable structure. The conversion file ensures backward compatibility by mapping old flat-format variables to the current nested structure.

**Path Pattern**: `/opt/certificates/{year}/{domain}/{environment}/{certificate_name}`

## Architecture

**Modular Roles**: `private_key`, `csr`, `certificate`, `pkcs12`, `verify`, `import` - each handles one PKI operation.

**Vault PKI Integration**: Uses HashiCorp Vault PKI engine exclusively for certificate issuance. All certificate operations go through Vault APIs. Vault operations use dynamic paths like `{vault_pki_name}/issue/{vault_pki_role}` where PKI engines and roles are environment-specific.

**Centralized Management**: CertificateAuthorityServer stores all certificates. Only the `import` role deploys to target servers.

**Dynamic Path Resolution**: Certificate paths are dynamically resolved using regex patterns in `roles/pki/tasks/main.yml`. Avoid hardcoding paths when possible.

**Verification**: The `verify` role validates certificate files (expiration, validity), not deployed service endpoints.

## Workflows

**Certificate Pipeline**:
```bash
# Complete pipeline (preferred).
ansible-playbook create_all.yml -e @extra-vars.yml

# Individual steps (debugging only).
ansible-playbook create_private_key.yml -e @extra-vars.yml
ansible-playbook create_csr.yml -e @extra-vars.yml
ansible-playbook create_certificate.yml -e @extra-vars.yml
ansible-playbook create_pkcs12.yml -e @extra-vars.yml
ansible-playbook verify_certificate.yml -e @extra-vars.yml

# Import to target servers.
ansible-playbook import_certificate.yml -e @extra-vars.yml
```

**Daily Verification**: Cronjob runs `verify_certificate.yml` to check certificate expiration.

**Configuration Management**: 4-layer hierarchy guarantees all variables are defined:

1. Base (`vars/config.yml`) - ALL defaults
2. Domain-specific (`vars/config-{domain}.yml`) - Domain overrides
3. Certificate-specific (`{certificate_path}/config.yml`) - Per-cert customization
4. Extra vars (`-e @extra-vars.yml`) - Runtime overrides

Configs merge: base → domain → certificate → extra_vars. Final result in `config_vars`.

**CRITICAL**: ALL config options defined in base `config.yml` will NEVER be undefined. Do NOT use `default(omit, true)` for standard fields. Use `default()` only for:

- Fallback substitution: `{{ config_vars.csr.common_name | default(certificate_name, true) }}`
- Truly optional parameters that should be omitted

**Secrets**: Pattern `Certificates/{domain}/{environment}/{certificate_name}`

## Development Guidelines

**Environment Variables** (required):

```bash
export ANSIBLE_HASHI_VAULT_ADDR="https://vault.lan.homelab"
export ANSIBLE_HASHI_VAULT_AUTH_METHOD="ldap"
export ANSIBLE_HASHI_VAULT_USERNAME="usr_ansible_pd"
```

**Key Variables**: `certificate_domain`, `certificate_environment` (stg|prd), `certificates` (array or [] for all)

**File Organization**: Use `extra-vars-{username}.yml` for personal configs.

**Import Templates**: Application-specific import configurations in `roles/import/templates/` for `SpringBoot`, `Tomcat`, `Wildfly`, etc. Each template defines service paths, permissions, and restart procedures for specific application types.

## Code Standards

**Pure Ansible**: No Python code. No `shell`/`command` modules except macOS Keychain (`security` command - no native module).

**Language**: English only.

**Naming**: Task names in gerund form ("Creating..." not "Create..."). Variables use snake_case with scope prefixes.

**Sensitive Data**: Use conditional no_log:

```yaml
no_log: "{{ not debug | default(true) }}"
```

## Critical Patterns

**Role Inclusion**: `roles/pki/tasks/steps.yml` includes roles based on `roles_to_include` variable.

**Certificate Exclusions**: Folders prefixed with `_` are excluded. Backup paths filtered out.

**Password Handling**:

- `"?"` = auto-generate (stored in secret manager, non-idempotent by design)
- `""` = no password (NOT stored)
- Explicit value = use as-is (stored)
