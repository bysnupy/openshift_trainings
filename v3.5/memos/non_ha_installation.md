# Non-HA OpenShift installation

## Introduction
It's just simple installation using 3 nodes including master node.

## Environment

### Versions

Item | Value | Target nodes
-|-|-
OS | RHEL7.4 | All
OpenShift Ver | 3.5 | All

### Node list

Hostname | Roles | Descriptions
-|-|-
console.host.local | Ansible control host, DNS, NTP | Having multiple roles required systems such as DNS, NTP and Ansible executing
master1.host.local | OpenShift Master server | single master
node1.host.local<br/>node2.host.local | OpenShift node servers | nodes running the pods included containers
*.openshift.host.local | Wildcard DNS name | This resource is required using router layer

## Installation steps

### Configuration of DNS and NTP on self-training environment

I've used console.host.local host as DNS and NTP server refereced from all nodes.

* NTP host configuraion file '/etc/chrony.conf' on console.host.local
```
server 0.rhel.pool.ntp.org iburst
server 1.rhel.pool.ntp.org iburst
server 2.rhel.pool.ntp.org iburst
server 3.rhel.pool.ntp.org iburst
driftfile /var/lib/chrony/drift
makestep 1.0 3
rtcsync
allow xxx.xxx.xxx.0/yy
logdir /var/log/chrony
```

* NTP client configuration file '/etc/chrony.conf' on master and nodes hosts
```
...omitted...
server console.host.local
...omitted...
```

* DNS host using dnsmasq service, configuration file '/etc/dnsmasq.d/host.local' on console.host.local host
```
domain-needed
bogus-priv
expand-hosts
domain=host.local
address=/.openshift.host.local/xxx.xxx.xxx.xxx
```

* If you use DHCP for setting up network interface, you need to disable PEERDNS=no on all node hosts
```
# nmcli con mod eth0 ipv4.ignore-dns-auto true
```

### Pre-installation tasks

* subscription registration using RHSM

* installation of required packages for initial setting up

* disabled the useless services, firewalld

* ensuring access with passwordless between hosts

* enabled the required services, NetworkManager and docker

* Step5: customizes the configurations of target nodes
  * node server's hostnames and wildcard domain name resolution is required
  * the time is synchonized by NTP on all nodes

* docker install and enabled with dedicated block volume

* atomic-openshift-utils installs only on master host

* remove the openshift packages exclusions on all nodes using 'atomic-openshift-excluder unexclude' command

### Installation

* execute 'atomic-openshift-installer install' on master1.host.local host using interactive installation mode at this time<br/><br/>
:star: This command also generates the answer file for unattended installation, the answer file has 'ansible_inventory_path' directive that point the ansible inventory file generated location. <br/>
The answer file first, not ansible inventory file on executed and generated orders. 

```
# atomic-openshift-installer install


Welcome to the OpenShift Enterprise 3 installation.

Please confirm that following prerequisites have been met:

* All systems where OpenShift will be installed are running Red Hat Enterprise
  Linux 7.
* All systems are properly subscribed to the required OpenShift Enterprise 3
  repositories.
* All systems have run docker-storage-setup (part of the Red Hat docker RPM).
* All systems have working DNS that resolves not only from the perspective of
  the installer, but also from within the cluster.

When the process completes you will have a default configuration for masters
and nodes.  For ongoing environment maintenance it's recommended that the
official Ansible playbooks be used.

For more information on installation prerequisites please see:
https://docs.openshift.com/enterprise/latest/admin_guide/install/prerequisites.html

Are you ready to continue? [y/N]: y

This installation process involves connecting to remote hosts via ssh. Any
account may be used. However, if a non-root account is used, then it must have
passwordless sudo access.

User for ssh access [root]: root

Which variant would you like to install?


(1) OpenShift Container Platform
(2) Registry

Choose a variant from above:  [1]: 1

*** Host Configuration ***

You must now specify the hosts that will compose your OpenShift cluster.

Please enter an IP address or hostname to connect to for each system in the
cluster. You will then be prompted to identify what role you want this system to
serve in the cluster.

OpenShift masters serve the API and web console and coordinate the jobs to run
across the environment. Optionally, you can specify multiple master systems for
a high-availability (HA) deployment. If you choose an HA deployment, then you
are prompted to identify a *separate* system to act as the load balancer for
your cluster once you define all masters and nodes.

Any masters configured as part of this installation process are also
configured as nodes. This enables the master to proxy to pods
from the API. By default, this node is unschedulable, but this can be changed
after installation with the 'oadm manage-node' command.

OpenShift nodes provide the runtime environments for containers. They host the
required services to be managed by the master.

http://docs.openshift.com/enterprise/latest/architecture/infrastructure_components/kubernetes_infrastructure.html#master
http://docs.openshift.com/enterprise/latest/architecture/infrastructure_components/kubernetes_infrastructure.html#node
    
Enter hostname or IP address: master1.host.local
Will this host be an OpenShift master? [y/N]: y
Will this host be RPM or Container based (rpm/container)? [rpm]: rpm

*** Installation Summary ***

Hosts:
- master1.host.local
  - OpenShift master
  - OpenShift node
  - Etcd

Total OpenShift masters: 1
Total OpenShift nodes: 1

NOTE: Add a total of 3 or more masters to perform an HA installation.

Do you want to add additional hosts? [y/N]: y
Enter hostname or IP address: node1.host.local
Will this host be an OpenShift master? [y/N]: N
Will this host be RPM or Container based (rpm/container)? [rpm]: rpm

*** Installation Summary ***

Hosts:
- master1.host.local
  - OpenShift master
  - OpenShift node (Unscheduled)
  - Etcd
- node1.host.local
  - OpenShift node (Dedicated)

Total OpenShift masters: 1
Total OpenShift nodes: 2

NOTE: Add a total of 3 or more masters to perform an HA installation.

Do you want to add additional hosts? [y/N]: y
Enter hostname or IP address: node2.host.local
Will this host be an OpenShift master? [y/N]: N
Will this host be RPM or Container based (rpm/container)? [rpm]: rpm

*** Installation Summary ***

Hosts:
- master1.host.local
  - OpenShift master
  - OpenShift node (Unscheduled)
  - Etcd
- node1.host.local
  - OpenShift node (Dedicated)
- node2.host.local
  - OpenShift node (Dedicated)

Total OpenShift masters: 1
Total OpenShift nodes: 3

NOTE: Add a total of 3 or more masters to perform an HA installation.

Do you want to add additional hosts? [y/N]: N

You have chosen to install a single master cluster (non-HA).

In a single master cluster, the cluster host name (Ansible variable openshift_master_cluster_public_hostname) is set by default to the host name of the single master. In a multiple master (HA) cluster, the FQDN of a host must be provided that will be configured as a proxy. This could be either an existing load balancer configured to balance all masters on
port 8443 or a new host that would have HAProxy installed on it.

(Optional)
If you want to override the cluster host name now to something other than the default (the host name of the single master), or if you think you might add masters later to become an HA cluster and want to future proof your cluster host name choice, please provide a FQDN. Otherwise, press ENTER to continue and accept the default.

Enter hostname or IP address [None]: 

Setting up high-availability masters requires a storage host. Please provide a
host that will be configured as a Registry Storage.

Note: Containerized storage hosts are not currently supported.

Enter hostname or IP address [master1.host.local]: 

You might want to override the default subdomain used for exposed routes. If you don't know what this is, use the default value.

New default subdomain (ENTER for none) []: openshift.host.local

If a proxy is needed to reach HTTP and HTTPS traffic, please enter the
name below. This proxy will be configured by default for all processes
that need to reach systems outside the cluster. An example proxy value
would be:

    http://proxy.example.com:8080/

More advanced configuration is possible if using Ansible directly:

https://docs.openshift.com/enterprise/latest/install_config/http_proxies.html

Specify your http proxy ? (ENTER for none) []: 
Specify your https proxy ? (ENTER for none) []: 

*** Installation Summary ***

Hosts:
- master1.host.local
  - OpenShift master
  - OpenShift node (Unscheduled)
  - Etcd
  - Storage
- node1.host.local
  - OpenShift node (Dedicated)
- node2.host.local
  - OpenShift node (Dedicated)

Total OpenShift masters: 1
Total OpenShift nodes: 3

NOTE: Add a total of 3 or more masters to perform an HA installation.

Gathering information from hosts...
 [WARNING]: Could not match supplied host pattern, ignoring: oo_lb_to_config
All hosts in config are uninstalled. Proceeding with installation...

The following is a list of the facts gathered from the provided hosts. The
hostname for a system inside the cluster is often different from the hostname
that is resolveable from command-line or web clients, therefore these settings
cannot be validated automatically.

For some cloud providers, the installer is able to gather metadata exposed in
the instance, so reasonable defaults will be provided.

Please confirm that they are correct before moving forward.


master1.host.local,192.168.123.144,192.168.123.144,master1.host.local,master1.host.local
node1.host.local,192.168.123.127,192.168.123.127,node1.host.local,node1.host.local
node2.host.local,192.168.123.183,192.168.123.183,node2.host.local,node2.host.local

Format:

connect_to,IP,public IP,hostname,public hostname

Notes:
 * The installation host is the hostname from the installer's perspective.
 * The IP of the host should be the internal IP of the instance.
 * The public IP should be the externally accessible IP associated with the instance
 * The hostname should resolve to the internal IP from the instances
   themselves.
 * The public hostname should resolve to the external IP from hosts outside of
   the cloud.

Do the above facts look correct? [y/N]: y

Wrote atomic-openshift-installer config: /root/.config/openshift/installer.cfg.yml
Wrote Ansible inventory: /root/.config/openshift/hosts

Ready to run installation process.

If changes are needed please edit the config file above and re-run.

Are you ready to continue? [y/N]: y

Play 1/28 (Create initial host groups for localhost)
..
Play 2/28 (Create initial host groups for all hosts)
.
Play 3/28 (Populate config host groups)
................ [WARNING]: Could not match supplied host pattern, ignoring: oo_lb_to_config
Play 4/28 (Ensure that all non-node hosts are accessible)
.
Play 5/28 (Initialize host facts)
.................
Play 6/28 (Gather and set facts for node hosts)
................
Play 7/28 (Verify compatible yum/subscription-manager combination)
..
Play 8/28 (Determine openshift_version to configure on first master)
..................................................................................................
Play 9/28 (Set openshift_version for all hosts)
..................................................................................................
Play 10/28 (Set oo_option facts)
........
Play 11/28 (Disable excluders)
...........................
Play 12/28 (Configure etcd)
.........................................................................................................................................................Pausing for 10 seconds
(ctrl+C then 'C' = continue early, ctrl+C then 'A' = abort)
...Pausing for 10 seconds
(ctrl+C then 'C' = continue early, ctrl+C then 'A' = abort)
...................................................................
Play 13/28 (Configure nfs)
...................................................
Play 14/28 (Configure load balancers)

Play 15/28 (Gather and set facts for master hosts)
........................
Play 16/28 (Determine if session secrets must be generated)
..........
Play 17/28 (Generate master session secrets)
...............
Play 18/28 (Configure masters)
...........................................................................................................................^[[A.....................................................................................................................................................................................................................................................................................................................................
Play 19/28 (Additional master configuration)
..................................................................................................................................................................................................................................
Play 20/28 (Gather and set facts for node hosts)
................
Play 21/28 (Evaluate node groups)
.. [WARNING]: Could not match supplied host pattern, ignoring: oo_containerized_master_nodes


Play 22/28 (Configure containerized nodes)

Play 23/28 (Configure nodes)
...................................................................................................................................................................................................................................................................................................................................................................
Play 24/28 (Additional node config)
..........................................................................................................................
Play 25/28 (Create persistent volumes)
.......................................................................................................................................................................
Play 26/28 (Create Hosted Resources)
.................................................................................................................................................................................................................. [WARNING]: Module did not set no_log for stats_password

 [WARNING]: Module did not set no_log for external_host_password

.Pausing for 30 seconds
(ctrl+C then 'C' = continue early, ctrl+C then 'A' = abort)
................................................
Play 27/28 (Update master-config for publicLoggingURL)

Play 28/28 (Re-enable excluder if it was previously enabled)
................
localhost                  : ok=11   changed=0    unreachable=0    failed=0   
master1.host.local         : ok=632  changed=186  unreachable=0    failed=0   
node1.host.local           : ok=1    changed=0    unreachable=1    failed=0   
node2.host.local           : ok=212  changed=61   unreachable=0    failed=0   



An error was detected. After resolving the problem please relaunch the
installation process.

```

### Post-Installation tasks

* verifies openshift services is running properly on each nodes

* verifies nodes and base system pods status

* exclude openshift packages on yum repositories again

* tests through processing all phases on openshift pipline














