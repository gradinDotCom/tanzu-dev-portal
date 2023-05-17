## Tanzu Kubernetes Clusters Networking

A Tanzu Kubernetes cluster provisioned by the Tanzu Kubernetes Grid supports the following Container Network Interface (CNI) options:

- [Antrea](https://antrea.io/)
- [Calico](https://www.tigera.io/project-calico/)

The CNI options are open-source software that provide networking for cluster pods, services, and ingress.

When you deploy a Tanzu Kubernetes cluster using the default configuration of Tanzu CLI, Antrea CNI is automatically enabled in the cluster.

To provision a Tanzu Kubernetes cluster using Calico CNI, see [Deploy Tanzu Kubernetes clusters with Calico](https://docs.vmware.com/en/VMware-Tanzu-Kubernetes-Grid/2.1/using-tkg-21/workload-clusters-networking.html?hWord=N4IghgNiBcIMaQJZwPYgL5A)

Each CNI is suitable for a different use case. The following table lists some common use cases for the CNI options that Tanzu Kubernetes Grid supports. This table will help you select the most appropriate CNI for your Tanzu Kubernetes Grid implementation.

|**CNI**|**Use Case**|**Pros and Cons**|
| --- | --- | --- |
|Antrea|<p>Enable Kubernetes pod networking with IP overlay networks using VXLAN or Geneve for encapsulation. Optionally encrypt node-to-node communication using IPSec packet encryption.</p><p></p><p>Antrea supports advanced network use cases like kernel bypass and network service mesh.</p>|<p>Pros</p><p>- Antrea leverages Open vSwitch as the networking data plane. Open vSwitch supports both Linux and Windows.</p><p>- VMware supports the latest conformant Kubernetes and stable releases of Antrea.</p>|
|Calico|<p>Calico is used in environments where factors like network performance, flexibility, and power are essential.</p><p></p><p>For routing packets between nodes, Calico leverages the BGP routing protocol instead of an overlay network. This eliminates the need to wrap packets with an encapsulation layer resulting in increased network performance for Kubernetes workloads.</p>|<p>Pros</p><p>- Support for Network Policies</p><p>- High network performance</p><p>- SCTP Support</p><p>Cons</p><p>- No multicast support</p><p></p>|

## Networking for vSphere with Tanzu

You can deploy vSphere with Tanzu on various networking stacks, including:

- VMware NSX-T Data Center Networking.

- vSphere Virtual Distributed Switch (VDS) Networking with NSX Advanced Load Balancer.

**Note:** The scope of this discussion is limited to vSphere Networking (VDS) with NSX Advanced Load Balancer.

### vSphere with Tanzu on vSphere Networking with NSX Advanced Load Balancer

In a vSphere with Tanzu environment, a Supervisor Cluster configured with vSphere networking uses distributed port groups to provide connectivity to Kubernetes control plane VMs, services, and workloads. All hosts from the cluster, which is enabled for vSphere with Tanzu, are connected to the distributed switch that provides connectivity to Kubernetes workloads and control plane VMs.

You can use one or more distributed port groups as Workload Networks. The network that provides connectivity to the Kubernetes Control Plane VMs is called Primary Workload Network. You can assign this network to all the namespaces on the Supervisor Cluster, or you can use different networks for each namespace. The Tanzu Kubernetes clusters connect to the Workload Network that is assigned to the namespace.

The Supervisor Cluster leverages NSX Advanced Load Balancer (NSX ALB) to provide L4 load balancing for the Tanzu Kubernetes clusters control-plane HA. Users access the applications by connecting to the Virtual IP address (VIP) of the applications provisioned by NSX Advanced Load Balancer.

The following diagram shows a general overview for vSphere with Tanzu on vSphere Networking.

![Overview diagram of vSphere with Tanzu on vSphere Networking](refarchs/tko-on-vsphere-with-tanzu/images/tko-vwt07.jpeg)

### NSX Advanced Load Balancer Components

NSX Advanced Load Balancer is deployed in write access mode in a vSphere environment. This mode grants NSX Advanced Load Balancer Controllers full write access to the vCenter, which helps in automatically creating, modifying, and removing SEs and other resources as needed to adapt to changing traffic needs. The following are the core components of NSX Advanced Load Balancer:

-  **NSX Advanced Load Balancer Controller:** NSX Advanced Load Balancer Controller manages Virtual Service objects and interacts with the vCenter Server infrastructure to manage the lifecycle of the service engines (SEs). It is the central repository for the configurations and policies related to services and management and provides the portal for viewing the health of VirtualServices and SEs and the associated analytics that NSX Advanced Load Balancer provides.

-  **NSX Advanced Load Balancer Service Engine:** NSX Advanced Load Balancer Service Engines (SEs) are lightweight VMs that handle all data plane operations by receiving and executing instructions from the controller. The SEs perform load balancing and all client and server-facing network interactions.

-  **Avi Kubernetes Operator (AKO):** Avi Kubernetes Operator is a Kubernetes operator that runs as a pod in the Supervisor Cluster. It provides ingress and load balancing functionality. Avi Kubernetes Operator translates the required Kubernetes objects to NSX Advanced Load Balancer objects and automates the implementation of ingresses/routes/services on the Service Engines (SE) via the NSX Advanced Load Balancer Controller.

Each environment configured in NSX Advanced Load Balancer is referred to as a cloud. Each cloud in NSX Advanced Load Balancer maintains networking and NSX Advanced Load Balancer Service Engine settings. Each cloud is configured with one or more VIP networks to provide IP addresses to L4 load balancing virtual services created under that cloud.

The virtual services can be spanned across multiple Service Engines if the associated Service Engine Group is configured in Active/Active HA mode. A Service Engine can belong to only one Service Engine group at a time.

IP address allocation for virtual services can be over DHCP or via NSX Advanced Load Balancer in-built IPAM functionality. The VIP networks created/configured in NSX Advanced Load Balancer are associated with the IPAM profile.

## Network Architecture

To deploy vSphere with Tanzu, build separate networks for the Tanzu Kubernetes Grid management (Supervisor) cluster, Tanzu Kubernetes Grid workload clusters, NSX Advanced Load Balancer components, and the Tanzu Kubernetes Grid control plane HA.

The network reference design can be mapped into this general framework.

![Diagram of network reference design](refarchs/tko-on-vsphere-with-tanzu/images/tko-vwt08.jpg)

**Note:** The network/portgroup designated for the workload cluster, carries both data and control traffic. Firewalls cannot be utilized to segregate traffic between workload clusters; instead, the underlying CNI must be employed as the main filtering system. Antrea CNI has the Custom Resource Definitions (CRDs) for firewall rules that can be enforce before Kubernetes network policy is added.

**Note:** Based on your requirements, you can create additional networks for your workload cluster. These networks are also referred to as vSphere with Tanzu workload secondary network.

This topology enables the following benefits:

- Isolate and separate SDDC management components (vCenter, ESX) from the vSphere with Tanzu components. This reference design allows only the minimum connectivity between the Tanzu Kubernetes Grid clusters and NSX Advanced Load Balancer to the vCenter Server.

- Isolate and separate the NSX Advanced Load Balancer management network from the supervisor cluster network and the Tanzu Kubernetes Grid workload networks.

- Separate vSphere Admin and Tenant access to the supervisor cluster. This prevents tenants from attempting to connect to the supervisor cluster.

- Allow tenants to access only their own workload cluster(s) and restrict access to this cluster from other tenants. This separation can be achieved by assigning permissions to the supervisor namespaces.

- Depending on the workload cluster type and use case, multiple workload clusters may leverage the same workload network or new networks can be used for each workload cluster.

**Network Requirements**

As per the reference architecture, the list of required networks is as follows:

|**Network Type**|**DHCP Service**|**Description**|
| --- | --- | --- |
|NSX Advanced Load Balancer Management Network|Optional|<p>NSX Advanced Load Balancer controllers and SEs will be attached to this network. </p> |
|TKG Management Network|Optional |Supervisor Cluster nodes will be attached to this network.|
|TKG Workload Network (Primary)|Optional |<p>Control plane and worker nodes of TKG workload clusters will be attached to this network.</p><p>The second interface of the Supervisor nodes is also attached to this network.</p>|
|TKG Cluster VIP/Data Network|No|<p>Virtual Services (L4) for Control plane HA of all TKG clusters (Supervisor and Workload).</p><p>Reserve sufficient IPs depending on the number of TKG clusters planned to be deployed in the environment.</p>|

## Subnet and CIDR Examples

For the purpose of demonstration, this document makes use of the following Subnet CIDR for TKO deployment.

|**Network Type**|**Segment Name**|**Gateway CIDR**|**DHCP Pool**|**NSX Advanced Load Balancer IP Pool**|
| --- | --- | --- | --- | --- |
|NSX Advanced Load Balancer Mgmt Network|NSX-Advanced Load Balancer-Mgmt|192.168.11.1/27|NA|192.168.11.14 - 192.168.11.30|
|Supervisor Cluster Network|TKG-Management|192.168.12.1/28|192.168.12.2 - 192.168.12.14|NA|
|TKG Workload Primary Network|TKG-Workload-PG01|192.168.13.1/24|192.168.13.2 - 192.168.13.251|NA|
|TKG Cluster VIP/Data Network|TKG-Cluster-VIP|192.168.14.1/26|NA|<p>SE Pool: </p><p></p><p>192.168.14.2 - 192.168.14.20</p><p></p><p>TKG Cluster VIP Range: </p><p>192.168.14.21 - 192.168.14.60</p>|

## Firewall Requirements
To prepare the firewall, you need the following information:

1. NSX Advanced Load Balancer Controller node and VIP addresses
1. NSX Advanced Load Balancer Service Engine management IP address
1. Supervisor Cluster network (Tanzu Kubernetes Grid Management) CIDR
1. Tanzu Kubernetes Grid workload cluster CIDR
1. Tanzu Kubernetes Grid cluster VIP address range
1. Client machine IP address
1. vCenter server IP address
1. VMware Harbor registry IP address
1. DNS server IP address(es)
1. NTP server IP address(es)

The following table provides a list of firewall rules based on the assumption that there is no firewall within a subnet/VLAN.

|**Source**|**Destination**|**Protocol:Port**|**Description**|
| --- | --- | --- | --- |
|Client Machine|NSX Advanced Load Balancer Controller Nodes and VIP|TCP:443|Access NSX Advanced Load Balancer portal for configuration.|
|Client Machine|vCenter Server|TCP:443|Access and configure WCP in vCenter.|
|Client Machine|TKG Cluster VIP Range|<p>TCP:6443</p><p>TCP:443</p><p>TCP:80</p>|<p>TKG Cluster Access</p><p>Access https workload</p><p>Access http workload</p>|
|<p>Client Machine</p><p>(optional)</p>|<p>\*.tmc.cloud.vmware.com</p><p>console.cloud.vmware.com</p>|TCP:443|Access TMC portal, etc.|
|TKG Management and Workload Cluster CIDR|<p>DNS Server</p><p>NTP Server</p>|<p>TCP/UDP:53</p><p>UDP:123</p>|<p>DNS Service</p><p>Time Synchronization</p>|
|TKG Management Cluster CIDR|vCenter IP|TCP:443|Allow components to access vCenter to create VMs and Storage Volumes|
|TKG Management and Workload Cluster CIDR|NSX Advanced Load Balancer controller nodes|TCP:443|Allow Avi Kubernetes Operator (AKO) and AKO Operator (AKOO) access to NSX Advanced Load Balancer Controller|
|TKG Management and Workload Cluster CIDR|TKG Cluster VIP Range|TCP:6443|Allow Supervisor cluster to configure workload clusters|
|TKG Management and Workload Cluster CIDR|Image Registry (Harbor) (If Private)|TCP:443|Allow components to retrieve container images.|
|TKG Management and Workload Cluster CIDR|<p>wp-content.vmware.com</p><p>\*.tmc.cloud.vmware.com</p><p>Projects.registry.vmware.com</p>|TCP:443|Sync content library, pull TKG binaries, and interact with TMC|
|TKG Management cluster CIDR|TKG Workload Cluster CIDR|TCP:6443|VM Operator and TKC VM communication|
|TKG Workload Cluster CIDR|TKG Management Cluster CIDR|TCP:6443|Allow the TKG workload cluster to register with the Supervisor Cluster|
|NSX Advanced Load Balancer Management Network|vCenter and ESXi Hosts|TCP:443|Allow NSX Advanced Load Balancer to discover vCenter objects and deploy SEs as required|
|NSX Advanced Load Balancer Controller Nodes|<p>DNS Server</p><p>NTP Server</p>|<p>TCP/UDP:53</p><p>UDP:123</p>|<p>DNS Service</p><p>Time Synchronization</p>|
|TKG Cluster VIP Range|TKG Management Cluster CIDR|TCP:6443|To interact with supervisor cluster|
|TKG Cluster VIP Range|TKG Workload Cluster CIDR|<p>TCP:6443</p><p>TCP:443</p><p>TCP:80</p>|To interact with workload cluster and K8s applications|
|vCenter Server|TKG Management Cluster CIDR|<p>TCP:443</p><p>TCP:6443</p><p>TCP:22 (optional)</p>||

**Note:** For TMC, if the firewall does not allow wildcards, all IP addresses of [account].tmc.cloud.vmware.com and extensions.aws-usw2.tmc.cloud.vmware.com need to be whitelisted.
