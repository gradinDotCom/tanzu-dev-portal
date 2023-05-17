## Design Recommendations

### NSX Advanced Load Balancer Recommendations

The following table provides recommendations for configuring NSX Advanced Load Balancer in a vSphere with Tanzu environment.

|**Decision ID**|**Design Decision**|**Design Justification**|**Design Implications**|
| --- | --- | --- | --- |
|TKO-Advanced Load Balancer-001|Deploy NSX Advanced Load Balancer controller cluster nodes on a network dedicated to NSX-Advanced Load Balancer.|To isolate NSX Advanced Load Balancer traffic from infrastructure management traffic and Kubernetes workloads.|Allows for ease of management of controllers.<br>Additional Network (VLAN) is required.</br>|
|TKO-Advanced Load Balancer-002|Deploy 3 NSX Advanced Load Balancer controllers nodes.|To achieve high availability for the NSX Advanced Load Balancer platform. In clustered mode, NSX Advanced Load Balancer availability is not impacted by an individual controller node failure. The failed node can be removed from the cluster and redeployed if recovery is not possible.|<p>Clustered mode requires more compute and storage resources. </p><p></p>|
|TKO-Advanced Load Balancer-003|Configure vCenter settings in Default-Cloud.|Using a non-default vCenter cloud is not supported with vSphere with Tanzu.|Using a non-default cloud can lead to deployment failures.|
|TKO-Advanced Load Balancer-004|Use static IPs for the NSX Advanced Load Balancer controllers if DHCP cannot guarantee a permanent lease.|NSX Advanced Load Balancer Controller cluster uses management IP addresses to form and maintain quorum for the control plane cluster. Any changes would be disruptive.|NSX Advanced Load Balancer Controller control plane might go down if the management IPs of the controller node changes.|
|TKO-Advanced Load Balancer-005|<p>Use NSX Advanced Load Balancer IPAM for Service Engine data network and virtual services IP assignment. </p><p></p>|Guarantees IP address assignment for Service Engine Data NICs and Virtual Services.|Removes the corner case scenario when the DHCP server runs out of the lease or is down.|
|TKO-Advanced Load Balancer-006|Reserve an IP in the NSX Advanced Load Balancer management subnet to be used as the Cluster IP for the Controller Cluster.|NSX Advanced Load Balancer portal is always accessible over Cluster IP regardless of a specific individual controller node failure.|NSX Advanced Load Balancer administration is not affected by an individual controller node failure.|
|TKO-Advanced Load Balancer-007|Use default Service Engine Group for load balancing of TKG clusters control plane.|Using a non-default Service Engine Group for hosting L4 virtual service created for TKG control plane HA is not supported.|Using a non-default Service Engine Group can lead to Service Engine VM deployment failure.|
|TKO-Advanced Load Balancer-008|Share Service Engines for the same type of workload (dev/test/prod)clusters.|Minimize the licensing cost.|<p>Each Service Engine contributes to the CPU core capacity associated with a license.</p><p></p><p>Sharing Service Engines can help reduce the licensing cost. </p>|
|TKO-Advanced Load Balancer-009|Configure anti-affinity rules for the NSX ALB controller cluster.|This is to ensure that no two controllers end up in same ESXi host and thus avoid single point of failure.|Anti-Affinity rules need to be created manually.|
|TKO-Advanced Load Balancer-0010|Configure backup for the NSX ALB Controller cluster.|Backups are required if the NSX ALB Controller becomes inoperable or if the environment needs to be restored from a previous state.|To store backups, a SCP capable backup location is needed. SCP is the only supported protocol currently.|
|TKO-Advanced Load Balancer-0011|Initial setup should be done only on one NSX ALB controller VM out of the three deployed to create an NSX ALB controller cluster.|NSX ALB controller cluster is created from an initialized NSX ALB controller which becomes the cluster leader.<br> Follower NSX ALB controller nodes need to be uninitialized to join the cluster.|NSX ALB controller cluster creation fails if more than one NSX ALB controller is initialized.|
|TKO-Advanced Load Balancer-0012|Configure Remote logging for NSX ALB Controller to send events on Syslog.|For operations teams to centrally monitor NSX ALB and escalate alerts events must be sent from the NSX ALB Controller|  Additional Operational Overhead.<br> Additional infrastructure Resource. |  
|TKO-Advanced Load Balancer-0013|Use LDAP/SAML based Authentication for NSX ALB| Helps to Maintain Role based Access Control. | Additional Configuration is required. |

### Network Recommendations

The following are the key network recommendations for a production-grade vSphere with Tanzu deployment:

|**Decision ID**|**Design Decision**|**Design Justification**|**Design Implications**|
| --- | --- | --- | --- |
|TKO-NET-001|Use separate networks for Supervisor cluster and workload clusters.|To have a flexible firewall and security policies|Sharing the same network for multiple clusters can complicate creation of firewall rules. |
|TKO-NET-002|Use distinct port groups for network separation of K8s workloads.|Isolate production Kubernetes clusters from dev/test clusters by placing them on distinct port groups.|Network mapping is done at the namespace level.  All Kubernetes clusters created in a namespace connect to the same port group. |
|TKO-NET-003|Use routable networks for Tanzu Kubernetes clusters.|Allow connectivity between the TKG clusters and infrastructure components.|Networks that are used for Tanzu Kubernetes cluster traffic must be routable between each other and the Supervisor Cluster Management Network.|

### Recommendations for Supervisor Clusters

|**Decision ID**|**Design Decision**|**Design Justification**|**Design Implications**|
| --- | --- | --- | --- |
|TKO-TKGS-001|Create a Subscribed Content Library. |<p>Subscribed Content Library can automatically pull the latest OVAs used by the Tanzu Kubernetes Grid Service to build cluster nodes.</p><p>Using a subscribed content library facilitates template management as new versions can be pulled by initiating the library sync.</p>|<p>Local Content Library would require manual upload of images, suitable for air-gapped or Internet restricted environment.</p>|
|TKO-TKGS-002|Deploy Supervisor cluster control plane nodes in large form factor.|Large form factor should suffice to integrate Supervisor Cluster with TMC and velero deployment.|Consume more Resources from Infrastructure.|
|TKO-TKGS-003|Register Supervisor cluster with Tanzu Mission Control.|Tanzu Mission Control automates the creation of the Tanzu Kubernetes clusters and manage the life cycle of all clusters centrally.|Need outbound connectivity to internet for TMC registration.|

**Note:** SaaS endpoints here refers to Tanzu Mission Control, Tanzu Service Mesh and Tanzu Observability. 

### Recommendations for Tanzu Kubernetes Clusters

|**Decision ID**|**Design Decision**|**Design Justification**|**Design Implications**|
| --- | --- | --- | --- |
|TKO-TKC-001|Deploy Tanzu Kubernetes clusters with prod plan and multiple worker nodes.|The prod plan provides high availability for the control plane. |Consume from resource from Infrastructure.|
|TKO-TKC-002|Use guaranteed VM class for Tanzu Kubernetes clusters.|Guarantees compute resources are always available for containerized workloads.|Could prevent automatic migration of nodes by DRS.|
|TKO-TKC-003|Implement RBAC for Tanzu Kubernetes clusters.|To avoid the usage of administrator credentials for managing the clusters.|External AD/LDAP needs to be integrated with vCenter or SSO groups need to be created manually.
|TKO-TKC-04|Deploy Tanzu Kubernetes clusters from Tanzu Mission Control.|Tanzu Mission Control provides life-cycle management for the Tanzu Kubernetes clusters and automatic integration with Tanzu Service Mesh and Tanzu Observability.|Only Antrea CNI is supported on Workload clusters created from TMC portal.|

## Kubernetes Ingress Routing
vSphere with Tanzu does not ship with a default ingress controller. Any Tanzu-supported ingress controller can be used.

One example of an ingress controller is Contour, an open-source controller for Kubernetes ingress routing. Contour is part of a Tanzu package and can be installed on any Tanzu Kubernetes cluster. Deploying Contour is a prerequisite for deploying Prometheus, Grafana, and Harbor on a workload cluster.

For more information about Contour, see the [Contour](https://projectcontour.io/) site and [Implementing Ingress Control with Contour](https://docs.vmware.com/en/VMware-Tanzu-Kubernetes-Grid/2.1/using-tkg-21/workload-packages-contour.html).

[Tanzu Service Mesh](https://tanzu.vmware.com/service-mesh) also offers an Ingress controller based on [Istio](https://istio.io/).

Each ingress controller has pros and cons of its own. The below table provides general recommendations on when you should use a specific ingress controller for your Kubernetes environment.

|**Ingress Controller**|**Use Cases**|
| --- | --- |
|Contour|<p>Use Contour when only north-south traffic is needed in a Kubernetes cluster. You can apply security policies for the north-south traffic by defining the policies in the manifest file for the application.</p><p></p><p>Contour is a reliable solution for simple Kubernetes workloads. </p>|
|Istio|Use Istio ingress controller when you need to provide security, traffic direction, and insight within the cluster (east-west traffic) and between the cluster and the outside world (north-south traffic).|

## NSX Advanced Load Balancer Sizing Guidelines

### NSX Advanced Load Balancer Controller Configuration

Regardless of NSX Advanced Load Balancer Controller configuration, each controller cluster can achieve up to 5,000 virtual services; 5,000 is a hard limit. For further details, see [Avi Controller Sizing](https://avinetworks.com/docs/22.1/avi-controller-sizing/).

| Controller Size | VM Configuration    | Virtual Services | Avi SE Scale |
| --------------- | ------------------- | ---------------- | ------------ |
| Essentials      | 4 vCPUS, 24 GB RAM  | 0-50             | 0-10         |
| Small           | 6 vCPUS, 24 GB RAM  | 0-200            | 0-100        |
| Medium          | 10 vCPUS, 32 GB RAM | 200-1000         | 100-200      |
| Large           | 16 vCPUS, 48 GB RAM | 1000-5000        | 200-400      |


### Service Engine Sizing Guidelines

See [Sizing Service Engines](https://avinetworks.com/docs/22.1/sizing-service-engines/) for guidance on sizing your SEs.

| Performance metric | 1 vCPU core |
| ------------------ | ----------- |
| Throughput         | 4 Gb/s      |
| Connections/s      | 40k         |
| SSL Throughput     | 1 Gb/s      |
| SSL TPS (RSA2K)    | ~600        |
| SSL TPS (ECC)      | 2500        |

Multiple performance vectors or features may have an impact on performance.  For example, to achieve 1 Gb/s of SSL throughput and 2000 TPS of SSL with EC certificates, NSX Advanced Load Balancer recommends two cores.

NSX Advanced Load Balancer Service Engines may be configured with as little as 1 vCPU core and 2 GB RAM, or up to 64 vCPU cores and 256 GB RAM. It is recommended for a Service Engine to have at least 4 GB of memory when GeoDB is in use.

## Container Registry

VMware Tanzu for Kubernetes Operations using vSphere with Tanzu includes Harbor as a container registry. Harbor provides a location for pushing, pulling, storing, and scanning container images used in your Kubernetes clusters.

The initial configuration and setup of the platform does not require any external registry because the required images are delivered through vCenter. Harbor registry is used for day 2 operations of the Tanzu Kubernetes workload clusters. Typical day-2 operations include tasks such as pulling images from Harbor for application deployment and pushing custom images to Harbor.

When vSphere with Tanzu is deployed on VDS networking, you can deploy an external container registry (Harbor) for Tanzu Kubernetes clusters.

You may use one of the following methods to install Harbor:

- [**Tanzu Kubernetes Grid Package deployment**](https://docs.vmware.com/en/VMware-Tanzu-Kubernetes-Grid/2.1/using-tkg-21/workload-packages-harbor.html) to a Tanzu Kubernetes Grid cluster - VMware recommends this installation method for general use cases. The Tanzu packages, including Harbor, must either be pulled directly from VMware or be hosted in an internal registry.
- [**VM-based deployment**](https://goharbor.io/docs/latest/install-config/installation-prereqs/) using `docker-compose` - VMware recommends using this installation method in cases where Tanzu Kubernetes Grid is being installed in an air-gapped or Internet-less environment and no pre-existing image registry exists to host the Tanzu Kubernetes Grid system images. VM-based deployments are only supported by VMware Global Support Services to host the system images for air-gapped or Internet-less deployments. Do not use this method for hosting application images.
- [**Helm-based deployment**](https://goharbor.io/docs/latest/install-config/harbor-ha-helm/) to a Kubernetes cluster - This installation method may be preferred for customers already invested in Helm. Helm deployments of Harbor are only supported by the Open Source community and not by VMware Global Support Services.

If you are deploying Harbor without a publicly signed certificate, you must include the Harbor root CA in your Tanzu Kubernetes Grid clusters. To do so, follow the procedure in [Trust Custom CA Certificates on Cluster Nodes](https://docs.vmware.com/en/VMware-Tanzu-Kubernetes-Grid/2.1/using-tkg-21/workload-clusters-secret.html).

  For VM-based deployments, the base VM for Harbor can be provisioned using the [VMOperator](https://github.com/vmware-tanzu/vm-operator).

![Screenshot of Harbor Registry UI](refarchs/tko-on-vsphere-with-tanzu/images/tko-vwt12.png)

## vSphere with Tanzu SaaS Integration

The SaaS products in the VMware Tanzu portfolio are on the critical path for securing systems at the heart of your IT infrastructure. VMware Tanzu Mission Control provides a centralized control plane for Kubernetes, and Tanzu Service Mesh provides a global control plane for service mesh networks. Tanzu Observability features include Kubernetes monitoring, application observability, and service insights.

To learn more about Tanzu Kubernetes Grid integration with Tanzu SaaS, see [Tanzu SaaS Services](tko-saas.md).

### Custom Tanzu Observability Dashboards

Tanzu Observability provides various out-of-the-box dashboards. You can customize the dashboards for your particular deployment. For information on how to customize Tanzu Observability dashboards for Tanzu for Kubernetes Operations, see [Customize Tanzu Observability Dashboard for Tanzu for Kubernetes Operations](../deployment-guides/tko-to-customized-dashboard.md).

## Summary

vSphere with Tanzu on hyper-converged hardware offers high-performance potential and convenience and addresses the challenges of creating, testing, and updating on-premises Kubernetes platforms in a consolidated production environment. This validated approach results in a production installation with all the application services needed to serve combined or uniquely separated workload types via a combined infrastructure solution.

This plan meets many Day-0 needs for quickly aligning product capabilities to full-stack infrastructure, including networking, configuring firewall rules, load balancing, workload compute alignment, and other capabilities.

## Deployment Instructions

For instructions on how to deploy this reference design, see [Deploy Tanzu for Kubernetes Operations using vSphere with Tanzu](../deployment-guides/tko-on-vsphere-with-tanzu.md).