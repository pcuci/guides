### This guide provides instructions on how to set up CRI-O for OCP 3.9 (3.10 to follow)

There are 4 ansible variables that should be set when installing CRI-O


```
#CRI-O variables
openshift_use_crio=true
openshift_crio_use_rpm=true 
openshift_crio_enable_docker_gc=true
openshift_crio_docker_gc_node_selector={'runtime': 'cri-o'}
```

And node labels are required
```
[nodes]
10.1.1.101 openshift_node_labels="{'region': 'master', 'zone': 'm1', 'runtime': 'cri-o' }"
10.1.1.104 openshift_node_labels="{'region': 'infra', 'zone': 'i1', 'runtime': 'cri-o' }"
10.1.1.107 openshift_node_labels="{'region': 'primary', 'zone': 'a1', 'runtime': 'cri-o' }"
10.1.1.108 openshift_node_labels="{'region': 'primary', 'zone': 'a2', 'runtime': 'cri-o' }"
10.1.1.109 openshift_node_labels="{'region': 'primary', 'zone': 'a3', 'runtime': 'cri-o' }"
```



for 3.10

```
#CRI-O variables
openshift_use_crio=true
openshift_crio_use_rpm=true
openshift_crio_enable_docker_gc=true
```

There are a couple of approches to do the next step.  

dockergc must run where builds happen. You can set the following variable if you would like to specify builds to happen on certain nodes via 

```
openshift_builddefaults_nodeselectors={'node-role.kubernetes.io/compute':'true'}
```

This will do builds on Compute nodes

So we want dockergc to run on compute nodes we must update the node config to have a "runtime:crio" label.

```
openshift_node_groups=[{'name': 'node-config-master', 'labels': ['node-role.kubernetes.io/master=true']}, {'name': 'node-config-infra', 'labels': ['node-role.kubernetes.io/infra=true',]}, {'name': 'node-config-compute', 'labels': ['node-role.kubernetes.io/compute=true','runtime=cri-o'] }]
```

Please see documentation:  
https://docs.openshift.com/container-platform/3.10/install/configuring_inventory_file.html#configuring-inventory-defining-node-group-and-host-mappings  


### Storage

CRI-O storage is defaulted to overlay (overlay2)  

If you would like to use lvm it requires some manual setup between the prerequisites.yml playbook and deploy_cluster.yml

Note: you may be used to looking /etc/sysconfig/...  for the storage options. There is a cri-storage file but I don't think it used.  

Update the following file: /etc/crio/crio.conf

```
# storage_driver select which storage driver is used to manage storage
# of images and containers.
storage_driver = "devicemapper"

# storage_option is used to pass an option to the storage driver.
storage_option = [
        "dm.directlvm_device=/dev/vdb"
]
```

You can pass other option to the dm.directlvm_device driver but the defaults seem reasonable.  
At least here: https://docs.docker.com/storage/storagedriver/device-mapper-driver/#configure-direct-lvm-mode-for-production  

Its dockers site so not sure how that relates to the driver we may be installing.

from a raw device it created this:  

```

vdb                      252:16   0   50G  0 disk
├─storage-thinpool_tmeta 253:2    0  508M  0 lvm
│ └─storage-thinpool     253:4    0 47.5G  0 lvm
└─storage-thinpool_tdata 253:3    0 47.5G  0 lvm
  └─storage-thinpool     253:4    0 47.5G  0 lvm

```

There was no need to a setup script. Simply starting and stopping the crio service created the  

One intersting option
dm.directlvm_device_force=false    Not sure if we want that but seems to me can be used to "clean up" crio storage if set to true. Please see documentation.  





 
