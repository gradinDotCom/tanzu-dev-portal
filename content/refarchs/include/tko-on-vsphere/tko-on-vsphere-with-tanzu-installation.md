## Deployment options

Starting with vSphere 8, when you enable vSphere with Tanzu, you can configure either one-zone Supervisor mapped to one vSphere cluster or three-zone Supervisor mapped to three vSphere clusters. 

## Single-Zone Deployment of Supervisor

A supervisor deployed on s single vSphere cluster has three control plane VMs, which reside on the ESXi hosts part of the cluster. A single zone is created for the Supervisor automatically or you can use a zone that is created in advance. In a Single-Zone deployment, cluster-level high availability is maintained through vSphere HA and can scale with vSphere with Tanzu setup by adding physical hosts to vSphere cluster that maps to the Supervisor. 
You can run workloads through vSphere Pods, Tanzu Kubernetes Grid clusters and VMs when Supervisor is enabled with the NSX networking stack.

## Three-Zone Deployment of Supervisor

Configure each vSphere cluster as an independent failure domain and map it to the vSphere zone. In a Three-Zone deployment, all three vSphere clusters become one Supervisor and can
provide :

- Cluster-level high availability to the Supervisor as vSphere cluster is an independent failure domain.
- Distribute the nodes of Tanzu Kubernetes Grid clusters across all three vSphere zones and provide availability via vSphere HA at cluster level.
- Scale the Supervisor by adding hosts to each of the three vSphere clusters.

For more information, see [Supervisor Architecture and Components](https://docs.vmware.com/en/VMware-vSphere/8.0/vsphere-with-tanzu-concepts-planning/GUID-74EC2571-4352-4E15-838E-5F56C8C68D15.html)

## Installation Experience

vSphere with Tanzu deployment starts with deploying the Supervisor cluster (Enabling Workload Management). The deployment is directly done from the vCenter user interface (UI). The Get Started page lists the pre-requisites for the deployment.

![Screenshot of the vCenter integrated Workload Management page](refarchs/tko-on-vsphere-with-tanzu/images/tko-vwt09.jpg)

The vCenter UI shows that, in the current version, it is possible to install vSphere with Tanzu on the VDS networking stack as well as NSX-T Data Center as the networking solution.

![Screenshot of the vCenter UI for configuring vSphere with Tanzu](refarchs/tko-on-vsphere-with-tanzu/images/tko-vwt10.jpg)

This installation process takes you through the steps of deploying **Supervisor Cluster** in your vSphere environment. Once the Supervisor cluster is deployed, you can use either [Tanzu Mission Control](https://tanzu.vmware.com/mission-control) or Kubectl utility to deploy the Tanzu Kubernetes Shared Service and workload clusters.
