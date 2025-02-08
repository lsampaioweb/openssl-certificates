# Create, Verify, and Import OpenSSL Certificates

This playbook automates the process of **creating, verifying, and importing OpenSSL certificates**. It ensures that certificates signed by a **custom Certificate Authority (CA)** remain valid, generates new ones when needed, and imports them into various applications.

#
### Credentials

Before running the playbook, store the SMTP user password securely in the **secret manager**. This will be used to send email notifications.

1. **Store the SMTP password** (you will be prompted to enter it):
   ```bash
   secret-tool store --label="ssl.certificate-smtp-password" password "ssl.certificate-smtp-password"
   ```

2. **Retrieve the stored SMTP password**:
   ```bash
   secret-tool lookup password "ssl.certificate-smtp-password"
   ```

#
### How to Execute

Run the following **Ansible playbooks** as needed:

### Control Machine Setup
**Explanation of Flags:**
- `-K` - Prompts for the **sudo password** to execute tasks as root.

```bash
ansible-playbook control_machine.yml -K
```

### Main Execution
```bash
ansible-playbook site.yml
```

### Certificate Management
```bash
ansible-playbook create_private_key.yml
ansible-playbook create_signing_request.yml
ansible-playbook create_certificate.yml
ansible-playbook create_pkcs12.yml
```

### Certificate Verification
```bash
ansible-playbook verify_certificate.yml
```

### Certificate Import
**Explanation of Flags:**
- `-k` - Prompts for the **SSH password** when connecting to remote hosts.
- `-K` - Prompts for the **sudo password** to execute tasks as root.

```bash
ansible-playbook import_certificate.yml -k -K
```

#
### Variables

Customize the execution using the following **extra variables**:

### 1. Debug Mode
Enables detailed logging. The default value is **True** if not specified.

```json
--extra-vars='{"debug":True}'
```

### 2. Certificates
Specifies which certificates to verify (or create, if needed). By default, all certificates are verified.

```json
--extra-vars='{"certificates":["domain.com"]}'
--extra-vars='{"certificates":["domain.com", "otherdomain.com.br"]}'
```

### 3. Backup Mode
Determines whether old certificates should be moved to the **backup folder** when generating new ones. The default value is **True**.

```json
--extra-vars='{"backup":True}'
```

#
### Exclude Folders

To exclude specific folders from verification, prefix their names with an **underscore (`_`)**.

Example:
```bash
# Rename the folder to exclude it:
domain.com.br -> _domain.com.br
```

#
### Config File

The **default configuration** used to generate certificates is stored in the [config file](roles/certificate/files/config.yml "Config File").
- Do **not** modify this file for a single certificate.
- Instead, create a **custom config file** inside the specific certificate’s folder and override only the required settings.

#
### Available Import Options

This playbook supports importing certificates into the following platforms:

1. **Debian**
2. **Eclipse**
3. **macOS Keychain**
4. **OpenSSL**
5. **Proxmox**
6. **Python**
7. **TP-Link Switch (T1600G-28TS-SG2424)**

See the import sample [file](roles/certificate/files/import-sample.yml "Import Sample File") for details.

#
### License

This project is licensed under the [MIT License](LICENSE).

#
### Created by

- **Luciano Sampaio**
