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

## Installation steps

### Pre-installation tasks

* Step1: subscription registration

* Step2: installatino of required packages for initial setting up

* Step3: disabled the useless services

* Step4: enabled the required services

* Step5: customizes the configurations of target nodes
  * node server's hostname resolution is required
  * the time is synchonized by NTP on all nodes
  









