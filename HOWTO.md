# How to run this role

To deploy the demo server:

Install Centos 7 and then login the first time,
for example:

**We assume the hypervisor has the IP 192.168.1.20**

```bash
ssh root@192.168.1.20
ssh-keygen -t rsa -f ~/.ssh/id_rsa -q -P ""
curl -sS https://github.com/ccamacho.keys >> ~/.ssh/authorized_keys
exit
```

## Platform deployment

* Provision the environment packages using libvirt:

```bash
ansible-playbook \
    --user root \
    -vv -i ./hosts/demo-noha/inventory \
    -l hypervisor-nodes \
    --become \
    --become-user root \
    --tags provision_libvirt \
    ./playbook.yml
```

After provisioning the environment, this command should work.

This will prove connectivity between the provisioning node
and all the nodes to be configured in this guide.

**They all need to SUCCESS before continue**

```bash
sleep 60
ansible --user root -i ./hosts/demo-noha/inventory -m ping all
```

If there are nodes not responding is because the virsh interface attach
command didn't work as expected or is not refreshed correctly in the guests.
So we run a task to wake up those zombies.

We will run the wake up routine in both groups (masters and workers).

```bash
ansible-playbook \
    --user root \
    -vv -i ./hosts/demo-noha/inventory \
    -l kubernetes-master-nodes:kubernetes-worker-nodes  \
    --become \
    --become-user root \
    --tags provision_libvirt_restart_zombies \
    ./playbook.yml
```

## Kubernetes deployment

* Install the kubernetes cluster (master nodes):

```bash
ansible-playbook \
    --user root \
    -vv -i ./hosts/demo-noha/inventory \
    -l kubernetes-master-nodes \
    --become \
    --become-user root \
    --tags kubernetes_common,kubernetes_master \
    ./playbook.yml
```

```bash
ansible --user root -i ./hosts/demo-noha/inventory -m ping all
```

In this point you should be able to connect to the master node and execute:

```bash
kubectl get nodes
```

Then get the output as:

```
NAME                                  STATUS   ROLES    AGE    VERSION
kubernetes-master-01.dev.pystol.org   Ready    master   2m7s   v1.16.3
```

Now, we proceed to install the worker nodes.

* Install the kubernetes cluster (worker nodes):

```bash
ansible-playbook \
    --user root \
    -vv -i ./hosts/demo-noha/inventory \
    -l kubernetes-worker-nodes \
    --become \
    --become-user root \
    --tags kubernetes_common,kubernetes_worker \
    ./playbook.yml
```

```bash
ansible --user root -i ./hosts/demo-noha/inventory -m ping all
```

* Install kubernetes base apps (master nodes):

```bash
ansible-playbook \
    --user root \
    -vv -i ./hosts/demo-noha/inventory \
    -l kubernetes-master-nodes \
    --become \
    --become-user root \
    --tags kubernetes_deploy_apps \
    ./playbook.yml
```

## Pystol deployment

* Install Pystol:

```bash
ansible-playbook \
    --user root \
    -vv -i ./hosts/demo-noha/inventory \
    -l kubernetes-master-nodes \
    --become \
    --become-user root \
    --tags pystol_install \
    ./playbook.yml
```