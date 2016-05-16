.. Licensed to the Apache Software Foundation (ASF) under one
   or more contributor license agreements.  See the NOTICE file
   distributed with this work for additional information#
   regarding copyright ownership.  The ASF licenses this file
   to you under the Apache License, Version 2.0 (the
   "License"); you may not use this file except in compliance
   with the License.  You may obtain a copy of the License at
   http://www.apache.org/licenses/LICENSE-2.0
   Unless required by applicable law or agreed to in writing,
   software distributed under the License is distributed on an
   "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
   KIND, either express or implied.  See the License for the
   specific language governing permissions and limitations
   under the License.


Concepts and Terminology
========================

What is Apache CloudStack?
--------------------------

Apache CloudStack is an open source Infrastructure-as-a-Service platform that 
manages and orchestrates pools of storage, network, and computer resource to 
build a public or private IaaS compute cloud. 

What can Apache CloudStack do?
------------------------------

With CloudStack you can:

-  Set up an on-demand elastic cloud computing service. 
-  Allow end-users to provision resources
-  Carry out all possible functions through the API

Multiple Hypervisor Support
~~~~~~~~~~~~~~~~~~~~~~~~~~~

CloudStack works with a variety of hypervisors and hypervisor-like 
technologies. A single cloud can contain multiple hypervisor implementations. 
As of the current release, CloudStack supports: 

-  KVM
-  vSphere (via vCenter)
-  Xenserver
-  Hyper-V
-  LXC
-  BareMetal (via IPMI)
-  OVM 3

Massively Scalable Infrastructure Management
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

CloudStack can manage tens of thousands of physical servers installed in 
geographically distributed datacenters. The management server service can be
installed in a clustered manner.
Maintenance or other outages of the management server(s) can occur without 
affecting the virtual machines running in the cloud. 


Automatic Cloud Configuration Management
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

CloudStack automatically configures the networking and storage for each virtual 
machine created. CloudStack deploys virtual appliances to provide internal
services. These appliances offer services such as firewalling, routing, DHCP, 
VPN, console proxy, storage access, and storage replication. The extensive use 
of horizontally scalable virtual machines simplifies the installation and ongoing 
operation of a cloud. 

Graphical User Interface
~~~~~~~~~~~~~~~~~~~~~~~~

CloudStack offers a web interface which can be used by administrators for provisioning
and managing the cloud, as well as an end-user's for creating and managing virtual 
machines, networks and VM templates. The UI can be customized to reflect the desired 
service provider or enterprise look and feel.

API
~~~

CloudStack provides a REST-like API for the operation, management and use of 
the cloud. 

AWS EC2 API and GCE Support
~~~~~~~~~~~~~~~~~~~~~~~~~~~

CloudStack can provide both an EC2 and GCE API translation layer to permit the common 
EC2 and GCE tools to be used when communicating with a CloudStack cloud. 

High Availability
~~~~~~~~~~~~~~~~~

CloudStack has a number of features designed to increase the availability of the 
system. The Management Server itself may be deployed in a multi-node clustered
installation supporting physical or virtual load balancers. MySQL may be configured
to use replication to provide failover in the event of database loss. For the 
hosts, CloudStack supports NIC bonding and the use of separate networks for 
storage as well as iSCSI Multipath.


Deployment Architecture Overview
--------------------------------

Generally speaking, most CloudStack deployments consist of the one or more management 
servers, a MySQL database server and the resources to be managed. 

The minimum installation consists of one machine running the CloudStack 
Management Service and a MySQL Database with another machine running a hypervisor
(it is technically possible to run the hypervisor the same machine as the management
service and the database for test or development purposes).

.. image:: _static/images/basic-deployment.png

A more full-featured installation consists of a highly-available multi-node 
Management Server installation and up to tens of thousands of hosts using any 
of several networking technologies.

Management Server Overview
~~~~~~~~~~~~~~~~~~~~~~~~~~

The management service orchestrates and allocates the resources in your cloud 
deployment. We refer to the host of the management service as the management server.

The management service can run on a dedicated physical host or as a virtual 
machine.  It controls the allocation of virtual machines to hosts and assigns 
storage and IP addresses to the virtual machine instances. The Management 
Service runs in an Apache Tomcat container and requires a MySQL database for 
persistence.

The management service:

-  Provides the web interface for both the administrator and end user. 
-  Provides the API interfaces for the CloudStack API.
-  Manages the assignment of guest VMs to a specific compute resource
-  Manages the assignment of public and private IP addresses. 
-  Allocates storage during the VM instantiation process. 
-  Manages snapshots, disk images (templates), and ISO images. 
-  Provides a single point of configuration for your cloud.

Cloud Infrastructure Overview
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Resources within the cloud are managed as follows: 

-  Regions: A collection of one or more geographically proximate zones managed 
   by one or more management servers. 

-  Zones: Typically, a zone is generally mapped to a single datacenter. A zone 
   consists of one or more pods and secondary storage.

-  Pods: A pod is logical construct. Pods are generally used to group clusters
   into failure domains ie. a number of clusters which share the same physical
   switch stack or physical storage array.

-  Clusters: A cluster consists of one or more homogeneous hosts and one or more
   primary storage pools. 

-  Host: A single compute node within a cluster, often a hypervisor. 

-  Primary Storage: A storage resource provided to clusters for the actual running
   of instance disk images. Each primary storage pool is generally attached to a
   specific cluster although zone-wide primary storage is an option, it is not 
   typically used.)

-  Secondary Storage: A zone-wide resource which stores disk templates, ISO 
   images and snapshots. 


Networking Overview
~~~~~~~~~~~~~~~~~~~

CloudStack offers many types of networking, but they typically fall into one 
of two scenarios: 

-  Basic: Most analogous to AWS-classic style networking. This model uses a 
   simple flat layer-2 network per pod where guests can be isolated from each 
   other. The isolation rules are defined through security groups and implemented
   at layer-3 by the hypervisor's bridge device via IP tables. Core routers 
   route traffic between pods.

-  Advanced: This provides isolation between networks rather than between VMs. 
   The isolation is typically provided at layer-2 by VLANs, although SDN 
   technologies such as VMware NSX, Nuage and VXLAN can be utilized.


CloudStack Terminology
----------------------

About Regions
~~~~~~~~~~~~~

To increase availability of a very large cloud, you can optionally group 
resources into geographic regions. A region is the largest available
organizational unit within a CloudStack deployment. A region is made up
of availability zones. Each region is controlled by its own cluster of 
Management Servers, running in one of the zones. The zones in a region are
typically located in relatively close geographical proximity. Regions are a
useful technique for providing fault tolerance and disaster recovery.

By grouping zones into regions, the cloud can achieve higher
availability and scalability. User accounts can span regions, so that
users can deploy VMs in multiple, widely-dispersed regions. Even if one
of the regions becomes unavailable, the services are still available to
the end-user through VMs deployed in another region. And by grouping
communities of zones under their own nearby Management Servers, the
latency of communications within the cloud is reduced compared to
managing widely-dispersed zones from a single central Management Server.

Usage records are consolidated and tracked at the region level, creating 
reports or invoices for each geographic region.

|region-overview.png: Nested structure of a region.|

Regions are visible to the end user. When a user starts a guest VM via a
particular CloudStack Management Server, the user is implicitly
selecting that region for their guest. Users might also be required to
copy their private templates to additional regions to enable creation of
guest VMs using their templates in those regions.


About Zones
~~~~~~~~~~~

A zone is the second largest organizational unit within a CloudStack
deployment. Zones are also referred to as Availability Zones as they usually
follow the concept of 'nothing shared' between Availability Zones. In this 
model, loss of an Availability Zone due to power loss, lightning strike or 
Internet interruption does not effect other zones. Therefore
a zone typically corresponds to a single datacenter.

It is permissible to have multiple zones within a datacenter. 

A zone consists of:

-  One or more pods. Each pod contains one or more clusters of hosts and
   one or more primary storage pools.

-  Secondary storage, which is shared by all the pods in the zone.

|zone-overview.png: Nested structure of a simple zone.|

Zones are visible to the end user. When a user starts a guest VM, the
user must select a zone for their VM. Users might also be required to
copy their private templates to additional zones to enable creation of
guest VMs using their templates in those zones.

Zones can be public or private. Public zones are visible to all users.
This means that any user may create a guest in that zone. Private zones
are reserved for a specific domain. Only users in that domain or its
subdomains may create guests in that zone.

For each zone, the administrator must decide the following.

-  How many pods to place in each zone.
-  How many clusters to place in each pod.
-  How many hosts to place in each cluster.
-  (Optional) How many primary storage servers to place in each zone and
   total capacity for these storage servers.
-  How many primary storage servers to place in each cluster and total
   capacity for these storage servers.
-  How much secondary storage to deploy in a zone.

When you add a new zone using the CloudStack UI, you will be prompted to
configure the zone’s physical network and add the first pod, cluster,
host, primary storage, and secondary storage.

In order to support zone-wide functions for vSphere, CloudStack is aware
of vSphere Datacenters and can map each Datacenter to a CloudStack zone.
To enable features like storage live migration and zone-wide primary
storage for vSphere hosts, CloudStack has to ensure that a zone contains
only a single vSphere Datacenter. Therefore, when you are creating a new 
CloudStack zone, you can select a vSphere Datacenter for the zone. If you 
are provisioning multiple vSphere Datacenters, each one will be set up as 
a single zone in CloudStack.

.. note::
   If you are upgrading from a previous CloudStack version, and your existing 
   deployment contains a zone with clusters from multiple vSphere Datacenters, 
   that zone will not be forcibly migrated to the new model. It will continue 
   to function as before. However, any new zone-wide operations, such as 
   zone-wide primary storage and live storage migration, will not be available 
   in that zone.


About Pods
~~~~~~~~~~

A pod is logical construct. Pods are generally used to group clusters into 
a failure domains ie. a number of clusters which share the same physical switch 
stack or physical storage array. Hosts in the same pod should be in the same 
subnet. A pod is the third-largest organizational unit within a CloudStack 
deployment. Pods are contained within zones. A pod consists of one or more 
clusters of hosts and one or more primary storage servers. Pods are not visible to
the end user.

|pod-overview.png: Nested structure of a simple pod|


About Clusters
~~~~~~~~~~~~~~

A cluster provides a way to group hosts. To be precise, a cluster is a
XenServer server pool, a set of KVM servers, or a VMware cluster
preconfigured in vCenter. The hosts in a cluster all have identical
hardware, run the same hypervisor, are on the same subnet, and access
the same shared primary storage pools. Virtual machine instances (VMs) can be
live-migrated from one host to another within the same cluster, without
interrupting service to the user.

A cluster can consist of one or more hosts and one or more primary storage
pools.

A cluster is the fourth-largest organizational unit within a CloudStack
deployment. Clusters are contained within pods, and pods are contained
within zones. Size of the cluster is limited by the underlying
hypervisor, although the CloudStack recommends fewer in most cases; see
Best Practices.


|cluster-overview.png: Structure of a simple cluster|

CloudStack allows multiple clusters in a cloud deployment.

Even when local storage is used exclusively, clusters are still required
organizationally, even if there is just one host per cluster.

When VMware is used, every VMware cluster is managed by a vCenter
server. An Administrator must register the vCenter server with
CloudStack. There may be multiple vCenter servers per zone. Each vCenter
server may manage multiple VMware clusters.


About Hosts
~~~~~~~~~~~

A host is a single physical server. Hosts provide the computing resources that
run guest virtual machines. Each host has hypervisor software installed
on it to manage the guest VMs. For example, a host can be a Citrix
XenServer server, a Linux KVM-enabled server, an ESXi server, or a
Windows Hyper-V server.

The host is the smallest organizational unit within a CloudStack
deployment. Hosts are contained within clusters, clusters are contained
within pods, pods are contained within zones, and zones can be contained
within regions.

Hosts in a CloudStack deployment:

-  Provide the CPU, memory, storage, and networking resources needed to
   host the virtual machines

-  Are interconnected using a high bandwidth TCP/IP networks.

-  May reside in multiple data centers across different geographic
   locations

-  May have different capacities (different CPU speeds, different
   amounts of RAM, etc.), although the hosts within a cluster must
   be homogeneous

Additional hosts can be added at any time to provide more capacity for
guest VMs.

CloudStack automatically detects the amount of CPU and memory resources
provided by the hosts.

Hosts are not visible to the end user. An end user cannot determine
which host their guest has been assigned to.

For a host to function in CloudStack, you must do the following:

-  Install hypervisor software on the host

-  Assign a management IP address to the host 

-  Connect the CloudStack Management Service to the host.


About Primary Storage
~~~~~~~~~~~~~~~~~~~~~

Primary storage pools store the virtual disks for all of the VMs running on 
the hosts. They can be either shared or local. 

Shared pools are accessed by a number of hosts simulataneously and each
pool is generally associated with a specific cluster where they store the 
virtual disks for the VMs running on the hosts in that cluster.
On KVM and VMware, you can provision primary storage on a per-zone basis.

Local primary storage pools are pools of storage on the hypervisors themselves
(one pool per host).  

At least one primary storage pool is required in each cluster (or potentially
zone).  It is typically located close to the hosts for increased performance. 
CloudStack manages the allocation of guest virtual disks to particular primary 
storage pools.

It is useful to set up zone-wide primary storage when you want to avoid
extra data copy operations, However it can limit the total number of hosts
which can reside in a particular zone. With cluster-based primary storage, data in
the primary storage is directly available only to VMs within that
cluster. If a VM in a different cluster needs some of the data, it must
be copied from one cluster to another, using the zone's secondary
storage as an intermediate step. This operation can be time-consuming.

For Hyper-V, only SMB/CIFS storage is supported. Note that Zone-wide Primary
Storage is not supported in Hyper-V.

CloudStack is designed to work with all standards-compliant iSCSI and
NFS servers that are supported by the underlying hypervisor, including,
for example:

-  SolidFire for iSCSI

-  Dell EqualLogic™ for iSCSI

-  Network Appliance's filers for NFS and iSCSI

-  Scale Computing for NFS

If you intend to use only local disk for your installation, it is not 
necessary to add separate primary storage.


About Secondary Storage
~~~~~~~~~~~~~~~~~~~~~~~

Secondary storage stores the following:

-  Templates — OS images that can be used to boot VMs and can include
   additional configuration information, such as installed applications

-  ISO images — disc images containing data or bootable media for
   operating systems

-  Disk volume snapshots — saved copies of VM disks which can be used for
   data recovery or to create new templates

The items in secondary storage are available to all of the hosts in the scope
of the secondary storage, which may be set as per zone or per region.

To make items in secondary storage available to all hosts throughout the
cloud, you can add object storage in conjunction with a the zone-based NFS
Secondary Staging Store. In this configuration it is not necessary to copy 
templates and snapshots from one zone to another, as would be required when 
using zone NFS alone.

For Hyper-V hosts, only SMB/CIFS storage is supported.

CloudStack provides plugins that enable both OpenStack Object Storage
(Swift, `swift.openstack.org <http://swift.openstack.org>`__) and Amazon
Simple Storage Service (S3) compliant object storage. When using one of these
storage plugins, you configure Swift or S3 storage for the entire
CloudStack, then set up the NFS Secondary Staging Store for each zone.
The NFS storage in each zone acts as a staging area through which all
templates and other secondary storage data pass before being forwarded
to Swift or S3. The backing object storage acts as a cloud-wide
resource, making templates and other data available to any zone in the
cloud.

.. warning::
   Heterogeneous Secondary Storage is not supported in Regions. For example, 
   you cannot set up multiple zones, one using NFS secondary and the other 
   using S3 or Swift secondary.


About Physical Networks
~~~~~~~~~~~~~~~~~~~~~~~

Part of adding a zone is describing the physical networking topology of your 
connected hosts. One or (in an
advanced zone) more physical networks can be associated with each zone.

Physical Networks are slightly confusingly named – may be better to call them 
Network types or groups.
Physically independent network interfaces don’t have to be different ‘physical networks’ unless:

-  They use different separation techniques VLAN vs VXLAN
-  You have multiple physical guest networks   

Each physical network can carry one or more types of network traffic. 
The choices of traffic type for each network vary depending on whether
you are creating a zone with basic networking or advanced networking.

A zone can have multiple physical networks. An administrator can:

-  Add/Remove/Update physical networks in a zone

-  Configure VLANs on the physical network

-  Configure labels so that the network traffic can be mapped to the 
   networks on the hypervisors

-  Configure the service providers (firewalls, load balancers, etc.)
   available on a physical network

-  Configure the IP addresses trunked to a physical network

-  Specify what type of traffic is carried on the physical network, as
   well as other properties like network speed


Basic Zone Network Traffic Types
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

When basic networking is used, there can be only one physical network in
the zone. That physical network carries the following traffic types:

-  Guest. When end users run VMs, they generate guest traffic. The guest
   VMs communicate with each other over a network that can be referred
   to as the guest network. Each pod in a basic zone is a broadcast
   domain, and therefore each pod has a different IP range for the guest
   network. The administrator must configure the IP range for each pod.

-  Management. When CloudStack's internal resources communicate with
   each other, they generate management traffic. This includes
   communication between hosts, system VMs (VMs used by CloudStack to
   perform various tasks in the cloud), and any other component that
   communicates directly with the CloudStack Management Server. You must
   configure the IP range for the system VMs to use.

.. note::
   We strongly recommend the use of separate NICs for management traffic
   and guest traffic.

-  Public. Public traffic is generated when VMs in the cloud access the
   Internet. Publicly accessible IPs must be allocated for this purpose.
   End users can use the CloudStack UI to acquire these IPs to implement
   NAT between their guest network and the public network, as described
   in Acquiring a New IP Address.

-  Storage. While labeled "storage" this is specifically about secondary
   storage, and doesn't affect traffic for primary storage. This
   includes traffic such as VM templates and snapshots, which is sent
   between the secondary storage VM and secondary storage servers.
   CloudStack uses a separate Network Interface Controller (NIC) named
   storage NIC for storage network traffic. Use of a storage NIC that
   always operates on a high bandwidth network allows fast template and
   snapshot copying. You must configure the IP range to use for the
   storage network.

In a basic network, configuring the physical network is fairly
straightforward. In most cases, you only need to configure one guest
network to carry traffic that is generated by guest VMs. If you use a
NetScaler load balancer and enable its elastic IP and elastic load
balancing (EIP and ELB) features, you must also configure a network to
carry public traffic. CloudStack takes care of presenting the necessary
network configuration steps to you in the UI when you add a new zone.


Basic Zone Guest IP Addresses
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

When basic networking is used, CloudStack will assign IP addresses in
the CIDR of the pod to the guests in that pod. The administrator must
add a Direct IP range on the pod for this purpose. These IPs are in the
same VLAN as the hosts.


Advanced Zone Network Traffic Types
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

When advanced networking is used, there can be multiple physical
networks in the zone. Each physical network can carry one or more
traffic types, and you need to let CloudStack know which type of network
traffic you want each network to carry. The traffic types in an advanced
zone are:

-  Guest. When end users run VMs, they generate guest traffic. The guest
   VMs communicate with each other over a network that can be referred
   to as the guest network. This network can be isolated or shared. In
   an isolated guest network, the administrator needs to reserve VLAN
   ranges to provide isolation for each CloudStack account’s network
   (potentially a large number of VLANs). In a shared guest network, all
   guest VMs share a single network.

-  Management. When CloudStack’s internal resources communicate with
   each other, they generate management traffic. This includes
   communication between hosts, system VMs (VMs used by CloudStack to
   perform various tasks in the cloud), and any other component that
   communicates directly with the CloudStack Management Server. You must
   configure the IP range for the system VMs to use.

-  Public. Public traffic is generated when VMs in the cloud access the
   Internet. Publicly accessible IPs must be allocated for this purpose.
   End users can use the CloudStack UI to acquire these IPs to implement
   NAT between their guest network and the public network, as described
   in “Acquiring a New IP Address” in the Administration Guide.

-  Storage. While labeled "storage" this is specifically about secondary
   storage, and doesn't affect traffic for primary storage. This
   includes traffic such as VM templates and snapshots, which is sent
   between the secondary storage VM and secondary storage servers.
   CloudStack uses a separate Network Interface Controller (NIC) named
   storage NIC for storage network traffic. Use of a storage NIC that
   always operates on a high bandwidth network allows fast template and
   snapshot copying. You must configure the IP range to use for the
   storage network.

These traffic types can each be on a separate physical network, or they
can be combined with certain restrictions. When you use the Add Zone
wizard in the UI to create a new zone, you are guided into making only
valid choices.


Advanced Zone Guest IP Addresses
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

When advanced networking is used, the administrator can create
additional networks for use by the guests. These networks can span the
zone and be available to all accounts, or they can be scoped to a single
account, in which case only the named account may create guests that
attach to these networks. The networks are defined by a VLAN ID, IP
range, and gateway. The administrator may provision thousands of these
networks if desired. Additionally, the administrator can reserve a part
of the IP address space for non-CloudStack VMs and servers.


Advanced Zone Public IP Addresses
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

When advanced networking is used, the administrator can create
additional networks for use by the guests. These networks can span the
zone and be available to all accounts, or they can be scoped to a single
account, in which case only the named account may create guests that
attach to these networks. The networks are defined by a VLAN ID, IP
range, and gateway. The administrator may provision thousands of these
networks if desired.


System Reserved IP Addresses
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The various system VMs, (Secondary Storage VMs, Console Proxy VMs, and 
in the case of vSphere, the Virtual Routers) require IP adresses in order
for the management server(s) to communicate with them.

In each zone, you need to configure a range of 'reserved' IP addresses 
on the management network in each pod. These IP addresses   This network carries communication between the
CloudStack Management Server 

The reserved IP addresses must be unique across the cloud. You cannot,
for example, have a host in one zone which has the same private IP
address as a host in another zone.

The hosts in a pod are assigned private IP addresses. These are
typically RFC1918 addresses. The Console Proxy and Secondary Storage
system VMs are also allocated private IP addresses in the CIDR of the
pod that they are created in.

Make sure computing servers and Management Servers use IP addresses
outside of the System Reserved IP range. For example, suppose the System
Reserved IP range starts at 192.168.154.2 and ends at 192.168.154.7.
CloudStack can use .2 to .7 for System VMs. This leaves the rest of the
pod CIDR, from .8 to .254, for the Management Server and hypervisor
hosts.

**In all zones:**

Provide private IPs for the system in each pod and provision them in
CloudStack.

For KVM and XenServer, the recommended number of private IPs per pod is
one per host. If you expect a pod to grow, add enough private IPs now to
accommodate the growth.

**In a zone that uses advanced networking:**

For zones with advanced networking, we recommend provisioning enough
private IPs for your total number of customers, plus enough for the
required CloudStack System VMs. Typically, about 10 additional IPs are
required for the System VMs. For more information about System VMs, see
the section on working with SystemVMs in the Administrator's Guide.

When advanced networking is being used, the number of private IP
addresses available in each pod varies depending on which hypervisor is
running on the nodes in that pod. Citrix XenServer and KVM use
link-local addresses, which in theory provide more than 65,000 private
IP addresses within the address block. As the pod grows over time, this
should be more than enough for any reasonable number of hosts as well as
IP addresses for guest virtual routers. VMWare ESXi, by contrast uses
any administrator-specified subnetting scheme, and the typical
administrator provides only 255 IPs per pod. Since these are shared by
physical machines, the guest virtual router, and other entities, it is
possible to run out of private IPs when scaling up a pod whose nodes are
running ESXi.

To ensure adequate headroom to scale private IP space in an ESXi pod
that uses advanced networking, use one or both of the following
techniques:

-  Specify a larger CIDR block for the subnet. A subnet mask with a /20
   suffix will provide more than 4,000 IP addresses.

-  Create multiple pods, each with its own subnet. For example, if you
   create 10 pods and each pod has 255 IPs, this will provide 2,550 IP
   addresses.


.. |1000-foot-view.png: Overview of CloudStack| image:: ./_static/images/1000-foot-view.png
.. |basic-deployment.png: Basic two-machine deployment| image:: ./_static/images/basic-deployment.png
.. |infrastructure_overview.png: Nested organization of a zone| image:: ./_static/images/infrastructure-overview.png
.. |region-overview.png: Nested structure of a region.| image:: ./_static/images/region-overview.png
.. |zone-overview.png: Nested structure of a simple zone.| image:: ./_static/images/zone-overview.png
.. |pod-overview.png: Nested structure of a simple pod| image:: ./_static/images/pod-overview.png
.. |cluster-overview.png: Structure of a simple cluster| image:: ./_static/images/cluster-overview.png
 