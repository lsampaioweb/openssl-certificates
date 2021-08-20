# Create, Verify and Import OpenSSL Certificates

The playbook can verify if the certificate created by our own Certificate Authority (CA) is still valid. Can also create a new one and import it into several different applications.

# Requirements:

The Certificate Authority Server (CAS) must be a MAC OSX. If you want to use these playbooks on a Linux, you only have to change where you save the passwords. Here on the MAC I am using the Keychain. I know Ubuntu has the keyring. I don't know what to use on other distributions.

# How to execute:

  ```bash
  ansible-playbook install-requirements.yml
  ```
  
  ```bash
  ansible-playbook create-private-key.yml
  ansible-playbook create-signing-request.yml
  ansible-playbook create-certificate.yml
  ansible-playbook create-pkcs12.yml
  ```

  ```bash
  ansible-playbook verify-certificate.yml
  ```  
  
  ```bash
  ansible-playbook import-certificate.yml
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

1. Eclipse.
2. MacOSX Keychain.
3. OpenSSL.
4. Proxmox.
5. Python.
6. Switch Tplink T1600G-28TS-SG2424.

See the import sample [file](roles/certificate/files/import-sample.yml "Import Sample File").

# License:

[MIT](LICENSE "MIT License")

# Created by: 

1. Luciano Sampaio.
