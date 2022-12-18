# StairMaster

StairMaster is an Ansible Playbook that will connect to a number of Linux servers, install the Step CA CLI tools, and request a certificate via step CLI.
It will then copy the certificate and key to the needed places, and restart the relevant services.

Run this Playbook on a schedule to ensure the certificates are always up to date.


```bash
## Install needed pip modules
pip3 install -r ./requirements.txt

## Install needed Ansible Collections
ansible-galaxy collection install -r ./collections/requirements.yml
```