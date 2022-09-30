# Redhat Cloud Assisted OpenShift cluster with Equinix Metal and Ansible
![](https://img.shields.io/badge/Stability-Experimental-red.svg) ![Ansible](https://img.shields.io/badge/ansible-%231A1918.svg?style=for-the-badge&logo=ansible&logoColor=white)

The Ansible and associated glue in this repository leverages the [Assisted Service](https://github.com/openshift/assisted-service) hosted by [Redhat Cloud](https://cloud.redhat.com/) and configures the [Automated Installer](https://github.com/openshift/assisted-installer) dynamically according to the on-demand provisioned resources hosted by [Equinix Metal](https://metal.equinix.com). 

The intent behind this repository is to reduce the barrier to entry for customers of Equinix Metal to build and test Redhat OpenShift based architectures.

This repository should be considered exploratory documentation only, it should not be considered as supported or production ready. As of September / October 2022, this repository is under active development with significant TODOs, so much so putting together the TODO is a TODO.

## Getting Started and other Documentation

1. [Getting Started](docs/getting_started.md)
2. [Known Issues](docs/known_issues.md)

### Marketechture Diagram

![](https://s3.us-east-1.wasabisys.com/metalstaticassets/metal_openshift_diagram.JPG)