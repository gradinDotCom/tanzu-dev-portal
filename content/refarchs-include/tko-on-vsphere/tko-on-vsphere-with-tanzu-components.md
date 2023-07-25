## vSphere with Tanzu Components

- **Supervisor Cluster:** When **Workload Management** is enabled on a vSphere cluster, it creates a Kubernetes layer within the ESXi hosts that are part of the cluster. A cluster that is enabled for **Workload Management** is called a Supervisor Cluster. You run containerized workloads by creating upstream Kubernetes clusters on the Supervisor Cluster through the Tanzu Kubernetes Grid Service.

  The Supervisor Cluster runs on top of an SDDC layer that consists of ESXi for compute, vSphere Distributed Switch for networking, and vSAN or another shared storage solution.

- **vSphere Namespaces:** A vSphere Namespace is a tenancy boundary within vSphere with Tanzu. A vSphere Namespace allows for sharing vSphere resources (computer, networking, storage) and enforcing resource limits with the underlying objects such as Tanzu Kubernetes clusters. For each namespace, you configure role-based access control ( policies and permissions ), images library amd virtual machine classes.

- **Tanzu Kubernetes Grid Service:** Tanzu Kubernetes Grid Service (TKGS) allows you to create and manage ubiquitous Kubernetes clusters on a VMware vSphere infrastructure using the Kubernetes Cluster API. The Cluster API provides declarative, Kubernetes-style APIs for the creation, configuration, and management of the Tanzu Kubernetes Cluster.

	Tanzu Kubernetes Grid Service also provides self-service lifecycle management of Tanzu Kubernetes clusters.

- **Tanzu Kubernetes Cluster (Workload Cluster):** Tanzu Kubernetes clusters are Kubernetes workload clusters in which your application workloads run. These clusters can be attached to SaaS solutions such as Tanzu Mission Control, Tanzu Observability, and Tanzu Service Mesh, which are part of Tanzu for Kubernetes Operations.

- **VM Class in vSphere with Tanzu:** A VM class is a template that defines CPU, memory, and reservations for VMs. VM classes are used for VM deployment in a Supervisor Namespace. VM classes can be used by standalone VMs that run in a Supervisor Namespace and by VMs hosting a Tanzu Kubernetes cluster.

  VM classes in vSphere with Tanzu are broadly categorized into two groups.

     - **guaranteed:** The guaranteed class fully reserves its configured resources.
     - **best-effort:** The best-effort class allows resources to be overcommitted.

	vSphere with Tanzu offers several default VM classes. You can use them as is or you can create new VM classes. The following screenshot shows the default VM classes that are available in vSphere with Tanzu.

	![Screenshot of default VM Classes in vSphere with Tanzu](refarchs/tko-on-vsphere-with-tanzu/images/tko-vwt01.jpg)

	![Screenshot of default VM Classes in vSphere with Tanzu (cont.)](refarchs/tko-on-vsphere-with-tanzu/images/tko-vwt02.jpg)

- **Storage Classes in vSphere with Tanzu:** A StorageClass provides a way for administrators to describe the classes of storage they offer. Different classes can map to quality-of-service levels, to backup policies, or to arbitrary policies determined by the cluster administrators.

	You can deploy vSphere with Tanzu with an existing default StorageClass or the vSphere Administrator can define StorageClass objects (Storage policy) that let cluster users dynamically create PVC and PV objects with different storage types and rules.

The following table provides recommendations for configuring VM Classes/Storage Classes in a vSphere with Tanzu environment.

|**Decision ID**|**Design Decision**|**Design Justification**|**Design Implications**|
| --- | --- | --- | --- |
|TKO-TKGS-001|Create custom Storage Classes/Profiles/Policies|To provide different levels of QoS and SLA for prod and dev/test K8s workloads. <br>To isolate Supervisor clusters from workload clusters.</br>|Default Storage Policy might not be adequate if deployed applications have different performance and availability requirements. |
|TKO-TKGS-002|Create custom VM Classes|To facilitate deployment of K8s workloads with specific compute/storage requirements.|Default VM Classes in vSphere with Tanzu are not adequate to run a wide variety of K8s workloads. |
