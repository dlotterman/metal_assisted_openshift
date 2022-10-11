### TODO's

#### Improvements

- Poll [reboot wait vs pause](https://github.com/dlotterman/metal_assisted_openshift/blob/74a771e214aabcfa53f2152b2f7525c97c09ff94/roles/metal/tasks/metal_reboot_ocp_hosts.yaml#L15)

#### Documentation

- Pretty much TODO all of it

#### Code quality / robustness
- Fix hardcoded bond.vlan inteface name in json_query string for `redhat_cloud_build_ip_host_id_table`\
- Fix hardcode network prefixes in [redhat_cloud_create_openshift_cluster](https://github.com/dlotterman/metal_assisted_openshift/blob/ea7b4e7720d3e20ae81f4bd4355a9d469a2a00b1/roles/redhat_cloud/tasks/redhat_cloud_create_openshift_cluster.yaml#L24)
