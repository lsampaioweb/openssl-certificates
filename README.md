# Create, Verify and Import OpenSSL Certificates

The playbook can verify if the certificate created by our own Certificate Authority (CA) is still valid. Can also create a new one and import it into several different applications.

# Requirements:

The Certificate Authority Server (CAS) must be a MAC OSX or a Ubuntu Linux.

# Credentials:
1. Create a strong password for the smtp's user and store it in the secret manager. After you hit enter, a password will be asked.
```bash
    secret-tool store --label="ssl.certificate-smtp-password" password "ssl.certificate-smtp-password"
```

2. Retrieve the smtp's user password.
```bash
    secret-tool lookup password "ssl.certificate-smtp-password"
```

# How to execute:

  ```bash
  ansible-playbook control_machine.yml -K
  ```

  ```bash
  ansible-playbook site.yml
  ```

  ```bash
  ansible-playbook create_private_key.yml
  ansible-playbook create_signing_request.yml
  ansible-playbook create_certificate.yml
  ansible-playbook create_pkcs12.yml
  ```

  ```bash
  ansible-playbook verify_certificate.yml
  ```

  ```bash
  ansible-playbook import_certificate.yml -K -k
  ```

# Variables:

1. debug

Specify if more messages should be displayed during the executions.
If no value is passed, the default value **True** will be used.

  ```json
  --extra-vars='{"debug":True}'
  ```

2. certificates

Specify which certificates will be verified and, if requested, created.
If no value is passed, all certificates will be verified.
  ```json
  --extra-vars='{"certificates":["domain.com"]}'
  --extra-vars='{"certificates":["domain.com", "otherdomain.com.br"]}'
  ```

3. backup

Specify if when creating new certificates the old ones should be moved to the backup folder.
If no value is passed, the default value **True** will be used.
  ```json
  --extra-vars='{"backup":True}'
  ```

# Exclude folders:

If you want to exclude one or more folders from the verification, you can add a **"_"** before the name.

  ```
  Rename the folder:
    domain.com.br -> _domain.com.br
  ```

# Config File:

In the config [file](roles/certificate/files/config.yml "Config File"), you will find all the **default** configurations that will be used to generate the certificate related files. You shouldn't change this file if the config will be used for only one certificate. If that's the case, you should create a specific config file inside the folder of the certificate, with only the configs you want to be different from the default ones.

# Available Import Options:

1. Debian.
2. Eclipse.
3. MacOSX Keychain.
4. OpenSSL.
5. Proxmox.
6. Python.
7. Switch Tplink T1600G-28TS-SG2424.

See the import sample [file](roles/certificate/files/import-sample.yml "Import Sample File").

# License:

[MIT](LICENSE "MIT License")

# Created by:

1. Luciano Sampaio.
