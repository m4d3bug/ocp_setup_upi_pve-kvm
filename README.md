# Automated OpenShift 4 Cluster Installation on PVE-KVM

### Prerequistes:

- Internet connected physical host running a modern linux distribution
- Virtualization enabled and PVE/PVE-KVM setup [(more details)](https://github.com/kxr/ocp4_setup_upi_pve-kvm/wiki/Setup-PVE-KVM-PVE)
- DNS on the host managed by dnsmasq or NetworkManager/dnsmasq [(more details)](https://github.com/kxr/ocp4_setup_upi_pve-kvm/wiki/Setting-Up-DNS)
- OpenShift 4 Pull secret (Download from [here](https://cloud.redhat.com/openshift/install/pull-secret))

## Installing OpenShift 4 Cluster

### Demo:

[![asciicast](https://asciinema.org/a/bw6Wja2vBLrAkpKHTV0yGeuzo.svg)](https://asciinema.org/a/bw6Wja2vBLrAkpKHTV0yGeuzo)

### Usage:
./ocp4_setup_upi_pve-kvm.sh [OPTIONS]


| Option  |Description   |
| :------------ | :------------ |
|______________________________||
| -O, --ocp-version VERSION | You can set this to "latest", "stable" or a specific version like "4.1", "4.1.2", "4.1.latest", "4.1.stable" etc.<br>Default: stable |
| -R, --rhcos-version VERSION | You can set a specific RHCOS version to use. For example "4.1.0", "4.2.latest" etc.<br>By default the RHCOS version is matched from the OpenShift version. For example, if you selected 4.1.2  RHCOS 4.1/latest will be used |
| -p, --pull-secret FILE | Location of the pull secret file<br>Default: /root/pull-secret |
| -c, --cluster-name NAME | OpenShift 4 cluster name<br>Default: ocp4 |
| -d, --cluster-domain DOMAIN | OpenShift 4 cluster domain<br>Default: local |
| -m, --masters N | Number of masters to deploy<br>Default: 3 |
| -w, --worker N | Number of workers to deploy<br>Default: 2 |
| --master-cpu N | Number of CPUs for the master VM(s)<br>Default: 4 |
| --master-mem SIZE(MB) | RAM size (MB) of master VM(s)<br>Default: 16000 |
| --worker-cpu N | Number of CPUs for the worker VM(s)<br>Default: 4 |
| --worker-mem SIZE(MB) | RAM size (MB) of worker VM(s)<br>Default: 8000 |
| --bootstrap-cpu N | Number of CPUs for the bootstrap VM<br>Default: 4 |
| --bootstrap-mem SIZE(MB) | RAM size (MB) of bootstrap VM<br>Default: 16000 |
| --lb-cpu N | Number of CPUs for the load balancer VM<br>Default: 1 |
| --lb-mem SIZE(MB) | RAM size (MB) of load balancer VM<br>Default: 1024 |
| -n, --pve-network NETWORK | The pve network to use. Select this option if you want to use an existing pve network<br>The pve network should already exist. If you want the script to create a separate network for this installation see: -N, --pve-oct<br>Default: default |
| -N, --pve-oct OCTET | You can specify a 192.168.{OCTET}.0 subnet octet and this script will create a new pve network for the cluster<br>The network will be named ocp-{OCTET}. If the pve network ocp-{OCTET} already exists, it will be used.<br>Default: [not set] |
| -v, --vm-dir | The location where you want to store the VM Disks<br>Default: /var/lib/pve/images |
| -z, --dns-dir DIR | We expect the DNS on the host to be managed by dnsmasq. You can use NetworkMananger's built-in dnsmasq or use a separate dnsmasq running on the host. If you are running a separate dnsmasq on the host, set this to "/etc/dnsmasq.d"<br>Default: /etc/NetworkManager/dnsmasq.d |
| -s, --setup-dir DIR | The location where we the script keeps all the files related to the installation<br>Default: /root/ocp4\_setup\_{CLUSTER_NAME} |
| -x, --cache-dir DIR | To avoid un-necessary downloads we download the OpenShift/RHCOS files to a cache directory and reuse the files if they exist<br>This way you only download a file once and reuse them for future installs<br>You can force the script to download a fresh copy by using -X, --fresh-download<br>Default: /root/ocp4_downloads |
| -X, --fresh-download | Set this if you want to force the script to download a fresh copy of the files instead of reusing the existing ones in cache dir<br>Default: [not set] |
| -k, --keep-bootstrap | Set this if you want to keep the bootstrap VM. By default bootstrap VM is removed once the bootstraping is finished<br>Default: [not set] |
| --autostart-vms | S
    ./ocp4_setup_upi_pve-kvm.sh --pull-secret /home/knaeem/Downloads/pull-secret --ocp-version latest
    ./ocp4_setup_upi_pve-kvm.sh -p /home/knaeem/Downloads/pull-secret -O latest

    # Deploy OpenShift 4.2.latest with custom cluster name and domain
    ./ocp4_setup_upi_pve-kvm.sh --cluster-name ocp43 --cluster-domain lab.test.com --ocp-version 4.2.latest
    ./ocp4_setup_upi_pve-kvm.sh -c ocp43 -d lab.test.com -O 4.2.latest

    # Deploy OpenShift 4.2.stable on new pve network (192.168.155.0/24)
    ./ocp4_setup_upi_pve-kvm.sh --ocp-version 4.2.stable --pve-oct 155
    ./ocp4_setup_upi_pve-kvm.sh -O 4.2.stable -N 155

    # Destory the already installed cluster
    ./ocp4_setup_upi_pve-kvm.sh --cluster-name ocp43 --cluster-domain lab.test.com --destroy
    ./ocp4_setup_upi_pve-kvm.sh -c ocp43 -d lab.test.com --destroy

___

## Adding Nodes

Once the installation is successful, you will find a `add_node.sh` script in the `--setup-dir` (default: /root/ocp4\_setup\_{CLUSTER_NAME}). You can use this to add more nodes to the cluster, post installation.

### Usage:
    cd [setup-dir]
    ./add_node.sh --name [node-name] [OPTIONS]

| Option  |Description   |
| :------------ | :------------ |
|______________________________||
| --name NAME | The node name without the domain.<br> For example: If you specify storage-1, and your cluster name is "ocp4" and base domain is "local", the new node would be "storage-1.ocp4.local".<br> Default: [not set] [REQUIRED] |
| -c, --cpu N | Number of CPUs to be attached to this node's VM. Default: 2|
| -m, --memory SIZE(MB) | Amount of Memory to be attached to this node's VM. Size in MB.<br> Default: 4096 |
| -a, --add-disk SIZE(GB) | You can add additional disks to this node. Size in GB.<br> This option can be specified multiple times. Disks are added in order for example if you specify "--add-disk 10 --add-disk 100", two disks will be added (on top of the OS disk vda) first of 10GB (/dev/vdb) and second disk of 100GB (/dev/vdc).<br> Default: [not set] |
| -v, --vm-dir | The location where you want to store the VM Disks.<br> By default the location used by the cluster VMs will be used. |
|  -N, --pve-oct OCTET| You can specify a 192.168.{OCTET}.0 subnet octet and this script will create a new pve network for this node.<br> The network will be named ocp-{OCTET}. If the pve network ocp-{OCTET} already exists, it will be used.<br> This can be useful if you want to add a node in different network than the one used by the cluster.<br> Default: [not set] |
| -n, --pve-network NETWORK | The pve network to use. Select this option if you want to use an existing pve network.<br> By default the existing pve network used by the cluster will be used. |

___

## Exposing the cluster outside the host/hypervisor
Once the installation is successful, you will find a `expose_cluster.sh` script in the `--setup-dir` (default: /root/ocp4\_setup\_{CLUSTER_NAME}). You can use this to expose this cluster so it can be accessed from outside.

### Usage:

    cd [setup-dir]
    ./expose_cluster.sh --method [ firewalld | haproxy ]

If you are running a single clust

___

## Auto Starting VMs

By default, if you reboot the host/hypervisor, the VMs will not start up automatically. You can set `--autostart-vms` when running the install script that will mark the VMs to auto-start. To see which VMs are set or not set to auto-start you can run `virsh list --all --name --autostart` or `virsh list --all --name --no-autostart` respectively.

If you want to change/set the autostart behaviour, you can set the VMs to auto-start by running:

~~~
for vm in $(virsh list --all --name --no-autostart | grep "<CLUSTER-NAME>"); do
  virsh autostart ${vm}
done
~~~

Similarly, to disable the auto starting of VMs, you can run:

~~~
for vm in $(virsh list --all --name --autostart | grep "<CLUSTER-NAME>"); do
  virsh autostart --disable ${vm}
done
~~~

Note: Replace `<CLUSTER-NAME>` with the cluster name or any matching string to filter out VMs that you want to set/un-set to be auto-started. 

___

## Errors While Waiting for clusterversion

When the bootstrap process is complete, the script waits for clusterversion to become ready before the cluster installation is considered completed. During this phase the script just shows the status/message of the the clustervresion operator. You can see different kind of errors which are normal. This is due to the nature of operator reconciliation process. For example:

~~~
====> Waiting for clusterversion:
--> Unable to apply 4.5.0-rc.6: an unknown error has occurred: MultipleErr ...
--> Unable to apply 4.5.0-rc.6: an unknown error has occurred: MultipleErr ...
--> Working towards 4.5.0-rc.6: 62% complete
--> Working towards 4.5.0-rc.6: 62% complete
--> Unable to apply 4.5.0-rc.6: an unknown error has occurred: MultipleErr ...
--> Working towards 4.5.0-rc.6: 99% complete
~~~

~~~
====> Waiting for clusterversion: 
  --> Working towards 4.3.12: 46% complete
  --> Unable to apply 4.3.12: an unknown error has occurred
  --> Working towards 4.3.12: 61% complete
  --> Unable to apply 4.3.12: an unknown error has occurred
  --> Unable to apply 4.3.12: an unknown error has occurred
  --> Unable to apply 4.3.12: an unknown error has occurred
~~~

Just let it run and hopefully the clusterversion operator will reconcile and become ready eventually.
___

## Number of masters and workers

___

## [Setting up OCS](https://github.com/kxr/ocp4_setup_upi_pve-kvm/wiki/Installing-OCS-4-(OpenShift-Container-Storage))
