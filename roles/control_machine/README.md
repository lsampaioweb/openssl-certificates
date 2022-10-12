# Setup the control machine to run Ansible scripts

Run the command in the terminal:
```bash
  ansible-playbook control_machine.yml -K #(--ask-become-pass)
```

# Tasks:

## 1. Create the folder that will contain all OpenSSL Certificates:
  Create a symbolic link.

## 2. Install required packages:
  1. **SSH Pass**. It allows you to provide the ssh password without using the prompt. This will be necessary for the first settings when we don't have a SSH keypair yet.

  2. Install the community.general module
    ansible-galaxy collection install community.general

# Created by: 

1. Luciano Sampaio.