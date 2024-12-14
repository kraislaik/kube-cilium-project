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

::: {#9e19de35-8443-4047-a5fc-f56b02d48f21 .cell .markdown}
### Use Kubespray to prepare a Kubernetes cluster
:::

::: {#89e786a4-ec20-4bdc-8622-502fc116913e .cell .markdown}
Now that are resources are "up", we will use Kubespray, a software utility for preparing and configuring a Kubernetes cluster, to set them up as a cluster.
:::

::: {#26d0c094-68c1-42ee-8981-41c2342d6796 .cell .code}
``` python
# install Python libraries required for Kubespray
remote = slice.get_node(name="node-0")
remote.execute("virtualenv -p python3 myenv")
remote.execute("git clone --branch release-2.22 https://github.com/kubernetes-sigs/kubespray.git")
_ = remote.execute("source myenv/bin/activate; cd kubespray; pip3 install -r requirements.txt")
```
:::

::: {#7ca837af-2fb0-4057-a6ec-6fe009aea198 .cell .code}
``` python
# copy config files to correct locations
remote.execute("mv kubespray/inventory/sample kubespray/inventory/mycluster")
remote.execute("git clone https://github.com/teaching-on-testbeds/k8s.git")
remote.execute("cp k8s/config/k8s-cluster.yml kubespray/inventory/mycluster/group_vars/k8s_cluster/k8s-cluster.yml")
remote.execute("cp k8s/config/inventory.py    kubespray/contrib/inventory_builder/inventory.py")
remote.execute("cp k8s/config/addons.yml      kubespray/inventory/mycluster/group_vars/k8s_cluster/addons.yml")
```
:::

::: {#b2e97e7b-3416-4390-a232-e089abad15cc .cell .code}
``` python
# build inventory for this specific topology
physical_ips = [n['addr'] for n in net_conf[0]['nodes']]
physical_ips_str = " ".join(physical_ips)
_ = remote.execute(f"source myenv/bin/activate; declare -a IPS=({physical_ips_str});"+"cd kubespray; CONFIG_FILE=inventory/mycluster/hosts.yaml python3 contrib/inventory_builder/inventory.py ${IPS[@]}")
```
:::

::: {#8b644c2a-b584-4ea9-9df0-4dc124453576 .cell .code}
``` python
# make sure "controller" node can SSH into the others
remote.execute('ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa -q -N ""')
public_key, stderr = remote.execute('cat ~/.ssh/id_rsa.pub', quiet=True)

for n in node_conf:
    node = slice.get_node(n['name'])
    print("Now copying key to node " + n['name'])
    node.execute(f'echo {public_key.strip()} >> ~/.ssh/authorized_keys')
```
:::

::: {#9e2fd08d-15ab-4de2-9baf-c66d6f27ad38 .cell .code}
``` python
# build the cluster
_ = remote.execute("source myenv/bin/activate; cd kubespray; ansible-playbook -i inventory/mycluster/hosts.yaml  --become --become-user=root cluster.yml")
```
:::

::: {#b1ad0ab5-9a78-4671-8705-aa33433a0b34 .cell .code}
``` python
# allow kubectl access for non-root user
remote.execute("sudo cp -R /root/.kube /home/ubuntu/.kube; sudo chown -R ubuntu /home/ubuntu/.kube; sudo chgrp -R ubuntu /home/ubuntu/.kube")
```
:::

::: {#df614587-04d2-42e2-959b-ae339a705f10 .cell .code}
``` python
# check installation
_ = remote.execute("kubectl get nodes")
```
:::

::: {#99d1cd55-35e6-4e67-8456-57d87ffa44f2 .cell .markdown}
### Set up Docker

Now that we have a Kubernetes cluster, we have a framework in place for container orchestration. But we still need to set up Docker, for building, sharing, and running those containers.
:::

::: {#6172f0ac-9279-49c7-b3eb-fafa15062d17 .cell .code}
``` python
# add the user to the "docker" group on all hosts
for n in node_conf:
    node = slice.get_node(n['name'])
    node.execute("sudo groupadd -f docker; sudo usermod -aG docker $USER")
```
:::

::: {#eab2e1ea-7280-4c70-8dfc-56adbdfe8f3f .cell .code}
``` python
# set up a private distribution registry on the "controller" node for distributing containers
# note: need a brand-new SSH session in order to "get" new group membership
remote.execute("docker run -d -p 5000:5000 --restart always --name registry registry:2")
```
:::

::: {#ae5a9ccb-9a18-4830-9bed-ba600c2170e6 .cell .code}
``` python
# set up docker configuration on all the hosts
for n in node_conf:
    node = slice.get_node(n['name'])
    node.execute("sudo wget https://raw.githubusercontent.com/teaching-on-testbeds/k8s/main/config/daemon.json -O /etc/docker/daemon.json")
    node.execute("sudo service docker restart")
```
:::

::: {#fc62f19f-aeb8-484c-965a-7f3100130519 .cell .code}
``` python
# check configuration
remote.execute("docker run hello-world")
```
:::

::: {#8779aa55-8a56-498f-983d-90dc393a3bcf .cell .markdown}
### Get SSH login details
:::

::: {#e81d7b82-02ef-40e6-afd3-609842527642 .cell .markdown}
At this point, we should be able to log in to our "controller" node over SSH! Run the following cell, and observe the output - you will see an SSH command this node.
:::

::: {#4bfb7d1f-1edb-4e07-86bb-ddd9509f782a .cell .code}
``` python
print(remote.get_ssh_command())
```
:::

::: {#13daef2d-5af9-4386-97d5-e3dacb5bbcb1 .cell .markdown}
Now, you can open an SSH session as follows:

-   In Jupyter, from the menu bar, use File \> New \> Terminal to open a new terminal.
-   Copy the SSH command from the output above, and paste it into the terminal.
:::