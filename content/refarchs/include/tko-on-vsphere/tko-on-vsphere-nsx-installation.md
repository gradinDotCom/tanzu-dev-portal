## Installation Experience

Tanzu Kubernetes Grid management cluster is the first component that you deploy to get started with Tanzu Kubernetes Grid.

You can deploy the management cluster in two ways:

- Run the Tanzu Kubernetes Grid installer, a wizard interface that guides you through the process of deploying a management cluster. This is the recommended method if you are installing a Tanzu Kubernetes Grid management cluster for the first time.
- Create and edit YAML configuration files, and use them to deploy a management cluster with the CLI commands.

The Tanzu Kubernetes Grid Installation user interface shows that, in the current version, it is possible to install Tanzu Kubernetes Grid on vSphere (including VMware Cloud on AWS), AWS EC2, and Microsoft Azure. The UI provides a guided experience tailored to the IaaS, in this case, VMware vSphere.

![Tanzu for Kubernetes Grid installer welcome screen](refarchs/tko-on-vsphere-nsx/images/tko-on-vsphere-nsxt-4.png)

The installation of Tanzu Kubernetes Grid on vSphere is done through the same UI as mentioned above but tailored to a vSphere environment.

![Tanzu for Kubernetes Grid installer UI for vSphere](refarchs/tko-on-vsphere-nsx/images/tko-on-vsphere-nsxt-5.png)

This installation process takes you through the setup of a management cluster on your vSphere environment. Once the management cluster is deployed, you can make use of [Tanzu Mission Control](https://tanzu.vmware.com/mission-control) or Tanzu CLI to deploy Tanzu Kubernetes shared service and workload clusters.

### Kubernetes Ingress Routing

The default installation of Tanzu Kubernetes Grid does not have any ingress controller installed. Users can use Contour, which is available for installation through Tanzu Packages, or any third-party ingress controller of their choice.

Contour is an open-source controller for Kubernetes ingress routing. Contour can be installed in the shared services cluster on any Tanzu Kubernetes Cluster. Deploying Contour is a prerequisite if you want to deploy Prometheus, Grafana, and Harbor packages on a workload cluster.

For more information about Contour, see the [Contour website](https://projectcontour.io/) and [Implementing Ingress Control with Contour](https://docs.vmware.com/en/VMware-Tanzu-Kubernetes-Grid/2.1/using-tkg-21/workload-packages-contour.html).

Another option is to use the NSX Advanced Load Balancer Kubernetes ingress controller that offers an advanced L4-L7 load balancing/ingress for containerized applications that are deployed in the Tanzu Kubernetes workload cluster.

![NSX Advanced Load Balancing capabilities for VMware Tanzu](refarchs/tko-on-vsphere-nsx/images/tko-on-vsphere-nsxt-6.png)

For more information about the NSX Advanced Load Balancer ingress controller, see [Configuring L7 Ingress with NSX Advanced Load Balancer](https://docs.vmware.com/en/VMware-Tanzu-Kubernetes-Grid/2.1/tkg-deploy-mc-21/mgmt-reqs-network-nsx-alb-ingress.html).

[Tanzu Service Mesh](https://tanzu.vmware.com/service-mesh), which is a SaaS offering for modern applications running across multi-cluster, multi-clouds, also offers an ingress controller based on [Istio](https://istio.io/).

The following table provides general recommendations on when you should use a specific ingress controller for your Kubernetes environment.

|**Ingress Controller**|**Use Cases**|
| --- | --- |
|Contour|<p>Use Contour when only north-south traffic is needed in a Kubernetes cluster. You can apply security policies for the north-south traffic by defining the policies in the application's manifest file.</p><p></p><p>It's a reliable solution for simple Kubernetes workloads. </p>|
|Istio|Use Istio ingress controller when you intend to provide security, traffic direction, and insights within the cluster (east-west traffic) and between the cluster and the outside world (north-south traffic).|
|NSX ALB ingress controller|<p>Use NSX ALB ingress controller when a containerized application requires features like local and global server load balancing (GSLB), web application firewall (WAF), performance monitoring, and so on. </p><p></p>|

### NSX Advanced Load Balancer (ALB) as an L4+L7 Ingress Service Provider

NSX Advanced Load Balancer provides an L4+L7 load balancing solution for vSphere. It includes a Kubernetes operator that integrates with the Kubernetes API to manage the lifecycle of load balancing and ingress resources for workloads.

Legacy ingress services for Kubernetes include multiple disparate solutions. The services and products contain independent components that are difficult to manage and troubleshoot. The ingress services have reduced observability capabilities with little analytics, and they lack comprehensive visibility into the applications that run on the system. Cloud-native automation is difficult in the legacy ingress services.

In comparison to the legacy Kubernetes ingress services, NSX Advanced Load Balancer has comprehensive load balancing and ingress services features. As a single solution with a central control, NSX Advanced Load Balancer is easy to manage and troubleshoot. NSX Advanced Load Balancer supports real-time telemetry with an insight into the applications that run on the system. The elastic auto-scaling and the decision automation features highlight the cloud-native automation capabilities of NSX Advanced Load Balancer.

NSX Advanced Load Balancer also lets you configure L7 ingress for your workload clusters by using one of the following options:

- L7 ingress in ClusterIP mode
- L7 ingress in NodePortLocal mode
- L7 ingress in NodePort mode
- NSX Advanced Load Balancer L4 ingress with Contour L7 ingress

#### L7 Ingress in ClusterIP Mode

This option enables NSX Advanced Load Balancer L7 ingress capabilities, including sending traffic directly from the service engines (SEs) to the pods, preventing multiple hops that other ingress solutions need when sending packets from the load balancer to the right node where the pod runs. The NSX Advanced Load Balancer controller creates a virtual service with a backend pool with the pod IPs which helps to send the traffic directly to the pods.

However, each workload cluster needs a dedicated SE group for Avi Kubernetes Operator (AKO) to work, which could increase the number of SEs you need for your environment. This mode is used when you have a small number of workload clusters.

#### L7 Ingress in NodePort Mode

The NodePort mode is the default mode when AKO is installed on Tanzu Kubernetes Grid. This option allows your workload clusters to share SE groups and is fully supported by VMware. With this option, the services of your workloads must be set to NodePort instead of ClusterIP even when accompanied by an ingress object. This ensures that NodePorts are created on the worker nodes and traffic can flow through the SEs to the pods via the NodePorts. Kube-Proxy, which runs on each node as DaemonSet, creates network rules to expose the application endpoints to each of the nodes in the format “NodeIP:NodePort”. The NodePort value is the same for a service on all the nodes. It exposes the port on all the nodes of the Kubernetes Cluster, even if the pods are not running on it.

#### L7 Ingress in NodePortLocal Mode

This feature is supported only with Antrea CNI. You must enable this feature on a workload cluster before its creation. The primary difference between this mode and the NodePort mode is that the traffic is sent directly to the pods in your workload cluster through node ports without interfering Kube-proxy. With this option, the workload clusters can share SE groups. Similar to the ClusterIP Mode, this option avoids the potential extra hop when sending traffic from the NSX Advanced Load Balancer SEs to the pod by targeting the right nodes where the pods run.

Antrea agent configures NodePortLocal port mapping rules at the node in the format “NodeIP:Unique Port” to expose each pod on the node on which the pod of the service is running. The default range of the port number is 61000-62000. Even if the pods of the service are running on the same Kubernetes node, Antrea agent publishes unique ports to expose the pods at the node level to integrate with the load balancer.

#### NSX Advanced Load Balancer L4 Ingress with Contour L7 Ingress

This option does not have all the NSX Advanced Load Balancer L7 ingress capabilities but uses it for L4 load balancing only and leverages Contour for L7 Ingress. This also allows sharing SE groups across workload clusters. This option is supported by VMware and it requires minimal setup.

## Design Recommendations

### NSX Advanced Load Balancer Recommendations

The following table provides the recommendations for configuring NSX Advanced Load Balancer in a vSphere environment backed by NSX-T networking.

|**Decision ID**|**Design Decision**|**Design Justification**|**Design Implications**|
| --- | --- | --- | --- |
|TKO-ALB-001|Deploy NSX ALB controller cluster nodes on a network dedicated to NSX ALB.|Isolate NSX ALB traffic from infrastructure management traffic and Kubernetes workloads.|Additional Network (VLAN) is required.|
|TKO-ALB-002|Deploy 3 NSX ALB controller nodes.|To achieve high availability for the NSX ALB platform.<br>In clustered mode, NSX ALB availability is not impacted by an individual controller node failure. The failed node can be removed from the cluster and redeployed if recovery is not possible.|Clustered mode requires more Compute and Storage resources. |
|TKO-ALB-003|Initial setup should be done only on one NSX Advanced Load Balancer controller VM out of the three deployed to create an NSX Advanced Load Balancer controller cluster.|NSX Advanced Load Balancer controller cluster is created from an initialized NSX Advanced Load Balancer controller which becomes the cluster leader.<br> Follower NSX Advanced Load Balancer controller nodes need to be uninitialized to join the cluster.|NSX Advanced Load Balancer controller cluster creation fails if more than one NSX Advanced Load Balancer controller is initialized.|
|TKO-ALB-004|Use static IP addresses for the NSX ALB controllers.|NSX ALB controller cluster uses management IP addresses to form and maintain quorum for the control plane cluster. Any changes to management IP addresses are disruptive.|NSX ALB Controller control plane might go down if the management IP addresses of the controller node change.|
|TKO-ALB-005|Use NSX ALB IPAM for service engine data network and virtual services. |Guarantees IP address assignment for service engine data NICs and virtual services.|None|
|TKO-ALB-006|Reserve an IP address in the NSX ALB management subnet to be used as the cluster IP address for the controller cluster.|NSX ALB portal is always accessible over cluster IP address regardless of a specific individual controller node failure.|None|
|TKO-ALB-007|Shared service engines for the same type of workload (dev/test/prod) clusters.|Minimize the licensing cost.|<p>Each service engine contributes to the CPU core capacity associated with a license.</p><p></p><p>Sharing service engines can help reduce the licensing cost. </p>|
|TKO-ALB-008|Configure anti-affinity rules for the NSX ALB controller cluster.|This is to ensure that no two controllers end up in same ESXi host and thus avoid single point of failure.| Anti-affinity rules need to be created manually.|
|TKO-ALB-009|Configure backup for the NSX ALB Controller cluster.|Backups are required if the NSX ALB Controller becomes inoperable or if the environment needs to be restored from a previous state.|To store backups, a SCP capable backup location is needed. SCP is the only supported protocol currently.|
|TKO-ALB-010|Create an NSX-T Cloud connector on NSX Advanced Load Balancer controller for each NSX transport zone requiring load balancing.|An NSX-T Cloud connector configured on the NSX Advanced Load Balancer controller provides load balancing for workloads belonging to a transport zone on NSX-T.|None |
|TKO-ALB-011|Replace default NSX ALB certificates with Custom CA or Public CA-signed certificates that contains  SAN entries of all Controller nodes |To establish a trusted connection with other infra components, and the default certificate doesn’t include SAN entries which is not acceptable by Tanzu. | None, </br> SAN entries are not applicable if using wild card certificate.<br> | 
|TKO-ALB-012|Create a dedicated resource pool with appropriate reservations for NSX ALB controllers.|Guarantees the CPU and Memory allocation for NSX ALB Controllers and avoids performance degradation in case of resource contention.| None |
|TKO-ALB-013|Configure Remote logging for NSX ALB Controller to send events on Syslog.|For operations teams to centrally monitor NSX ALB and escalate alerts events sent from the NSX ALB Controller.| Additional Operational Overhead.<br> Additional infrastructure Resource.|  
|TKO-ALB-014|Use LDAP/SAML based Authentication for NSX ALB| Helps to Maintain Role based Access Control. | Additional Configuration is required.|
### NSX Advanced Load Balancer Service Engine Recommendations

|**Decision ID**|**Design Decision**|**Design Justification**|**Design Implications**|
| --- | --- | --- | --- |
|TKO-ALB-SE-001|Configure SE Group for Active/Active HA mode.|Provides optimum resiliency, performance, and utilization.|Certain applications might not work in Active/Active mode. For instance, applications that require preserving client IP address. In such cases, use the legacy Active/Standby HA mode.|
|TKO-ALB-SE-002|Configure anti-affinity rule for the SE VMs.|This is ensure that no two SEs in the same SE group end up on same ESXi Host and thus avoid single point of failure.|DRS must be enabled on vSphere cluster where SE VMs are deployed.|
|TKO-ALB-SE-003|Configure CPU and Memory reservation for the SE VMs.|This is to ensure that service engines don’t compete with other VMs during resource contention.|CPU and memory reservation is configured at SE Group level.|
|TKO-ALB-SE-004|Enable 'Dedicated dispatcher CPU' on SE groups that contain the SE VMs of 4 or more vCPUs.<br>Note: This setting must be enabled on SE groups that are servicing applications that have high network requirement.|This enables a dedicated core for packet processing enabling high packet pipeline on the SE VMs.|None.|
|TKO-ALB-SE-005|Create multiple SE groups as desired to isolate applications.|Allows efficient isolation of applications for better capacity planning.<br> Allows flexibility of life-cycle-management.|Multi SE groups will increase the licensing cost.|
|TKO-ALB-SE-006|Create separate service engine groups for TKG management and workload clusters.|Allows isolating the load balancing traffic of management cluster from shared services cluster and workload clusters.|Dedicated service engine groups increase licensing cost.|
|TKO-ALB-SE-007|Set 'Placement across the Service Engines' setting to 'distributed'.|This allows maximum fault tolerance and even utilization of capacity.| None|
|TKO-ALB-SE-008|Set the SE size to a minimum 2vCPU and 4GB of Memory. |This configuration should meet the most generic use case. | For services that require higher throughput, these configurations need to be investigated and modified accordingly. |

### NSX Advanced Load Balancer L7 Ingress Recommendations

|**Decision ID**|**Design Decision**|**Design Justification**|**Design Implications**|
| --- | --- | --- | --- |
|TKO-ALB-L7-001|Deploy NSX ALB L7 Ingress in NodePortLocal mode.| 1. Network hop efficiency is gained by by-passing the kube-proxy to receive external traffic to applications.<br>2. TKG clusters can share SE groups, optimizing or maximizing capacity and license consumption.<br>3. Pod's node port only exists on nodes where the Pod is running, and it helps to reduce the east-west traffic and encapsulation overhead.<br>4. Better session persistence. |1. This is supported only with Antrea CNI.<br>2. NodePortLocal mode is currently only supported for nodes running Linux or Windows with IPv4 addresses. Only TCP and UDP service ports are supported (not SCTP). For more information, see [Antrea NodePortLocal Documentation](https://antrea.io/docs/v1.8.0/docs/node-port-local/).|ß|

VMware recommends using NSX Advanced Load Balancer L7 ingress with the NodePortLocal mode as it gives you a distinct advantage over other modes as mentioned below:

- Although there is a constraint of one SE group per Tanzu Kubernetes Grid cluster, which results in increased license capacity, ClusterIP provides direct communication to the Kubernetes pods, enabling persistence and direct monitoring of individual pods.

- NodePort resolves the issue for needing a SE group per workload cluster, but a kube-proxy is created on each and every workload node even if the pod doesn’t exist in it, and there’s no direct connectivity. Persistence is then broken.

- NodePortLocal is the best of both use cases. Traffic is sent directly to the pods in your workload cluster through node ports without interfering with kube-proxy. SE groups can be shared and load balancing persistence is supported.

### Network Recommendations

The key network recommendations for a production-grade Tanzu Kubernetes Grid deployment with NSX Data Center Networking are as follows:

|**Decision ID**|**Design Decision**|**Design Justification**|**Design Implications**|
| --- | --- | --- | --- |
|TKO-NET-001|Use separate logical segments for management cluster, shared services cluster, workload clusters, and VIP network.|To have a flexible firewall and security policies.|Sharing the same network for multiple clusters can complicate firewall rules creation.|
|TKO-NET-002|Configure DHCP  for each TKG cluster network.|Tanzu Kubernetes Grid does not support static IP address assignments for Kubernetes VM components.|IP address pool can be used for the TKG clusters in absence of the DHCP.|
|TKO-NET-003|Use NSX for configuring DHCP|This avoids setting up dedicated DHCP server for TKG.|For a simpler configuration, make use of the DHCP local server to provide DHCP services for required segments.|
|TKO-NET-004|Create a overlay-backed NSX segment connected to a Tier-1 gateway for the SE management for the NSX-T Cloud of overlay type.|This network is used for the controller to the SE connectivity.|None|
|TKO-NET-005|Create a overlay-backed NSX segment as data network for the NSX-T Cloud of overlay type.|The SEs are placed on overlay Segments created on Tier-1 gateway.|None|

