# Copilot Instructions

This is an Ansible-based certificate management system for automated SSL/TLS certificate lifecycle management. Your primary role is to help maintain project consistency and prevent deviations from established architecture.

## Project Discipline Guidelines

**VAULT-ONLY APPROACH**: Although code supports multiple providers for historical reasons, all new development should use Vault only. Consider if `openssl` provider are truly necessary for the specific use case.

**VARIABLE CONSISTENCY**: Prefer using existing variable names. When creating new variables is necessary, ensure they follow established naming patterns and don't duplicate existing functionality.

**ROLE STRUCTURE**: Follow the established modular role pattern. Assess whether new tasks can fit within existing roles before creating monolithic playbooks or bypassing the role-based architecture.

**PATH PATTERNS**: Follow `/opt/certificates/{year}/{domain}/{environment}/{certificate_name}` unless there's a compelling technical reason for deviation.

## Architecture Overview

**Modular PKI System**: Each PKI operation is handled by dedicated roles (`private_key`, `csr`, `certificate`, `pkcs12`, `verify`, `import`). Consider whether new tasks can be integrated into existing roles.

**Vault PKI Integration**: Uses HashiCorp Vault PKI engine exclusively for certificate issuance. All certificate operations should go through Vault APIs unless there's a specific technical requirement.

**Centralized Certificate Management**: The CertificateAuthorityServer acts as a centralized certificate repository storing certificates for all managed servers. All certificate lifecycle operations (creation, verification, storage) execute on this server. Only the import role deploys certificates to target servers.

**Certificate Verification Approach**: The `verify` role validates certificate files directly (expiration dates, validity periods) rather than performing HTTP/HTTPS endpoint checks. It verifies the certificate artifacts themselves, not deployed service endpoints.

**Dynamic Path Resolution**: Certificate paths are dynamically resolved using regex patterns in `roles/pki/tasks/main.yml`. Avoid hardcoding paths when possible.

## Required Workflows

**Certificate Lifecycle Pipeline**:
```bash
# Individual steps (only when needed for debugging).
ansible-playbook verify_certificate.yml
ansible-playbook create_private_key.yml
ansible-playbook create_csr.yml
ansible-playbook create_certificate.yml
ansible-playbook create_pkcs12.yml

# Complete pipeline (preferred).
ansible-playbook create_all.yml

# Import to target servers.
ansible-playbook import_certificate.yml
```

**Daily Verification Process**: A cronjob runs `verify_certificate.yml` daily to check certificate expiration on stored certificate files.

**Configuration Management**: Each certificate uses `{certificate_path}/config.yml`. System auto-generates from `roles/pki/templates/config-vault.j2` when missing. Consider whether existing config patterns can meet new requirements.

**Vault Integration**: All secrets use pattern `Certificates/{domain}/{environment}/{certificate_name}`. Avoid hardcoding passwords when possible.

## Strict Development Guidelines

**Environment Setup**: Always require these environment variables:
```bash
export ANSIBLE_HASHI_VAULT_ADDR="https://vault.lan.homelab"
export ANSIBLE_HASHI_VAULT_AUTH_METHOD="ldap"
export ANSIBLE_HASHI_VAULT_USERNAME="usr_ansible_pd"
```

**Variable Usage**: Prefer existing variables from `vars/config.yml`. Common required variables:
- `certificate_domain`: Target domain from approved list.
- `certificate_environment`: "stg" | "prd" only.
- `certificate_signing_provider:` "vault" (recommended default).
- `certificates`: Array for specific processing, `[]` for all.

**File Organization**: Use personal config files as `extra-vars-{username}.yml`. Avoid modifying shared configuration files directly.

**Inventory Rules**: Use correct inventory files:
- `inventory/hosts`: Production environment.
- Avoid hardcoding hostnames in playbooks when possible.

**Import Templates**: Application-specific import configurations in `roles/import/templates/` for `SpringBoot`, `Tomcat`, `Wildfly`, etc. Each template defines service paths, permissions, and restart procedures.

## Code Standards

**No Python Code**: This is a pure Ansible project. Do not suggest Python filters, plugins, or scripts. All solutions must use native Ansible features (Jinja2 templates, filters, lookups, modules).

**No Shell Commands**: NEVER use `ansible.builtin.shell` or `ansible.builtin.command` modules. These violate project standards. Use native Ansible modules from `ansible.builtin.*` or community collections only. The `command` module is only acceptable in exceptional cases with explicit justification.

**Language**: All code must be in English - variable names, comments, task names, file names, and documentation.

**Task Naming**: Use gerund form for task names (e.g., "Copying config file", "Creating certificate directory") rather than imperative form ("Copy config file", "Create certificate directory").

**Task Structure**: Follow this order for task attributes:
```yaml
- name: "Task description in gerund form"
  when: condition_here
  ansible.builtin.module_name:
    parameter: "value"
  vars:
    variable_name: "value"
  register: "output_variable"
  notify: "handler name"
```

**Variable Naming**: Use snake_case with descriptive prefixes that indicate scope (e.g., `certificate_path`, `vault_response`, `private_key_passphrase`).

**File Paths**: Use double quotes for all file paths and string values. Use absolute paths when referencing files across roles.

**Loop Control**: Always use `loop_control` with meaningful `loop_var` and `label` for readability:
```yaml
loop: "{{ certificate_list }}"
loop_control:
  loop_var: "certificate_item"
  label: "{{ certificate_item.name }}"
```

**Debug Output**: Use consistent debug pattern for troubleshooting messages:
```yaml
- name: "Printing certificate variables"
  when: debug | bool
  ansible.builtin.debug:
    msg: "{{ variable_content }}"
```

**Sensitive Data**: For tasks handling passwords or secrets, use conditional no-log:
```yaml
- name: "Creating password variable"
  no_log: "{{ not debug | default(true) }}"
  ansible.builtin.set_fact:
    passphrase: "{{ generated_password }}"
```

## Critical Patterns

**Role Inclusion Logic**: The `roles/pki/tasks/steps.yml` dynamically includes roles based on `roles_to_include` variable, defaulting to the complete PKI pipeline.

**Vault PKI Paths**: Vault operations use dynamic paths like `{vault_pki_name}/issue/{vault_pki_role}` where PKI engines and roles are environment-specific (e.g., "icp-cert-sub/issue/server-sign").

**Certificate Exclusions**: Folders prefixed with `_` are automatically excluded from processing. Backup paths and incomplete directory structures are filtered out in the main processing loop.

**Password Handling**: Private key passphrases use marker `"?"` for auto-generation, empty string for no password, or explicit values. ALL generated and user-provided passwords are stored in the secret manager, EXCEPT empty strings. When passphrase is `"?"`, new keys are generated on every run (non-idempotent by design) to allow forced regeneration.

**Legacy Compatibility**: PKCS12 files support legacy encryption via `pkcs12_encryption: "compatibility2022"` for older systems compatibility.

When modifying this system, always consider the multi-environment nature, ensure Vault integration remains secure, and maintain the modular role structure for maintainability.
