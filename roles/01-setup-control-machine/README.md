# Setup the control machine to run Ansible scripts

Run the command in the terminal:
```bash
  ansible-playbook 01-setup-control-machine.yml -K (--ask-become-pass)
```

# Tasks:

## 1. Install required packages:
  1. **SSH Pass**. It allows you to provide the ssh password without using the prompt. This will be necessary for the first settings when we don't have a SSH keypair yet.
  2. Install the community.general module
    ansible-galaxy collection install community.general

## 2. Add the IP and URL of each host into the /etc/hosts file:
    10.0.3.5 DC-BR-SE-AJU-KVM-01.homelab DC-BR-SE-AJU-KVM-01
    10.0.3.6 DC-BR-SE-AJU-KVM-02.homelab DC-BR-SE-AJU-KVM-02
    10.0.3.7 DC-BR-SE-AJU-KVM-03.homelab DC-BR-SE-AJU-KVM-03
    10.0.3.8 DC-BR-SE-AJU-KVM-04.homelab DC-BR-SE-AJU-KVM-04
    10.0.3.9 DC-BR-SE-AJU-KVM-05.homelab DC-BR-SE-AJU-KVM-05
    10.0.3.10 DC-BR-SE-AJU-KVM-06.homelab DC-BR-SE-AJU-KVM-06

## 3. Add the fingerprint of each host into the ~/.ssh/known_hosts file:
    DC-BR-SE-AJU-KVM-01.homelab ecdsa-sha2-nistp256 ...
    DC-BR-SE-AJU-KVM-02.homelab ecdsa-sha2-nistp256 ...
    DC-BR-SE-AJU-KVM-03.homelab ecdsa-sha2-nistp256 ...
    DC-BR-SE-AJU-KVM-04.homelab ecdsa-sha2-nistp256 ...
    DC-BR-SE-AJU-KVM-05.homelab ecdsa-sha2-nistp256 ...
    DC-BR-SE-AJU-KVM-06.homelab ecdsa-sha2-nistp256 ...

## 4. Create the folder that will contain all OpenSSL Certificates:
  Create a symbolic link.

# Created by: 

1. Luciano Sampaio.