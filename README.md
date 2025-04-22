# Some samples for CKA preparation for Kubernetes

## Background

This repository contains some sample code and configurations to help you prepare for the Certified Kubernetes Administrator (CKA) exam. The CKA exam tests your knowledge and skills in managing Kubernetes clusters, including installation, configuration, and troubleshooting.

## About the CKA Exam

The CKA exam is a performance-based test that requires you to complete tasks in a live Kubernetes environment. The exam is 2 hours long and consists of multiple-choice questions and hands-on tasks. You will be required to demonstrate your knowledge of Kubernetes concepts, tools, and best practices.

[Learn more about the CKA Exam](https://training.linuxfoundation.org/certification/certified-kubernetes-administrator-cka/)

## Domains & Competencies

The CKA exam is divided into several domains, each with its own set of competencies. The following table outlines the domains and their respective weightings in the exam:

### Storage (10%)

- Implement storage classes and dynamic volume provisioning.
- Configure volume types, access modes, and reclaim policies.
- Manage persistent volumes and persistent volume claims.

### Troubleshooting (30%)

- Troubleshoot clusters and nodes.
- Troubleshoot cluster components.
- Cluster backup and restore.
- Monitor cluster and application resource usage.
- Manage and evaluate container output streams.
- Troubleshoot services and networking.

### Workloads & Scheduling (15%)

- Understand application deployments and how to perform rolling updates and rollbacks.
- Use ConfigMaps and Secrets to configure applications.
- Configure workload autoscaling.
- Understand the primitives used to create robust, self-healing application deployments.
- Configure Pod admission and scheduling (limits, node affinity, etc.).

### Cluster Architecture, Installation & Configuration (25%)

- Manage role-based access control (RBAC).
- Prepare underlying infrastructure for installing a Kubernetes cluster.
- Create and manage Kubernetes clusters using kubeadm.
- Manage the lifecycle of Kubernetes clusters.
- Implement and configure a highly-available control plane.
- Use Helm and Kustomize to install cluster components.
- Understand extension interfaces (CNI, CSI, CRI, etc.).
- Understand CRDs, install and configure operators.

### Services & Networking (20%)

- Understand connectivity between Pods.
- Define and enforce Network Policies.
- Use ClusterIP, NodePort, LoadBalancer service types, and endpoints.
- Use the Gateway API to manage Ingress traffic.
- Know how to use Ingress controllers and Ingress resources.
- Understand and use CoreDNS.

## Useful links

- [Official Kubernetes Documentation](https://kubernetes.io/docs/home/)
- [Candidate Handbook](https://docs.linuxfoundation.org/tc-docs/certification/lf-handbook2)
- [Exam Tips for CKA and CKAD](https://docs.linuxfoundation.org/tc-docs/certification/tips-cka-and-ckad)
- [Curriculum](https://github.com/cncf/curriculum) - here you can find the curriculum for the CKA exam

## Examples

- [01-storage](01-storage/README.md) - All about storage classes and dynamic volume provisioning
- [02-troubleshooting](02-troubleshooting/README.md) - Troubleshooting clusters and nodes
- [03-workloads](03-workloads-and-scheduling/README.md) - Workloads and scheduling
- [04-cluster-architecture](04-cluster-architecture/README.md) - Cluster architecture, installation and configuration
- [05-services](05-services-and-networking/README.md) - Services and networking

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
