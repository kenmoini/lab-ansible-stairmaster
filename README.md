# Lab - StairMaster

StairMaster is an Ansible Playbook that will:

- Connect to a number of Linux servers
- Read in the Site Config for that Linux server
- Install the [Step](https://smallstep.com/docs/step-cli) & optionally the [Step CA](https://smallstep.com/docs/step-ca) CLI tools
- Request any number of SSL certificate(s)
- Place copies of them wherever needed in whatever format needed and keeps them in sync
- Run post creation/renewal commands to do things such as set SELinux contexts & restart services

> For more information on Step CA and how to set up your own Step CA Server, [see my blog post on how to do just that with other fun things like ACME and certbot](https://kenmoini.com/post/2022/04/diy-ca-with-small-step/)!

This is helpful in situations where you can't leverage ACME due to port conflicts, service availability, lack of solvers, or other reasons.  It's also help just to do scale out certificate management for things like Cockpit, NGINX, HAProxy, Kubernetes, OpenShift, FreeIPA/IDM, etc in a more GitOps-y automated manner.

Run this Playbook on a schedule to ensure the certificates are always up to date.

## Getting Started

First off, the configuration in this repo is mine which means it's not likely to be usable by you - and you don't have edit access to this repo more than likely.  So, you'll need to fork this repo and then clone your forked repo to your local machine.

1. [Fork this repo](https://github.com/kenmoini/lab-ansible-stairmaster/fork)
2. Clone your forked repo to your local machine - `git clone https://github.com/YOUR_USERNAME/lab-ansible-stairmaster.git`.
3. If you'll be running the Playbook manually with the CLI then set up your inventory file - see the [examples/inventory](examples/inventory) file for an example of how to do this.  If you plan on using this with Ansible Tower you can skip creating the inventory file.
4. Next, you'll need to create the directory structure for your Site Config files.  This allows you to deploy multiple certificates for multiple services to multiple hosts, and have it all atomically defined as YAML.  The directory structure for the Site Configs is as follows:

```
site_configs/
├── {inventory_hostname} (the name of the Host in your Ansible Inventory)
│   ├── {YAML FILES} (files for your Site Config, like certificates.yml)
```

5. Create your Site Config files.  See the [examples/site_configs](examples/site_config.yml) file for examples.  You can organize the `certificates` variable for each inventory host however you'd like, even adding host-specific vaulted secrets or other variables in the host's site config folder.
6. If you're using global secrets you will likely need to make a vaulted file to store them.  See the [examples/vault.yml](examples/vault.yml) file for an example of how to do this.

### Getting Started - CLI

```bash
## Install needed Ansible Collections
ansible-galaxy collection install -r ./collections/requirements.yml

## If you need to create some vaulted variables, like for the signing CA or IDM Directory Manager Password
ansible-vault create vars/vault.yml

## Run the Playbook
ansible-playbook -i inventory deploy.yml --ask-vault-pass
```

### Getting Started - Ansible Tower

Once your Site Configs are defined and any additional secrets are vaulted, you can import the Playbook into Ansible Tower and run it on a schedule and on `git push`es - handy!

Ideally as soon as you commit a change to the `main` branch of this repo, the changes will be synced to the actual certificates and services running on the hosts.  To do this, we'll use Ansible Tower to run the `deploy.yml` playbook.  Assuming you already have Ansible Tower or the AAP2 Controller installed and ready to go, you can follow these steps to set things up:

1. Create a **Credential(s)** for the systems that you will be connecting to.  You can use the `Machine` credential type for this.  You can also use the `Source Control` credential type for the Git repo that you'll be using for the configuration files.
2. Create an **Inventory** for the hosts that you'll be deploying the certificates to.  This Inventory needs two groups, the `server` for the StepCA server and the `clients` for the clients that will be requesting certificates.  See the [examples/inventory](examples/inventory) file for an example of the Inventory structure.
3. Add the **Hosts** to the **Inventory**.
4. Create a **Project** for your fork of this Git repo.  I like to keep the names the same as the repo, so I named mine `lab-ansible-stairmaster`.  Make sure to check the `Update Revision on Launch` box.
5. Create a **Template** for the `deploy.yml` Playbook, add the Credentials you'll need to connect to the hosts.  Important: Make sure to check the `Enable Webhook` box.  This will allow you to trigger the Playbook from a GitHub/GitLab webhook.
6. Once the Template is created, you can get the `Webhook URL` and the `Webhook Key` from the **Details** page of the Template.  You'll need these to set up the Webhook in GitHub/GitLab.

Assuming you'll be using this with GitHub, you can follow these steps to set up the Webhook: https://docs.ansible.com/ansible-tower/latest/html/userguide/webhooks.html

> With this, now when you perform a `git push` to the repo to update the intended state of your managed certificate files, the Playbook will be triggered and the changes will be automatically deployed to the hosts!
