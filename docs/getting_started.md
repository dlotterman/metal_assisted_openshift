## Getting Started
TODO

### Required Accounts and Credentials:

- [Equinix Metal Read / Write Token](https://github.com/openshift/assisted-installer)
- [Equinix Metal Project ID](https://metal.equinix.com/developers/docs/accounts/projects/)
- [Redhat Cloud OpenShift Manager API Token](https://console.redhat.com/openshift/token)
- [Redhat Cloud OpenShift pull-secret](https://console.redhat.com/openshift/install/pull-secret)

You will also need an SSH key-pair to use as the administrative SSH credentials to log into the Control / Worker nodes. You can use the same SSH key-pair as your Metal account or create a new one. Ansible will assume that passwordless SSH authentication so the keys must be properly configured prior to beginning. 

### Environment setup:

This guide assumes a modern but *simple* execution environment that makes use of Python's virtualenv functionality to create a isolated workspace. Any distrobution with a Python distrobution version of `3.8` or greater should work as expected.

Besides standard Linux tooling, the remaining requirement is access to the public Internet for the packageinstallation and reaching the Metal and Redhat Cloud's public APIs. There is no requirement for the environment to be able to host public facing applications, as in a NAT'ed VM or equivalent should work fine.

For the sake of being *copy + paste* useable, commands here will assume a RHEL-clone (CentOS, Rocky or Alma) environment.

- [Install git](https://github.com/git-guides/install-git)
  - `sudo dnf install git -y`

- Clone this repository:
  - `git clone https://github.com/dlotterman/metal_assisted_openshift`

- (On RHEL-8 clones) Install Python39 or newer:
  - `sudo dnf install python39 -y`

- Create the Python virtualenv
  - `python3.9 -m venv metal_assisted_openshift`

- Change into the repository directory
  - `cd metal_assisted_openshift`

- Source the virtualenv environment setup
  - `source bin/activate`

- Update pip
  - `pip install --upgrade pip`

- Install Python packages required
  - `pip install -r requirements.txt`

#### Per session / authentication setup
Each time you start a new shell session, I.E a new ssh session or new `tmux` session, you will need to set certain environment variables related to your Metal and Redhat Cloud account.

The list of these is documented at the [top of this page](getting_started.md#Required-Accounts-and-Credentials)

These are:

- `export METAL_API_TOKEN=$YOUR_METAL_API_KEY_HERE`
- `export METAL_PROJ_ID=YOUR_METAL_PROJECT_ID_HERE`
- `export REDHAT_CLOUD_OPENSHIFT_TOKEN=$YOUR_REDHAT_CLOUD_OPENSHIFT_TOKEN_HERE`
- `export REDHAT_CLOUD_PULL_SECRET_AUTH=$REDHAT_CLOUD_PULL_SECRET_AUTH`
- `export REDHAT_CLOUD_PULL_SECRET_REGISTRY_AUTH=$REDHAT_CLOUD_PULL_SECRET_REGISTRY_AUTH`
- `export REDHAT_CLOUD_PULL_SECRET_EMAIL=$REDHAT_CLOUD_PULL_SECRET_EMAIL`
- `export OCP_PUBLIC_SSH_KEY=$OCP_PUBLIC_SSH_KEY`


You must also always source the `activate` environment setup helper every new session: `source bin/activate`
### Configuration
#### Dynamic Inventory

This ansible configuration uses Ansible's [Dynamic Inventory](https://docs.ansible.com/ansible/latest/user_guide/intro_dynamic_inventory.html) feature, meaning that all of the inventorying of Metal resources will be done dynamically via the API. The collection of Metal instances provisioned and their associated attributes is managed via groupings of [Device Taggings](https://metal.equinix.com/developers/docs/server-metadata/device-tagging/) in realtime.

The Dynamic Inventory plugin for Equinix Metal comes via it's [Ansible Galaxy package](https://galaxy.ansible.com/equinix/metal). 

The only configuration required is by editing the `projects:` stanza of the [Dynamic Inventory file](../equinix_metal.yaml#L15)

This should be updated to the same Metal Project UUID string as used in the earlier environment variable setup. 

#### OpenShift Cluster Naming

The name and search domain of the cluster must be set in the [group_vars file](../group_vars/all.yaml). 

It makes life significantly easier if this can be a publically resovable / updateable name and domain.

#### Defaults
There should be no other configuration required, defaults are provided to provision a sample 3x node cluster using [c3.small.x86](../roles/metal/defaults/main.yaml#L7)'s as the control / worker hosts and a `m3.small.x86` as the utility `opsbox` [host](../roles/metal/tasks/metal_get_or_provision_opsbox.yaml).

#### Confirmation

If everything is setup correctly, you should be able to run `ansible-inventory -i equinix_metal.yaml --list` and the command will return **empty** but *without errors*.

### Provisioning the environment

This is the fun part. Execute this command to have Ansible provision the Equinix Metal resources and bootstrap them through the Redhat Cloud managed OpenShift installation process.

`ansible-playbook -i equinix_metal.yaml -u root metal_ocp_ai_provision.yaml`

It can take up an hour and a half for it to complete, you can watch the progress from:
- The session running Ansible which will progress through named steps
- The Equinix Metal UI (It can be fun to watch the [SOS](https://metal.equinix.com/developers/docs/resilience-recovery/serial-over-ssh/) of one of the control instances as it goes through the installation) 
