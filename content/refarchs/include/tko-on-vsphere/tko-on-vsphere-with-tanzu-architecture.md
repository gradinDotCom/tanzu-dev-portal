## vSphere with Tanzu Architecture

The following diagram shows the high-level architecture of vSphere with Tanzu.

![Diagram of vSphere with Tanzu Architecture](refarchs/tko-on-vsphere-with-tanzu/images/tko-vwt03.png)

The Supervisor Cluster consists of the following:

*   **Kubernetes control plane VM:** Three Kubernetes control plane VMs in total are created on the hosts that are part of the Supervisor Cluster. The three control plane VMs are load-balanced as each one of them has its own IP address.

*   **Cluster API and Tanzu Kubernetes Grid Service:** These are modules that run on the Supervisor Cluster and enable the provisioning and management of Tanzu Kubernetes clusters.

The following diagram shows the general architecture of the Supervisor Cluster.

![Diagram of Supervisor Cluster Architecture](refarchs/tko-on-vsphere-with-tanzu/images/tko-vwt04.jpg)

After a Supervisor Cluster is created, the vSphere administrator creates vSphere namespaces. When initially created, vSphere namespaces have unlimited resources within the Supervisor Cluster. The vSphere administrator defines the limits for CPU, memory, and storage, as well as the number of Kubernetes objects such as deployments, replica sets, persistent volumes, etc. that can run within the namespace. These limits are configured for each vSphere namespace.

For the maximum supported number, see the **vSphere with Tanzu [Configuration Maximums](https://configmax.esp.vmware.com/guest?vmwareproduct=vSphere&release=vSphere%208.0&categories=1-0,70-58,71-0)** guide.

![vSphere Namespace](refarchs/tko-on-vsphere-with-tanzu/images/tko-vwt05.png)

To provide tenants access to namespaces, the vSphere administrator assigns permission to users or groups available within an identity source that is associated with vCenter Single Sign-On.

Once the permissions are assigned, tenants can access the namespace to create Tanzu Kubernetes Clusters using YAML files and the Cluster API.

Here are some recommendations for using namespaces in a vSphere with Tanzu environment.

|**Decision ID**|**Design Decision**|**Design Justification**|**Design Implications**|
| --- | --- | --- | --- |
|TKO-TKGS-003|Create namespaces to logically separate K8s workloads.|Create dedicated namespaces for the type of workloads (prod/dev/test) that you intend to run. |All Kubernetes clusters created under a namespace share the same access policy/quotas/network resources. |
|TKO-TKGS-004|Enable self-service namespaces.|Enable DevOps/Cluster admin users to provision namespaces in a self-service manner.|The vSphere administrator must publish a namespace template to the LDAP users/groups to enable them to create namespaces.|
|TKO-TKGS-005|Register external identity source (AD/LDAP) with vCenter.|Limit access to a namespace to authorized users/groups.|A prod namespace can be accessed by a handful of users, whereas a dev/test namespace can be exposed to a wider audience.|
