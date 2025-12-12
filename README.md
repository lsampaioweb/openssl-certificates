# Verify, Create, and Import Certificates

This set of playbooks can verify if the certificate created by our own Certificate Authority (CA) is still valid, create new certificates if necessary, and import certificates into multiple applications.

## Requirements

- The Ansible server must be running a Linux-based OS (tested on Oracle Linux and Ubuntu).

## Extra Variables from the command line

1. Debug:

    Enables or disables additional debug messages. Defaults to **false**.

    ```json
    --extra-vars='{ "debug":true }'
    ```

1. Domain:

    Specifies the domain context for the certificate (e.g., `home` or `homelab`). Defaults to **home**.

    ```json
    --extra-vars='{ "domain_context":"home" }'
    ```

1. Environment:

    Specifies the environment for the certificate (e.g., `stg` for Staging or `prd` for Production). Defaults to **prd**.

    ```json
    --extra-vars='{ "certificate_environment":"prd" }'
    ```

1. Certificates:

    Defines which certificates should be verified, created, or imported. If no value is provided, all certificates will be processed. Defaults to `[]` to process all certificates found in the folder structure.

    ```json
    --extra-vars='{ "certificates":["gitea", "minio"] }'
    ```

## Extra Variables from a file

You can also provide extra variables using a YAML file. For example, to create a file named `extra-vars.yml` with the following content:

```yaml
debug: true
domain_context: "home"
certificate_environment: "prd"
certificates: ["proxmox", "grafana", "zabbix"]
```

## Exclude Folders

To exclude specific folders from execution, prepend an **underscore** ("_") to the folder name.

- These folders will be processed:

  `proxmox`, `grafana`, `zabbix`

- These folders will be excluded:

  `_backup`, `_old`

## Config Variables

1. Signing Method (certificate_signing_provider):

    Specifies the method used to sign the certificate.

    Options:
    - `vault`: Use HashiCorp Vault  (**default**).
    - `openssl`: Use the local OpenSSL tool to sign the certificate.
      - `selfsigned`: Creates a self-signed certificate, meaning the certificate will be signed by the entity that created it (not by an external or trusted CA)..
      - `ownca`: Uses a custom Certificate Authority (CA) defined within the playbook or role.

1. Pkcs12 Encryption (pkcs12_encryption):

    Specifies the encryption method used for PKCS#12 files.

    Options:
    - `auto`: Automatically select the encryption method based on the OpenSSL version (**default**).
    - `compatibility2022`: For Windows servers prior to Windows Server 2022.

## Supported Import Types

1. [Traefik](roles/import/templates/traefik.yml)
1. [Springboot](roles/import/templates/springboot.yml)

## How to Execute

1. Install the required roles and collections on the `Ansible Controller` server:

    Choose the inventory file, the default is `inventory/hosts`:
    ```bash
    ansible-playbook prepare_controller.yml -i inventory/hosts
    ```

    Specify the server name in the `--limit` parameter to prepare only that host, or omit it to prepare all hosts in the inventory:
    ```bash
    ansible-playbook prepare_controller.yml --limit "server-name"
    ```

    To run the playbook with specific tags, use the `--tags` option. For example, to run only untagged tasks and dev_environment tasks:
    ```bash
    ansible-playbook prepare_controller.yml --tags "untagged,dev_environment"
    ```

1. Set environment variables for HashiCorp Vault access:

    ```bash
    nano ~/.bashrc

    export ANSIBLE_HASHI_VAULT_ADDR="https://vault.lan.homelab"
    export ANSIBLE_HASHI_VAULT_AUTH_METHOD="ldap"
    export ANSIBLE_HASHI_VAULT_USERNAME="usr_ansible_pd"
    export ANSIBLE_HASHI_VAULT_PASSWORD="$(secret-tool lookup secret 'ansible_hashi_vault_password')"

    source ~/.bashrc
    ```

1. Generate all components of a certificate:

    1. Configuration File:

        Each certificate can have its own configuration file. If no specific config file is provided, the playbook will use the default [config file](vars/config.yml).

    1. Generate individual components of a certificate:

        Create the private key:
        ```bash
        ansible-playbook create_private_key.yml -e @extra-vars.yml
        ```

        Create the Certificate Signing Request (CSR):
        ```bash
        ansible-playbook create_csr.yml -e @extra-vars.yml
        ```

        Create the certificate:
        ```bash
        ansible-playbook create_certificate.yml -e @extra-vars.yml
        ```

        Create the PKCS#12 file:
        ```bash
        ansible-playbook create_pkcs12.yml -e @extra-vars.yml
        ```

    1. Generate all components of a certificate in one step:

        Create all files using `OpenSSL` or `HCP Vault` (default):
        ```bash
        ansible-playbook create_all.yml -e @extra-vars.yml
        ```

1. Verify a certificate:

    ```bash
    ansible-playbook verify_certificate.yml -e @extra-vars.yml
    ```

1. Import existing certificates:

    ```bash
    ansible-playbook import_certificate.yml -e @extra-vars.yml
    ```

## Check the Created Files

1. Check a Private Key

    If the private key is not password-protected, use the following command:
    ```bash
    openssl rsa -in *.key -check
    ```

    If the private key is password-protected, provide the password using the `-passin` option.
    ```bash
    openssl rsa -in *.key -check -passin pass:<private_key_password>
    ```

1. Check a Certificate Signing Request (CSR)

    ```bash
    openssl req -text -noout -verify -in *.csr
    ```

1. Check a Certificate

    ```bash
    openssl x509 -text -noout -in *.cer
    ```

1. Check a PKCS#12 File (.pfx or .p12)

    ```bash
    openssl pkcs12 -info -in *.pfx -passin pass:<pfx_password>
    ```

**Created by:**

1. Luciano Sampaio
