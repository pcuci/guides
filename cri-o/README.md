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

