# Setup the control machine to run Ansible scripts

Run the command in the terminal:
```bash
  ansible-playbook control_machine.yml -K (--ask-become-pass)
```

# Tasks:

## 1. Install required packages:
  1. **SSH Pass**. It allows you to provide the ssh password without using the prompt. This will be necessary for the first settings when we don't have a SSH keypair yet.

  2. Install the community.general module
    ansible-galaxy collection install community.general

## 2. Add the IP and URL of each host into the /etc/hosts file:
    10.0.2.5 kvm-01.homelab kvm-01
    10.0.2.6 kvm-02.homelab kvm-02
    10.0.2.7 kvm-03.homelab kvm-03
    10.0.2.8 kvm-04.homelab kvm-04
    10.0.2.9 kvm-05.homelab kvm-05
    10.0.2.10 kvm-06.homelab kvm-06
    10.0.2.11 kvm-07.homelab kvm-07

## 3. Add the fingerprint of each host into the ~/.ssh/known_hosts file:
    kvm-01.homelab,kvm-01,10.0.2.5 ecdsa-sha2-nistp256 ...
    kvm-02.homelab,kvm-02,10.0.2.6 ecdsa-sha2-nistp256 ...
    kvm-03.homelab,kvm-03,10.0.2.7 ecdsa-sha2-nistp256 ...
    kvm-04.homelab,kvm-04,10.0.2.8 ecdsa-sha2-nistp256 ...
    kvm-05.homelab,kvm-05,10.0.2.9 ecdsa-sha2-nistp256 ...
    kvm-06.homelab,kvm-06,10.0.2.10 ecdsa-sha2-nistp256 ...
    kvm-07.homelab,kvm-07,10.0.2.11 ecdsa-sha2-nistp256 ...

## 4. Create the folder that will contain all OpenSSL Certificates:
  Create a symbolic link.

# Created by: 

1. Luciano Sampaio.