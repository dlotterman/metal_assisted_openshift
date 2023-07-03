## Getting Started

Before beginning, it is strongly recomemended that an operator complete the [Equinix Metal Getting Started](https://metal.equinix.com/developers/docs/) guide before proceeding with anything referenced here. The getting started documentation will make sure the account is correctly setup and walk the operator through the basics of operating Metal.

### Required Accounts and Credentials:

- [Equinix Metal Read / Write Token](https://github.com/openshift/assisted-installer)
- [Equinix Metal Project ID](https://metal.equinix.com/developers/docs/accounts/projects/)
- [Redhat Cloud OpenShift Manager API Token](https://console.redhat.com/openshift/token)
- [Redhat Cloud OpenShift pull-secret](https://console.redhat.com/openshift/install/pull-secret)

You will also need an SSH key-pair to use as the administrative SSH credentials to log into the Control / Worker nodes. You can use the same SSH key-pair as your [Metal account](https://metal.equinix.com/developers/docs/accounts/ssh-keys/) or create a new one specific to OCP cluster management. Ansible will assume passwordless SSH authentication so the keys must be properly configured prior to beginning.

### Environment setup:

This guide assumes a modern but *simple* execution environment. It makes use of Python's virtualenv functionality to create a isolated workspace. Any OS with a Python distrobution version of `3.8` or greater should work as expected.

Besides standard nix-like tooling, the remaining requirement is access to the public Internet for the package installation and reaching the Metal and Redhat Cloud's public APIs. There is no requirement for the environment to be able to host public facing applications, as in a NAT'ed VM or equivalent should work fine.

For the sake of being *copy + paste* useable, commands here will assume a RHEL8-clone (CentOS, Rocky or Alma) environment.

- [Install git](https://github.com/git-guides/install-git)
  - RHEL-8 clone: `sudo dnf install git -y`

- Clone this repository:
  - `git clone https://github.com/dlotterman/metal_assisted_openshift`

- Install Python39 or newer:
  - On RHEL-8 clones `sudo dnf install python39 -y`
  - On Ubuntu 22.04: `sudo apt install python3.10-venv`

- Create the Python virtualenv
  - `python3 -m venv metal_assisted_openshift`
	- On RHEL8-clone: `python3.9 -m venv metal_assisted_openshift`

- Change into the repository directory
  - `cd metal_assisted_openshift`

- Source the virtualenv environment setup
  - `source bin/activate`

- Update pip
  - `pip install --upgrade pip`

- Install Python packages required
  - `pip install -r requirements.txt`

- Install Equinix Metal Ansible Galaxy Collection
  - `ansible-galaxy collection install equinix.metal`

#### Per session / authentication setup
Each time you start a new shell session, I.E a new ssh session or new `tmux` session, you will need to set certain environment variables related to your Metal and Redhat Cloud account.

The list of these is documented at the [top of this page](getting_started.md#Required-Accounts-and-Credentials)

These are:

- `export METAL_API_TOKEN=$YOUR_METAL_API_KEY_HERE`
- `export METAL_PROJ_ID=YOUR_METAL_PROJECT_ID_HERE`
- `export REDHAT_CLOUD_OPENSHIFT_TOKEN=$YOUR_REDHAT_CLOUD_OPENSHIFT_TOKEN_HERE`
- `export REDHAT_CLOUD_PULL_SECRET_AUTH=$REDHAT_CLOUD_PULL_SECRET_AUTH` --- *{"cloud.openshift.com":{"auth"* in your pull-secret.txt
- `export REDHAT_CLOUD_PULL_SECRET_REGISTRY_AUTH=$REDHAT_CLOUD_PULL_SECRET_REGISTRY_AUTH` --- *registry.connect.redhat.com":{"auth":"* in your pull-secret.txt
- `export REDHAT_CLOUD_PULL_SECRET_EMAIL=$REDHAT_CLOUD_PULL_SECRET_EMAIL`
- `export OCP_PUBLIC_SSH_KEY=$OCP_PUBLIC_SSH_KEY`


You must also always source the `activate` environment setup helper every new session: `source bin/activate`
### Configuration
#### Dynamic Inventory

This ansible configuration uses Ansible's [Dynamic Inventory](https://docs.ansible.com/ansible/latest/user_guide/intro_dynamic_inventory.html) feature, meaning that all of the inventorying of Metal resources will be done dynamically via the API. The collection of Metal instances provisioned and their associated attributes is managed via groupings of [Device Taggings](https://metal.equinix.com/developers/docs/server-metadata/device-tagging/) in realtime.

The Dynamic Inventory plugin for Equinix Metal comes via it's [Ansible Galaxy package](https://galaxy.ansible.com/equinix/metal).

To configure the dynamic inventory file, edit the YAML file to add your project UUID string:
  - Editing the `projects:` stanza of the [Dynamic Inventory file](../equinix_metal.yaml#L15)

This should be updated to the same Metal Project UUID string as used in the earlier environment variable setup.

The domain name for the cluster also needs to be updated in `group_vars/all.yaml`.

#### OpenShift Cluster Naming

The name and search domain of the cluster must be set in the [group_vars file](../group_vars/all.yaml).

It makes life significantly easier if this can be a publically resovable / updateable name and domain.

#### Defaults
There should be no other configuration required, defaults are provided to provision a sample 3x node cluster using [c3.small.x86](../roles/metal/defaults/main.yaml#L7)'s as the control / worker hosts and a `m3.small.x86` as the utility `opsbox` [host](../roles/metal/tasks/metal_get_or_provision_opsbox.yaml).

#### Confirmation

If everything is setup correctly, you should be able to run `ansible-inventory -i equinix_metal.yaml --list` and the command will return **empty** but *without errors*.

With errors:
```
$ ansible-inventory -i equinix_metal.yaml --list
[WARNING]:  * Failed to parse /home/adminuser/code/metal_assisted_openshift/equinix_metal.yaml with auto plugin: Failed
to query devices from Equinix Metal API. Error 404: Not found
[WARNING]:  * Failed to parse /home/adminuser/code/metal_assisted_openshift/equinix_metal.yaml with yaml plugin: Plugin
configuration YAML file, not YAML inventory
[WARNING]:  * Failed to parse /home/adminuser/code/metal_assisted_openshift/equinix_metal.yaml with ini plugin: Invalid
host pattern '---' supplied, '---' is normally a sign this is a YAML file.
[WARNING]: Unable to parse /home/adminuser/code/metal_assisted_openshift/equinix_metal.yaml as an inventory source
[WARNING]: No inventory was parsed, only implicit localhost is available
{
    "_meta": {
        "hostvars": {}
    },
    "all": {
        "children": [
            "ungrouped"
        ]
    }
}
```

*Without* errors:
```
$ ansible-inventory -i equinix_metal.yaml --list
{
    "_meta": {
        "hostvars": {}
    },
    "all": {
        "children": [
            "equinix_metal",
            "ungrouped"
        ]
    }
}
```

### Provisioning the environment

This is the fun part. Execute this command to have Ansible provision the Equinix Metal resources and bootstrap them through the Redhat Cloud managed OpenShift installation process.

- `ansible-playbook -i equinix_metal.yaml -u root metal_ocp_ai_provision.yaml`

To run the playbook without having Ansible stop to prompt for SSH Keys and just auto-accept them:
- `ANSIBLE_HOST_KEY_CHECKING=False ansible-playbook -i equinix_metal.yaml -u root metal_ocp_ai_provision.yaml`

It can take up an hour and a half for it to complete, you can watch the progress from:
- The session running Ansible which will progress through named steps
- The Equinix Metal UI (It can be fun to watch the [SOS](https://metal.equinix.com/developers/docs/resilience-recovery/serial-over-ssh/) of one of the control instances as it goes through the installation)
- The [Redhat Cloud Console](https://console.redhat.com/openshift/), which will provide the best view on cluster installation status


#### Common Errors:

- SSH Error to OpsBox

	```failed: [opsbox01.ocp06.da.dlott.casa] (item=/metal) => {"ansible_loop_var": "item", "item": "/metal", "msg": "Failed to connect to the host via ssh: Warning: Permanently added '147.75.53.55' (ECDSA) to the list of known hosts.\r\nroot@147.75.53.55: Permission denied (publickey).", "unreachable": true}
	failed: [opsbox01.ocp06.da.dlott.casa] (item=/metal/ansible_lock_dir) => {"ansible_loop_var": "item", "item": "/metal/ansible_lock_dir", "msg": "Failed to connect to the host via ssh: root@147.75.53.55: Permission denied (publickey).", "unreachable": true}
	```
  - Ensure the execution environment you have is setup with the right private SSH keys as to use the same Public SSH key uploaded to your [Metal Account](https://metal.equinix.com/developers/docs/accounts/ssh-keys/)

#### De-provisioning the environment

The Redhat Cluster can be deleted via `ansible-playbook -i equinix_metal.yaml -u root ocp_ai_deprovision.yaml` . Note that the Redhat Cloud console exposes 2x seperate entries (`Assisted cluster ID` vs `Cluster ID`) for the cluster in it's UI, one for the hosted [Assited Installer service](https://console.redhat.com/openshift/assisted-installer/clusters/) and one for the [OpenShift Cloud platform](https://console.redhat.com/openshift). The playbook only deletes the Assisted Installer entry, to delete the OpenShift Cloud entry, simply archive it through the UI.

If someothing goes wrong with the Redhat Cloud steps, it is possible to walk back only the Redhat Cloud steps, as in run the `ocp_ai_deprovision.yaml` playbook and archive the other cluster object and re-run the playbook. The Metal steps are idempotent once provisioned and can be re-run as needed.

To delete the Metal provisions and return to a clean project:
* Delete the Metal Instances
* Delete the Metal Gateway
* Delete the Metal VLAN
* Delete the Metal IP Reservation (note you can leave this is you have DNS set and it will be correctly re-used)
