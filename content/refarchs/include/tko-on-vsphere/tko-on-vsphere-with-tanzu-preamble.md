# VMware Tanzu for Kubernetes Operations using vSphere with Tanzu Reference Design

vSphere with Tanzu transforms the vSphere cluster into a platform for running Kubernetes workloads in dedicated resource pools. When vSphere with Tanzu is enabled on a vSphere cluster, vSphere with Tanzu creates a Kubernetes control plane directly in the hypervisor layer. You can then run Kubernetes containers by creating upstream Kubernetes clusters through the VMware Tanzu Kubernetes Grid Service and run your applications inside these clusters.

This document provides a reference design for deploying VMware Tanzu for Kubernetes Operations on vSphere with Tanzu.

The following reference design is based on the architecture and components described in [VMware Tanzu for Kubernetes Operations Reference Architecture](index.md).

![Diagram of TKO using vSphere with Tanzu Reference Architecture](refarchs/tko-on-vsphere-with-tanzu/images/TKO-VWT-Reference-Design.jpg)
