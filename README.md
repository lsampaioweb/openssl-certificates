# Verify, Create, and Import Certificates

This set of playbooks can verify if the certificate created by our own Certificate Authority (CA) is still valid, create new certificates if necessary, and import certificates into multiple applications.

## Requirements

- The Ansible server must be running a Linux-based OS (tested on Oracle Linux and Ubuntu).

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

1. [Debian/Ubuntu](roles/import/tasks/linux/debian_ca.yml)

## How to Execute

1. Install the required roles and collections on the `Ansible Controller` server:

    - Choose the inventory file, the default is `inventory/hosts`:
      ```bash
      ansible-playbook prepare_controller.yml -i inventory/hosts -K
      ```

    - Specify the server name in the `--limit` parameter to prepare only that host, or omit it to prepare all hosts in the inventory:
      ```bash
      ansible-playbook prepare_controller.yml --limit "server-name" -K
      ```

    - To run the playbook with specific tags, use the `--tags` option. For example, to run only untagged tasks and dev_environment tasks:
      ```bash
      ansible-playbook prepare_controller.yml --tags "untagged,dev_environment" -K
      ```

1. Set environment variables for HashiCorp Vault access:

    ```bash
    nano ~/.bashrc

    export ANSIBLE_HASHI_VAULT_ADDR="https://vault.lan.home"
    export ANSIBLE_HASHI_VAULT_AUTH_METHOD="ldap"
    export ANSIBLE_HASHI_VAULT_USERNAME="usr_ansible"
    export ANSIBLE_HASHI_VAULT_PASSWORD="$(secret-tool lookup secret 'ansible_hashi_vault_password')"

    source ~/.bashrc
    ```

1. Save the SMTP password in the secret manager:

    The playbook sends an email notification after creating a certificate. The SMTP password must be stored in the OS secret manager (e.g: Keyring in Ubuntu) on the Ansible controller — never hardcode credentials in source files.

    The secret key `certificate_smtp_password` matches the default value of `config_vars.notification_mail.smtp_password_id` in `vars/config.yml`. Override that variable if you use a different key name.

    - Store the password:
      ```bash
      secret-tool store --label="certificate_smtp_password" password "certificate_smtp_password"
      ```

    - Confirm it was saved correctly:
      ```bash
      secret-tool lookup password "certificate_smtp_password"
      ```

1. Set up the Ansible user on each target host (required for certificate import):

    **Note**: This local user approach is a temporary solution until an LDAP/AD-based user is available. When that time comes, only `ANSIBLE_USER` and `ANSIBLE_PASSWORD` env vars need to be updated — no playbook changes required.

    The import playbook connects to target hosts using a dedicated local user. Run these commands **on each target host**:

    - Create the user:
      ```bash
      sudo useradd -m -s /bin/bash usr_ansible
      ```

    - Generate a strong random password and set it:
      ```bash
      openssl rand -base64 32
      sudo passwd usr_ansible
      ```

    - Grant passwordless sudo:
      ```bash
      sudo visudo -f /etc/sudoers.d/usr_ansible
      ```
      Add this single line — `visudo` validates syntax before writing, so it won't corrupt sudoers:
      ```
      usr_ansible ALL=(ALL) NOPASSWD: ALL
      ```

    Then, back **on the Ansible controller**, store the password in the secret manager and set the environment variables:

    - Store the password:
      ```bash
      secret-tool store --label="usr_ansible" password "usr_ansible"
      ```

    - Add the connection env vars to your shell profile:
      ```bash
      echo 'export ANSIBLE_USER="usr_ansible"' >> ~/.bashrc
      echo 'export ANSIBLE_PASSWORD="$(secret-tool lookup password usr_ansible)"' >> ~/.bashrc
      source ~/.bashrc
      ```

    - Test the connection before running the playbook:
      ```bash
      ssh usr_ansible@<target-host>
      ```

1. Generate all components of a certificate:

    1. Configuration File:

        Each certificate can have its own configuration file. If no specific config file is provided, the playbook will use the default [config file](vars/config.yml).

    1. Generate individual components of a certificate:

        - Create the private key:
          ```bash
          ansible-playbook create_private_key.yml -e @extra-vars.yml
          ```

        - Create the Certificate Signing Request (CSR):
          ```bash
          ansible-playbook create_csr.yml -e @extra-vars.yml
          ```

        - Create the certificate:
          ```bash
          ansible-playbook create_certificate.yml -e @extra-vars.yml
          ```

        - Create the PKCS#12 file:
          ```bash
          ansible-playbook create_pkcs12.yml -e @extra-vars.yml
          ```

    1. Generate all components of a certificate in one step:

        - Create all files using `OpenSSL` or `HCP Vault` (default):
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

    - Works for `RSA` keys.
      ```bash
      openssl rsa -in *.key -check
      ```

    - Works for `ECC` keys.
      ```bash
      openssl ec -in *.key -check
      ```

    - Works for both `RSA` and `ECC` keys in PKCS#8 format.
      ```bash
      openssl pkcs8 -in *.key
      ```

    - If the private key is password-protected, provide the password using the `-passin` option.
      ```bash
      openssl rsa -in *.key -check -passin pass:<private_key_password>
      ```

1. Check a Certificate Signing Request (CSR)

    ```bash
    openssl req -text -noout -verify -in *.csr
    ```

1. Check a Certificate

    ```bash
    openssl x509 -text -noout -in *.crt
    ```

1. Check a PKCS#12 File (.pfx or .p12)

    ```bash
    openssl pkcs12 -info -in *.pfx -passin pass:<pfx_password>
    ```

**Created by:**

1. Luciano Sampaio
