Upgrades within the Kubernetes ecosystem are plentiful and happen often. While upgrades are not something that we want to process all the time, it is something that we want to be able to confidently process. With our Kubernetes providers, upgrades are handled in a way that maximizes uptime and should mostly not force resources into data-plane downtime.

# Kubespray

Running upgrades with Kubespary is handled by the `upgrade-cluster.yml` playbook. While this playbook works, it does have a couple of caveats.

1. An upgrade can only handle one major jump.  If you're running 1.26 and want to go to 1.28, you'll need to upgrade to 1.27 first and repeat the process until you land on the desired version.

2. The upgrade playbook will drain and move workloads around to ensure the environment maximizes uptime. While maximizing uptime makes for incredible user experiences, it does mean the process of executing an upgrade can be very long (2+ hours is normal); plan accordingly.

## Preparing the upgrade

When running Kubespray using the Genestack submodule, review the [Genestack Update Process](https://github.com/rackerlabs/genestack/wiki/Running-a-Genestack-upgrade) before continuing with the kubespray upgrade and deployment.

Genestack stores inventory in the `/etc/genestack/inventory` directory. Before running the upgrade, you will need to set the **kube_version** variable to your new target version. This variable is generally found within the `/etc/genestack/inventory/group_vars/k8s_cluster/k8s-cluster.yml` file.

> Review all of the group variables within an environment before running a major upgrade. Things change, and you need to be aware of your environment details before running the upgrade.

Once the group variables are set, you can proceed with the upgrade execution.

## Running the upgrade

Running an upgrade with Kubespray is fairly simple and executed via `ansible-playbook`.

Before running the playbook be sure to source your environment variables.

``` shell
source /opt/genestack/scripts/genestack.rc
```

Change to the `kubespary` directory.

``` shell
cd /opt/genestack/submodules/kubespray
```

Now run the upgrade.

``` shell
ansible-playbook upgrade-cluster.yml
```

> While the basic command could work, be sure to include any and all flags needed for your environment before running the upgrade.

### Running an unsafe upgrade

When running an upgrade, it is possible to force the upgrade by running the cluster playbook with the `upgrade_cluster_setup` flag set to **true**. This option is a lot faster, though does introduce the possibility of service disruption during the upgrade operation.

``` shell
ansible-playbook cluster.yml -e upgrade_cluster_setup=true
```

### Post upgrade operations

After running the upgrade it's sometimes a good thing to do some spot checks of your nodes and ensure everything is online, operating normally.

#### Dealing with failure

If an upgrade failed on the first attempt but succeeded on a subsequent run, you may have a node in a `Ready`, but `SchedulingDisabled` state. If you find yourself in this scenario you may need to `uncordon` the node to get things back to operating normally.

``` shell
kubectl uncordon $NODE
```
