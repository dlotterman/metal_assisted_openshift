# Redhat Cloud Assisted OpenShift cluster with Equinix Metal and Ansible
![](https://img.shields.io/badge/Stability-Experimental-red.svg) ![Ansible](https://img.shields.io/badge/ansible-%231A1918.svg?style=for-the-badge&logo=ansible&logoColor=white)

The Ansible and associated glue in this repository leverages the [Assisted Service](https://github.com/openshift/assisted-service) hosted by [Redhat Cloud](https://cloud.redhat.com/) and configures it's hosted [Automated Installer](https://github.com/openshift/assisted-installer) dynamically according to the on-demand provisioned resources it provisions out of [Equinix Metal](https://metal.equinix.com). 

The intent behind this repository is to reduce the barrier to entry for customers looking to explore the OpenShift ecosystem via Equinix Metal as the Bare Metal infrastructure provider.

This repository should be considered exploratory documentation only, it should not be considered as supported or production ready. 

## Getting Started and other Documentation

1. [Getting Started](docs/getting_started.md)
2. [Known Issues](docs/known_issues.md)

### Marketechture Diagram and Demo Video

![](https://s3.us-east-1.wasabisys.com/metalstaticassets/metal_openshift_diagram.JPG)

- [Time accelerated demo video](https://equinixinc-my.sharepoint.com/:v:/g/personal/dlotterman_equinix_com/EQ8sqptzxftClH3fVmJrvJgBYq7NIE3hNL3NaDrzNZyL4g?e=wr4SMi)

### TODO's

[Tracked here](TODO.md)