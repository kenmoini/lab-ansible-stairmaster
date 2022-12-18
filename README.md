# StairMaster

StairMaster is an Ansible Playbook that will connect to a number of Linux servers, install the Step CA CLI tools, and request a certificate via step CLI.
It will then copy the certificate and key to the needed places, and restart the relevant services.

Run this Playbook on a schedule to ensure the certificates are always up to date.


```bash
## Install needed pip modules
pip3 install -r ./requirements.txt

## Install needed Ansible Collections
ansible-galaxy collection install -r ./collections/requirements.yml

## If you need to create some vaulted variables, like for the signing CA
ansible-vault create vars/vault.yml

## Run the Playbook
ansible-playbook -i inventory deploy.yml --ask-vault-pass
```