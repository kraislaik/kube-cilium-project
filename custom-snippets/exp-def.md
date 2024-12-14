---
jupyter:
  jupytext:
    text_representation:
      extension: .md
      format_name: pandoc
      format_version: 3.1.12.3
      jupytext_version: 1.16.2
  nbformat: 4
  nbformat_minor: 5
---

::: {#92acc8f7-8c79-46c4-92c0-83519904a02e .cell .markdown}
### Define configuration for this experiment (3 VMs)
:::

::: {#7831fd3b-e9dd-4536-903c-561390d0c7d2 .cell .markdown}
For this specific experiment, we will need three virtual machines connected to a common network. Each of the virtual machines will be of the `m1.large` type, with 4 VCPUs, 8 GB memory, 40 GB disk space.
:::

::: {#9dd2aa42-41cf-4c32-993f-7b5bfc887ddc .cell .code}
``` python
import os
slice_name = "k8s_" + os.getenv('NB_USER')

node_conf = [
 {'name': "node-0", 'cores': 4, 'ram': 8, 'disk': 40, 'image': 'default_ubuntu_22', 'packages': ["virtualenv"]}, 
 {'name': "node-1", 'cores': 4, 'ram': 8, 'disk': 40, 'image': 'default_ubuntu_22', 'packages': []}, 
 {'name': "node-2", 'cores': 4, 'ram': 8, 'disk': 40, 'image': 'default_ubuntu_22', 'packages': []} 
]
net_conf = [
 {"name": "net0", "subnet": "192.168.1.0/24", "nodes": [{"name": "node-0",   "addr": "192.168.1.10"}, {"name": "node-1", "addr": "192.168.1.11"}, {"name": "node-2", "addr": "192.168.1.12"}]},
]
route_conf = []

exp_conf = {'cores': sum([ n['cores'] for n in node_conf]), 'nic': sum([len(n['nodes']) for n in net_conf]) }
```
:::