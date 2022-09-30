### Equinix Metal Configuration Types
- `m3.small.x86`: Known issue with BMC virtual NIC prevents use as OCP instance, can be opsbox instance. Work pending from Redhat to fix
- `c3.small.x86`: Should work as both OCP and opsbox instances
- `n2/n3.xlarge.x86`: Not tested, likely Ansible issues around 4x NIC awareness
- `c2/3.medium.x86 / m3.large.x86`: Mixed bag. Instances of these types delivered as R6515's may have an old firmware that errors OS booting after the initial OCP installation. These instance types likely take baby sitting.
- `s3.xlarge.x86`: not actively tested but should work

### SOS / OOB / Console Output
OCP currently gets installed in such a way as the redirection to console is broken for use with the Metal OOB / SOS. This is known and understood, fix is known and undertood, simply not yet implemented in this repository

### Metal BGP Functionality:
Currently the networkign architecture implemented here puts the OCP instances into the Layer-2 networking mode. This restricts access to the Metal BGP feature. This is a known flaw and will be corrected by moving OCP networking into a VLAN context allowing the nodes to be placed into the Hybrid Networking mode allowing for BGP.