Getting started
===============

The easiest way to run the deployement is to use the **kubespray-cli** tool.
A complete documentation can be found in its [github repository](https://github.com/kubespray/kubespray-cli).

Here is a simple example on AWS:

* Create instances and generate the inventory

```
kubespray aws --instances 3
```

* Run the deployment

```
kubespray deploy --aws -u centos -n calico
```

Building your own inventory
---------------------------

Ansible inventory can be stored in 3 formats: YAML, JSON, or INI-like. There is
an example inventory located
[here](https://github.com/kubernetes-incubator/kubespray/blob/master/inventory/inventory.example).

You can use an
[inventory generator](https://github.com/kubernetes-incubator/kubespray/blob/master/contrib/inventory_builder/inventory.py)
to create or modify an Ansible inventory. Currently, it is limited in
functionality and is only use for making a basic Kubespray cluster, but it does
support creating large clusters. It now supports
separated ETCD and Kubernetes master roles from node role if the size exceeds a
certain threshold. Run inventory.py help for more information.

Example inventory generator usage:

```
cp -r inventory my_inventory
declare -a IPS=(10.10.1.3 10.10.1.4 10.10.1.5)
CONFIG_FILE=my_inventory/inventory.cfg python3 contrib/inventory_builder/inventory.py ${IPS[@]}
```

Starting custom deployment
--------------------------

Once you have an inventory, you may want to customize deployment data vars
and start the deployment:

**IMPORTANT: Edit my_inventory/groups_vars/*.yaml to override data vars**

```
ansible-playbook -i my_inventory/inventory.cfg cluster.yml -b -v \
  --private-key=~/.ssh/private_key
```

See more details in the [ansible guide](ansible.md).

Adding nodes
------------

You may want to add worker nodes to your existing cluster. This can be done by re-running the `cluster.yml` playbook, or you can target the bare minimum needed to get kubelet installed on the worker and talking to your masters. This is especially helpful when doing something like autoscaling your clusters.

- Add the new worker node to your inventory under kube-node (or utilize a [dynamic inventory](https://docs.ansible.com/ansible/intro_dynamic_inventory.html)).
- Run the ansible-playbook command, substituting `scale.yml` for `cluster.yml`:
```
ansible-playbook -i my_inventory/inventory.cfg scale.yml -b -v \
  --private-key=~/.ssh/private_key
```

Connecting to Kubernetes
------------------------
By default, Kubespray configures kube-master hosts with insecure access to
kube-apiserver via port 8080. A kubeconfig file is not necessary in this case,
because kubectl will use http://localhost:8080 to connect. The kubeconfig files
generated will point to localhost (on kube-masters) and kube-node hosts will
connect either to a localhost nginx proxy or to a loadbalancer if configured.
More details on this process is in the [HA guide](ha.md).

Kubespray permits connecting to the cluster remotely on any IP of any 
kube-master host on port 6443 by default. However, this requires 
authentication. One could generate a kubeconfig based on one installed 
kube-master hosts (needs improvement) or connect with a username and password.
By default, two users are created: `kube` and `admin` with the same password.
The password can be viewed after deployment by looking at the file 
`PATH_TO_KUBESPRAY/credentials/kube_user`. This contains a randomly generated
password. If you wish to set your own password, just precreate/modify this
file yourself. 

For more information on kubeconfig and accessing a Kubernetes cluster, refer to
the Kubernetes [documentation](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/).
