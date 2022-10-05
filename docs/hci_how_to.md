### Purposing a OCP Cluster installed on Metal for Hyperconverged Virtualiazation hosting

This guide assumes 4.11, which is noteably different that 4.10

#### Install Web Terminal

This will be used to write out a couple of configuration files as well as just being a handy tool.

- Go to the Operator Hub
  - Search Web Terminal
  - Intall Web Terminal
  
#### Install Local Storage Operator

- Go to Operator Hub
- Search for Local Storage operator provided by Redhat
  - Install the Local Storage Operator with defaults
  - When the installer finishes, click the view `View Operator` button on the finish screen
    - If you loose the button, you can find it again by going to "Installed Operators" and making sure you have the Project set to "All Projects"
  - In the Local Storage Operator, click on the "Create Instance" button for "Local Volume Discovery" or go to the Local Volume Discovery tab and click the "Create Local Volume Discovery" button there.
    - Follow defaults, node selector should be "Disks on all nodes"
	  - This should take only a couple of seconds (10) to complete
- When complete, go to the "Compute" section then -> nodes. Each node can be opened and on it's "Disks" tab, should have a sensible number of drives based on the Metal configuration. If an instance has a different number of drives then any other, it's likely that drive somehow had a parition table on it that the Local Storage discovery operator did not like. The drive should be wiped (`sgdisk --zap-all`) and the local storage operator task should be re-run.

#### Install Data Foundation

- Go to Operator Hub
  - Be sure to select "All projects" in the projects drop down
- Search for Data Foundation
- Install Data Foundation
  - All defaults should be correct, click through
- When Data Foundation install complete, click the "Create StorageSystem" button
  - If you clicked away and the button is no longer available, it can be found by loading the Data Foundation operator from the "Installed Operators" page.
    - ** IMPORTANT ** If when you enter the "Create StorageSystem" page, you get a page that asks for "Name, Labels, name" etc, and has a drop down for "kind" with only two options where one of those options is Flashblade, ** THIS IS A BUG in OCP 4.11 ** and the browser has a bad cache. Simply clear cache for the site or log in with a new incognito window and you should be good to proceed
	- What you ** SHOULD SEE ** is  form with a drop down for "Deployment Type" and a list of task subjects that includes "Backing Storage, create local volume set" etc.
		- One should be able to click though this with all the defaults besides adding names
		  - ** NAMES SHOULD BE SHORT ** There is a bug in names that are longer than 12 chars or similar. 
	- The Storage System creation can take several minutes while the storage system is spun up. 
- It make take a "Forced Refresh" of your browser for the "Storage" category to load.
- Once Data Foundation is installed, we need to set a default storage class ahead of virtualization being installed.
  - Under "Storage", go to "Storage Classes"
  - Go into "ocs-storagecluster-ceph-rbd" (This is the Ceph Block Device storage class and is the shared read / write class we want to use for virt)
    - Go to the "YAML" tab
	- In the YAML editor, near the top should be a section for "annotations", add `storageclass.kubernetes.io/is-default-class: 'true'`, this should make this stanza of yaml look like:
	  ``` kind: StorageClass
	  apiVersion: storage.k8s.io/v1
	  metadata:
	    name: ocs-storagecluster-ceph-rbd
	   resourceVersion: '431189'
       creationTimestamp: '2022-10-05T13:18:46Z'
       annotations:
         description: 'Provides RWO Filesystem volumes, and RWO and RWX Block volumes'
	     storageclass.kubernetes.io/is-default-class: 'true'
        managedFields:
	  ```
    - Click "Save"

#### Install Virtualization

- Go to Operator Hub
  - Be sure to select "All projects" in the projects drop down
- Search for "OpenShift Virtualization" and install it (defaults should be fine)
- When Virtualizaiton is installed, a button for "Create Hyperconverged" should be availble
  - If you clicked away from the installer, the "Create Hyperconvered" button should be available in the OpenShift Virtualization Operator hub page. Click that button.
  - Defaults for configuring Virtualization are fine, click through.
- It make take a "Forced Refresh" of your browser for the "Virtualization" category to load.

#### Install Kubernetes NMState
- Go to Operator Hub
  - Be sure to select "All projects" in the projects drop down
- Search for "NMState"
- Install NMState Operator
  - All defaults are fine
- When installed, find the NMState Operator in the Operator hub and go into it's details page
  - Click on and add "Create instance" of NMstate in it's Operator Hub page.
    - Click through defaults are fine
  
#### Apply networking

We first need to add a `NodeNetworkConfigurationPolicy` that adds a VM centric VLAN and bridge interface. This will let hosted VM's communicate in their own Metal VLAN network.

It does so by creating a Linux VLAN instance, and then a bridge into the VLAN interface.

- In the upper right hand corner of the page, there should be a `>_` icon, this is the icon for the web terminal which can be brought up at any time. Do so now (it may take a couple of seconds)
- In the web terminal, use your editor of choice to write a file called `nncp.yaml` with the following:
```
apiVersion: nmstate.io/v1
kind: NodeNetworkConfigurationPolicy
metadata:
  name: worker-br-2824-bond0-vlan1044
spec:
  nodeSelector:
    node-role.kubernetes.io/worker: ""
  desiredState:
    interfaces:
      - name: bond0.2824
        type: vlan
        state: up
        vlan:
          base-iface: bond0
          id: 2824
      - name: br-2824
        description: br-2824 with bond0.2824
        type: linux-bridge
        state: up
        ipv4:
          enabled: false
        bridge:
          options:
            stp:
              enabled: false
          port:
            - name: bond0.2824
```

  - Where VLAN `2824` should be replaced with your Metal VLAN
- Apply with `oc apply -f nncp.yaml`

Next we need to add a Network Attachment Definition that patches the Linux bridge we created above into the Kubernetes / OpenShift managed network.

- In the web terminal, use your editor of choice to write a file called `nad.yaml` with the following:
```
apiVersion: "k8s.cni.cncf.io/v1"
kind: NetworkAttachmentDefinition
metadata:
  name: br-2824
  namespace: default
  annotations:
    k8s.v1.cni.cncf.io/resourceName: bridge.network.kubevirt.io/br-2824
spec:
  config: '{
    "cniVersion": "0.3.1",
    "name": "br-2824",
    "plugins": [
      {
        "type": "cnv-bridge",
        "bridge": "br-2824"
      },
      {
        "type": "tuning"
      }
    ]
  }'
```
- Apply with: `oc apply -f nad.yaml`
- NAD should be visible in Network -> NetworkAttachmentDefinitions
- You can minimize the web terminal


#### Create VM

- Go to Virtualization -> VirtualMachines
- Choose "Create VirtualMachine"
- Choose "CentOS Stream 9 VM"
- Uncheck "Start this Virtual Machine after creation"
- Choose "Customize VirtualMachine"
  - Add a Network Interface that is attached to our newly created `NAD`
- Provision the VM
  - Add a Metal public IP to the VM