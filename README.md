# Verify, Create, and Import Certificates

This set of playbooks can verify if the certificate created by our own Certificate Authority (CA) is still valid, create new certificates if necessary, and import certificates into multiple applications.

## Requirements

- The Ansible server must be running a Linux-based OS (tested on Oracle Linux and Ubuntu).

## Configuration File

Each certificate can have its own configuration file. If no specific config file is provided, the playbook will use the [default config file](vars/config.yml).

## How to Execute

1. Install the required roles and collections on the AWX server:

    ```bash
    ansible-playbook prepare_host.yml --limit "server"
    ansible-playbook prepare_host.yml -i inventory/hosts --limit "server"
    ```

1. Generate private keys, CSRs, certificates, and PKCS12 bundles:

    ```bash
    ansible-playbook create_private_key.yml
    ansible-playbook create_csr.yml
    ansible-playbook create_certificate.yml
    ansible-playbook create_pkcs12.yml
    ```

    Or, you can run all these steps in a single playbook:

    ```bash
    ansible-playbook create_certificate_vault.yml
    ansible-playbook create_certificate_windows.yml
    ```

1. Verify a certificate:

    ```bash
    ansible-playbook verify.yml
    ```

1. Import existing certificates:

    ```bash
    ansible-playbook import.yml
    ```

1. Install CA certificates on target servers:

    ```bash
    ansible-playbook install_ca.yml --limit "server"
    ansible-playbook install_ca.yml --tags "all"
    ansible-playbook install_ca.yml --tags "os"
    ansible-playbook install_ca.yml --tags "python"
    ansible-playbook install_ca.yml --tags "os,python"
    ```

## Variables

### 1. Signing Method

Specifies the method used to sign the certificate.

Options:
- `"vault"`: Use HashiCorp Vault.
- `"windows-server"`: Use a Windows Server CA (**default**).
- `"openssl"`: Use the local OpenSSL tool to sign the certificate.
  - `"selfsigned"`: Creates a self-signed certificate, meaning the certificate will be signed by the entity that created it (not by an external or trusted CA)..
  - `"ownca"`: Uses a custom Certificate Authority (CA) defined within the playbook or role.

```json
--extra-vars='{ "certificate_signing_method": "openssl", "certificate_provider": "selfsigned" }'
--extra-vars='{ "certificate_signing_method": "openssl", "certificate_provider": "ownca" }'
--extra-vars='{ "certificate_signing_method": "vault" }'
```

### 2. Domain

Specifies the domain name for the certificate (e.g., `lan.homelab`).

```json
--extra-vars='{ "certificate_domain":"lan.homelab" }'
```

### 3. Environment

Specifies the environment for the certificate (e.g., `hg` or `pd`).

```json
--extra-vars='{ "certificate_environment":"pd" }'
```

### 4. Certificates

Defines which certificates should be verified, created, or imported. If no value is provided, all certificates will be processed.

```json
--extra-vars='{ "certificates":["pbes321", "pbes123.lan.homelab"] }'
```

### 5. Debug

Enables or disables additional debug messages. Defaults to **False**.

```json
--extra-vars='{ "debug":True }'
```

### 6. All Together

Example of combining multiple variables in one command:

```json
ansible-playbook create_private_key.yml --extra-vars='{ "certificate_signing_method": "vault", "certificate_domain":"lan.homelab", "certificate_environment":"pd", "certificates":["pbes123"] }' -i inventory/hosts
```

```json
ansible-playbook create_csr.yml --extra-vars='{ "certificate_signing_method": "windows-server", "certificate_domain":"lan.homelab", "certificate_environment":"hg", "certificates":["hbes123"], "debug":True }' -i inventory/hosts-hg
```

```json
ansible-playbook create_certificate.yml --extra-vars='{ "certificate_signing_method": "openssl", "certificate_provider": "selfsigned", "certificate_domain":"lan.homelab", "certificate_environment":"pd", "certificates":["pbes123"], "debug":True }' -i inventory/hosts
```

```json
ansible-playbook create_pkcs12.yml --extra-vars='{ "certificate_domain":"lan.homelab", "certificate_environment":"hg", "certificates":["teste1", "teste2.lan.homelab"], "debug":True }' -i inventory/hosts-hg
```

```json
ansible-playbook create_certificate_vault.yml --extra-vars='{ "certificate_domain":"lan.homelab", "certificate_environment":"hg", "certificates":["hbes123", "hbes321.lan.homelab"], "debug":False }' -i inventory/hosts-hg
```

```json
ansible-playbook create_certificate_windows.yml --extra-vars='{ "certificate_domain":"lan.homelab", "certificate_environment":"hg", "certificates":["hbes123", "hbes321.lan.homelab"], "debug":False }' -i inventory/hosts-hg
```

```json
ansible-playbook verify.yml --extra-vars='{ "path": "/opt/certificates/", "debug":False }' -i inventory/hosts
```

```json
ansible-playbook import.yml --extra-vars='{ "certificate_domain":"lan.homelab", "certificate_environment":"hg", "certificates":["hbes1617", "sicwebhg"], "debug":True }' -i inventory/hosts-hg
```

### 6.1 Using extra-vars from file

Example using variables from file at command:

Create extra-vars.yml and declare values
```yaml
# extra-vars.yml
certificate_domain: "lan.homelab"
certificate_environment: "hg"
certificates: ["hbes123", "hbes321.lan.homelab"]
debug: "false"
```

```json
ansible-playbook create_certificate_vault.yml -e extra-vars.yml -i inventory/hosts-hg
```

## Exclude Folders

To exclude specific folders from execution, prepend an **underscore** ("_") to the folder name.

Example:

```bash
mv domain.com.br _domain.com.br
```

## For legacy pkcs12 encryption - Windows 2016 and older

Just declare variable pkcs12_encryption: 'compatibility2022'

Example:

```json
ansible-playbook create_certificate_vault.yml --extra-vars='{ "certificate_domain":"lan.homelab", "certificate_environment":"hg", "certificates":["teste1", "teste2.lan.homelab"], "pkcs12_encryption": "compatibility2022", "debug":True }'
```
This option will create pkcs12 with legacy encryption.

## Check the Files

### 1. Check a Private Key

```bash
openssl rsa -in certificate.cer -check -passin pass:<private_key_password>
```

### 2. Check a Certificate Signing Request (CSR)

```bash
openssl req -text -noout -verify -in certificate.csr
```

### 3. Check a Certificate

```bash
openssl x509 -text -noout -in certificate.cer
```

### 4. Check a PKCS#12 File (.pfx or .p12)

```bash
openssl pkcs12 -info -in certificate.pfx -passin pass:<private_key_password>
```

## Supported Import Types

1. [Springboot](roles/import/templates/springboot.yml)
1. [Traefik](roles/import/templates/traefik.yml)
1. [Wildfly Linux](roles/import/templates/wildfly-linux.yml)

**Created by:**

1. Luciano Sampaio
