### TODO's

#### Production Breaking Bugs

- [ ] Need to fix serial / TTYS1
  - Just need to patch the installer like documented [here](https://github.com/openshift/installer/blob/master/docs/user/customization.md)


#### Improvements

- [ ] Poll [reboot wait vs pause](https://github.com/dlotterman/metal_assisted_openshift/blob/74a771e214aabcfa53f2152b2f7525c97c09ff94/roles/metal/tasks/metal_reboot_ocp_hosts.yaml#L15)
- [ ] Handle 4x NIC configurations (currently only handles 2x NIC configurations)
- [ ] Improve cluster status / runbook to include polling of install

#### Documentation

- Pretty much TODO all of it
- Update Marketecture diagram to include new Hybrid model (vs old Layer-2 model)

#### Code quality / robustness
- Fix hardcoded bond.vlan inteface name in json_query string for `redhat_cloud_build_ip_host_id_table`
- Fix hardcode network prefixes in [redhat_cloud_create_openshift_cluster](https://github.com/dlotterman/metal_assisted_openshift/blob/ea7b4e7720d3e20ae81f4bd4355a9d469a2a00b1/roles/redhat_cloud/tasks/redhat_cloud_create_openshift_cluster.yaml#L24)
- Fix [creation / templating](https://github.com/dlotterman/metal_assisted_openshift/blob/main/roles/redhat_cloud/templates/openshift_host_template.json.j2) of host YAML during Openshift Cluster creation
