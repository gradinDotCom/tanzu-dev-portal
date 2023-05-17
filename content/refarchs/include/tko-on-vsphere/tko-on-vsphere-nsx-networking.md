## Tanzu Kubernetes Clusters Networking

A Tanzu Kubernetes cluster provisioned by Tanzu Kubernetes Grid supports two Container Network Interface (CNI) options:

- [Antrea](https://antrea.io/)
- [Calico](https://www.tigera.io/project-calico/)

Both are open-source software that provide networking for cluster pods, services, and ingress.

When you deploy a Tanzu Kubernetes cluster using Tanzu Mission Control or Tanzu CLI, Antrea CNI is automatically enabled in the cluster.

Tanzu Kubernetes Grid also supports [Multus](https://github.com/k8snetworkplumbingwg/multus-cni) CNI which can be installed through Tanzu user-managed packages. Multus CNI lets you attach multiple network interfaces to a single pod and associate each with a different address range.

To provision a Tanzu Kubernetes cluster using a non-default CNI, see the following instructions:

- [Deploy Tanzu Kubernetes clusters with Calico](https://docs.vmware.com/en/VMware-Tanzu-Kubernetes-Grid/2.1/using-tkg-21/workload-clusters-networking.html?hWord=N4IghgNiBcIMaQJZwPYgL5A)
- [Implement Multiple Pod Network Interfaces with Multus](https://docs.vmware.com/en/VMware-Tanzu-Kubernetes-Grid/2.1/using-tkg-21/workload-packages-multus.html?hWord=N4IghgNiBcIMaQJZwPYgL5A)

Each CNI is suitable for a different use case. The following table lists some common use cases for the three CNIs that Tanzu Kubernetes Grid supports. This table helps you with information on selecting the right CNI in your Tanzu Kubernetes Grid implementation.

|**CNI**|**Use Case**|**Pros and Cons**|
| --- | --- | --- |
|Antrea|<p>Enable Kubernetes pod networking with IP overlay networks using VXLAN or Geneve for encapsulation. Optionally encrypt node-to-node communication using IPSec packet encryption.</p><p></p><p>Antrea supports advanced network use cases like kernel bypass and network service mesh.</p>|<p>Pros</p><p>- Provides an option to configure egress IP pool or static egress IP for Kubernetes workloads.</p>|
|Calico|<p>Calico is used in environments where factors like network performance, flexibility, and power are essential.</p><p></p><p>For routing packets between nodes, Calico leverages the BGP routing protocol instead of an overlay network. This eliminates the need to wrap packets with an encapsulation layer resulting in increased network performance for Kubernetes workloads.</p>|<p>Pros</p><p>- Support for network policies</p><p>- High network performance</p><p>- SCTP support</p><p>Cons</p><p>- No multicast support</p><p></p>|
|Multus|Multus CNI provides multiple interfaces per each Kubernetes pod. Using Multus CRDs, you can specify which pods get which interfaces and allow different interfaces depending on the use case.|<p>Pros</p><p>- Separation of data/control planes.</p><p>- Separate security policies can be used for separate interfaces. </p><p>- Supports SR-IOV, DPDK, OVS-DPDK, and VPP workloads in Kubernetes with both cloud native and NFV based applications in Kubernetes.</p>|

## Tanzu Kubernetes Grid Infrastructure Networking

Tanzu Kubernetes Grid on vSphere can be deployed on various networking stacks including:

- VMware NSX-T Data Center Networking
- vSphere Networking (VDS)

**Note:** The scope of this document is limited to VMware NSX-T Data Center Networking with NSX Advanced Load Balancer Enterprise Edition.

## Tanzu Kubernetes Grid on VMware NSX Data Center Networking with NSX Advanced Load Balancer

When deployed on VMware NSX-T Networking, Tanzu Kubernetes Grid uses the NSX-T logical segments and gateways to provide connectivity to Kubernetes control plane VMs, worker nodes, services, and applications. All hosts from the cluster where Tanzu Kubernetes clusters are deployed are configured as NSX-T transport nodes, which provide network connectivity to the Kubernetes environment.

You can configure NSX Advanced Load Balancer in Tanzu Kubernetes Grid as:

- L4 load balancer for application hosted on the TKG cluster.

- The L7 ingress service provider for the applications in the clusters that are deployed on vSphere.

- L4 load balancer for the control plane API server.

Each workload cluster integrates with NSX Advanced Load Balancer by running an Avi Kubernetes Operator (AKO) on one of its nodes. The cluster’s AKO calls the Kubernetes API to manage the lifecycle of load balancing and ingress resources for its workloads.

## NSX Advanced Load Balancer Components

NSX Advanced Load Balancer is deployed in Write Access Mode in VMware NSX Environment. This mode grants NSX Advanced Load Balancer controllers full write access to vCenter which helps in automatically creating, modifying, and removing service engines (SEs) and other resources as needed to adapt to changing traffic needs. The core components of NSX Advanced Load Balancer are as follows:

- **NSX Advanced Load Balancer Controller** - NSX Advanced Load Balancer controller manages virtual service objects and interacts with the vCenter Server infrastructure to manage the lifecycle of the service engines (SEs). It is the central repository for the configurations and policies related to services and management, and it provides the portal for viewing the health of VirtualServices and SEs and the associated analytics that NSX Advanced Load Balancer provides.
- **NSX Advanced Load Balancer Service Engine** - The service engines (SEs) are lightweight VMs that handle all data plane operations by receiving and executing instructions from the controller. The SEs perform load balancing and all client- and server-facing network interactions.
- **Service Engine Group -** Service engines are created within a group, which contains the definition of how the SEs should be sized, placed, and made highly available. Each cloud has at least one SE group.
- **Cloud -** Clouds are containers for the environment that NSX Advanced Load Balancer is installed or operating within. During the initial setup of NSX Advanced Load Balancer, a default cloud, named `Default-Cloud`, is created. This is where the first controller is deployed into Default-Cloud. Additional clouds may be added containing SEs and virtual services.
- **Avi Kubernetes Operator (AKO)** - It is a Kubernetes operator that runs as a pod in the Supervisor Cluster and Tanzu Kubernetes clusters, and it provides ingress and load balancing functionality. AKO translates the required Kubernetes objects to NSX Advanced Load Balancer objects and automates the implementation of ingresses, routes, and services on the service engines (SE) through the NSX Advanced Load Balancer Controller.
- **AKO Operator (AKOO)** - This is an operator which is used to deploy, manage, and remove the AKO pod in Kubernetes clusters. This operator when deployed creates an instance of the AKO controller and installs all the relevant objects like:
  - AKO `Statefulset`
  - `Clusterrole` and `Clusterrolebinding`
  - `Configmap` (required for the AKO controller and other artifacts).

Tanzu Kubernetes Grid management clusters have an AKO operator installed out of the box during cluster deployment. By default, a Tanzu Kubernetes Grid management cluster has a couple of AkoDeploymentConfig created which dictate when and how AKO pods are created in the workload clusters. For more information, see [AKO Operator documentation](https://github.com/vmware/load-balancer-and-ingress-services-for-kubernetes/tree/master/ako-operator).

Each environment configured in NSX Advanced Load Balancer is referred to as a cloud. Each cloud in NSX Advanced Load Balancer maintains networking and service engine settings. The cloud is configured with one or more VIP networks to provide IP addresses for load balancing (L4/L7) virtual services created under that cloud.

The virtual services can be spanned across multiple service Engines if the associated Service Engine Group is configured in Active/Active HA mode. A service engine can belong to only one SE group at a time.

IP address allocation for virtual services can be over DHCP or through NSX Advanced Load Balancer in-built IPAM functionality. The VIP networks created or configured in NSX Advanced Load Balancer are associated with the IPAM profile.

## Network Architecture

For the deployment of Tanzu Kubernetes Grid in the VMware NSX environment, it is required to build separate networks for the Tanzu Kubernetes Grid management cluster and workload clusters, NSX Advanced Load Balancer management, and cluster-VIP network for control plane HA.

The network reference design can be mapped into this general framework.

![TKG with NSX-T Data Center Networking general network layout](refarchs/tko-on-vsphere-nsx/images/tko-on-vsphere-nsxt-3.png)

This topology enables the following benefits:

- Isolate and separate SDDC management components (vCenter, ESX) from the Tanzu Kubernetes Grid components. This reference design allows only the minimum connectivity between the Tanzu Kubernetes Grid clusters and NSX Advanced Load Balancer to the vCenter Server.
- Isolate and separate NSX Advanced Load Balancer management network from the Tanzu Kubernetes Grid management segment and the Tanzu Kubernetes Grid workload segments.
- Depending on the workload cluster type and use case, multiple workload clusters may leverage the same workload network or new networks can be used for each workload cluster. To isolate and separate Tanzu Kubernetes Grid workload cluster networking from each other it’s recommended to make use of separate networks for each workload cluster and configure the required firewall between these networks. For more information, see [Firewall Requirements](#fwreq).
- Separate provider and tenant access to the Tanzu Kubernetes Grid environment.
  - Only provider administrators need access to the Tanzu Kubernetes Grid management cluster. This prevents tenants from attempting to connect to the Tanzu Kubernetes Grid management cluster.

### <a id="netreq"> </a> Network Requirements

As per the defined architecture, the list of required networks follows:

|**Network Type**|**DHCP Service**|<p>**Description & Recommendations**</p><p></p>|
| --- | --- | --- |
|NSX ALB Management Logical Segment|Optional|<p>NSX ALB controllers and SEs are attached to this network. </p><p></p><p>DHCP is not a mandatory requirement on this network as NSX ALB can handle IPAM services for a given network.</p>|
|TKG Management Logical Segment|Yes|Control plane and worker nodes of TKG management cluster and shared service clusters are attached to this network.|
|TKG Shared Service Logical Segment|Yes|Control plane and worker nodes of TKG shared services cluster are attached to this network.|
|TKG Workload Logical Segment|Yes|Control plane and worker nodes of TKG workload clusters are attached to this network.|
|TKG Cluster VIP Logical Segment|No|Virtual services for control plane HA of all TKG clusters (management, shared services, and workload)<br>Reserve sufficient IP addresses depending on the number of TKG clusters planned to be deployed in the environment, NSX Advanced Load Balancer takes care of IPAM on this network.|

#### <a id="cidr"> </a> Subnet and CIDR Examples

For this demonstration, this document makes use of the following subnet CIDR for Tanzu for Kubernetes Operations deployment.

|**Network Type**|**Segment Name**|**Gateway CIDR**|**DHCP Pool in NSXT**|**NSX ALB IP Pool**|
| --- | --- | --- | --- | --- |
|NSX ALB Management Network|alb-mgmt-ls|172.19.71.1/27|N/A|172.19.71.6 - 172.19.71.30|
|TKG Cluster VIP Network|tkg-cluster-vip|172.19.75.1/26|N/A|172.19.75.2 - 172.19.75.60|
|TKG Management Network|tkg-mgmt-ls|172.19.72.1/27|172.19.72.2 - 172.19.72.30|N/A|
|TKG Shared Service Network|tkg-ss-ls|172.19.73.1/27|172.19.73.2 - 172.19.73.30|N/A|
|TKG Workload Network|tkg-workload-ls|172.19.77.1/24|172.19.77.2- 172.19.77.251|N/A|

### <a id="fwreq"> </a> Firewall Requirements
To prepare the firewall, you need to gather the following information:

1. NSX ALB Controller nodes and Cluster IP address.
2. NSX ALB Management Network CIDR.
3. TKG Management Network CIDR
4. TKG Shared Services Network CIDR
5. TKG Workload Network CIDR
6. TKG Cluster VIP Address Range
7. Client Machine IP Address
8. Bootstrap machine IP Address
9. Harbor registry IP address
10. vCenter Server IP.
11. DNS server IP(s).
12. NTP Server(s).
13. NSX-T nodes and VIP address.

|**Source**|**Destination**|**Protocol:Port**|**Description**|
| --- | --- | --- | --- |
|<p>TKG management network CIDR</p><p></p><p>TKG shared services network CIDR</p><p></p><p>TKG workload network CIDR</p>|DNS Server<br>NTP Server|UDP:53<br>UDP:123|DNS Service <br>Time Synchronization|
|<p>TKG management network CIDR</p><p></p><p>TKG shared services network CIDR</p><p></p><p>TKG workload network CIDR</p>|vCenter IP|TCP:443|Allows components to access vCenter to create VMs and storage volumes|
|<p>TKG management network CIDR</p><p></p><p>TKG shared services network CIDR</p><p></p><p>TKG workload network CIDR</p>|Harbor Registry|TCP:443|<p>Allows components to retrieve container images </p><p>This registry can be a local or a public image registry (projects.registry.vmware.com)</p>|
|<p>TKG management network CIDR</p><p></p><p>TKG shared services network CIDR</p><p></p><p>TKG workload network CIDR</p>|TKG Cluster VIP Network |TCP:6443|For management cluster to configure workload cluster<br>Allow shared cluster to register with management cluster<br>Allow workload cluster to register with management cluster|
|<p>TKG management network CIDR</p><p></p><p>TKG shared services network CIDR</p><p></p><p>TKG workload network CIDR</p>|NSX ALB controllers (NSX ALB management network)|TCP:443|Allow Avi Kubernetes Operator (AKO) and AKO Operator (AKOO) access to NSX ALB controller|
|NSX ALB controllers (NSX ALB Management Network)|vCenter and ESXi hosts|TCP:443|Allow NSX ALB to discover vCenter objects and deploy SEs as required|
|NSX Advanced Load Balancer management network CIDR|<p>DNS Server</p><p>NTP Server</p>|<p>UDP:53</p><p>UDP:123</p>|<p>DNS Service</p><p>Time synchronization</p>|
|NSX ALB controllers (NSX ALB Management Network)|NSX-T nodes and VIP address|TCP:443|Allow NSX ALB to discover vCenter objects and deploy SEs as required|
|Admin network|Bootstrap VM|SSH:22|To deploy, manage  and configure TKG clusters|
|deny-all|any|any|deny|
