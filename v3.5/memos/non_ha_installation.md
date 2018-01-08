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
node1.host.local<br/>node2.host.local | OpenShift node servers | nodes runing the pods included containers
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

* Step1: subscription registration

* Step2: installation of required packages for initial setting up

* Step3: disabled the useless services

* Step4: enabled the required services

* Step5: customizes the configurations of target nodes
  * node server's hostname resolution is required
  * the time is synchonized by NTP on all nodes

* docker install and enabled with dedicated block volume

* atomic-openshift-utils installs only on master host











