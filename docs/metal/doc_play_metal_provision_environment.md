### doc_play_metal_provision_environment

This play setups up the Metal environment (VLAN's, Reserved IP's etc) around prior to launching any Metal instances.

Everything in this play is idempotent and can be re-run as needed.

#### doc_metal_provision_ocp_vlan

This task provisions the VLAN that will be used for OpenShift's cluster networking. We need a VLAN because of our need for Metal Gateways.

#### doc_metal_get_ocp_vlan_id

Once we've create the OpenShift VLAN, we will need it's ID later, for example in the [Metal Gateway](../roles/metal/tasks/metal_provision_metal_gateway.yaml#L10) creation.

Think of this as a library call.

#### doc_metal_get_or_provision_ocp_ip_reservation

We need to reserve a block of IPs (16 /28 is really the smallest useable currently) for OpenShift's public facing network. This IP block will be used immediately after to create the Metal Gateway, and will also be used for OpenShifts front end cluster network, including floating VIPs.

First we get a list of reservations, then if we see one of the reservations has our OpenShift domain tag, we get it's ID. If we don't find one, we request a new reservation, and then we get it's ID. This means we should only ever reserve one block.

We then build a list of special or reserved IP's that will be set aside based on their order.

This is functionally a library call, this lets us build a map of IP configuration from the API without having to pre-define anything.

#### doc_metal_provision_metal_gateway

This task provisions a Metal Gateway if none exists. It uses the VLAN and Reserved IP information gathered earlier.

We need Metal Gateways because we need to be able to [float](https://en.wikipedia.org/wiki/Virtual_IP_address) Metal owned Layer-3 IP's for Openshift's cluster VIPs.
