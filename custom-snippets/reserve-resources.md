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

::: {#7e7b78a8-c4f6-42eb-896f-e7d3d68e4916 .cell .markdown}
### Reserve resources

Now, we are ready to reserve resources!
:::

::: {#9f9530a6-1dd0-4c28-86fe-3cf428b83a4f .cell .markdown}
First, make sure you don't already have a slice with this name:
:::

::: {#59ea0a49-1d8b-493b-be2e-b3d2ec62c7c2 .cell .code}
``` python
try:
    slice = fablib.get_slice(slice_name)
    print("You already have a slice by this name!")
    print("If you previously reserved resources, skip to the 'log in to resources' section.")
except:
    print("You don't have a slice named %s yet." % slice_name)
    print("Continue to the next step to make one.")
    slice = fablib.new_slice(name=slice_name)
```
:::

::: {#46c43ff8-8992-48ef-8ec9-68bf7c5b6f74 .cell .markdown}
We will select a site for our experiment that has an IPv4 management network - otherwise, setting up the cluster is more complicated:
:::

::: {#0d73aba0-267e-4304-8bc1-1306e8cabafe .cell .code}
``` python
import random
site_name = "FIU"
fablib.show_site(site_name)
```
:::

::: {#ab3d525a-54b3-4664-8f74-523c62ce4d09 .cell .markdown}
Then we will add hosts and network segments:
:::

::: {#6bbe7a77-f5d2-4ffc-954f-3065af9e53ac .cell .code}
``` python
# this cell sets up the nodes
for n in node_conf:
    slice.add_node(name=n['name'], site=site_name, 
                   cores=n['cores'], 
                   ram=n['ram'], 
                   disk=n['disk'], 
                   image=n['image'])
```
:::

::: {#040a9853-4b99-4240-8d83-f6ed32d19c9f .cell .code}
``` python
# this cell sets up the network segments
for n in net_conf:
    ifaces = [slice.get_node(node["name"]).add_component(model="NIC_Basic", 
                                                 name=n["name"]).get_interfaces()[0] for node in n['nodes'] ]
    slice.add_l2network(name=n["name"], type='L2Bridge', interfaces=ifaces)
```
:::

::: {#ab6c96c8-55dd-4e63-96a8-190283265aac .cell .markdown}
The following cell submits our request to the FABRIC site. The output of this cell will update automatically as the status of our request changes.

-   While it is being prepared, the "State" of the slice will appear as "Configuring".
-   When it is ready, the "State" of the slice will change to "StableOK".

You may prefer to walk away and come back in a few minutes (for simple slices) or a few tens of minutes (for more complicated slices with many resources).
:::

::: {#2d4eceab-8270-4f79-ac47-6942e4ac7e52 .cell .code}
``` python
slice.submit()
```
:::

::: {#664f08ca-fabd-4d46-a7fc-a3c5c20c8146 .cell .code}
``` python
slice.get_state()
slice.wait_ssh(progress=True)
```
:::