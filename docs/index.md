# OpenShift POC

[Red Hat OpenShift Container Platform](https://www.redhat.com/en/technologies/cloud-computing/openshift) (OCP) proof of concept (POC) deployment using [Red Hat Advanced Cluster Management](https://www.redhat.com/en/technologies/management/advanced-cluster-management) (ACM) to provision and manage the POC cluster. A Single Node OpenShift (SNO) instance acts as the hub cluster with ACM installed. ACM uses BMC access to provision the POC cluster on bare metal nodes, mimicking real-world patterns for cluster deployments. This install would support both container and virtual machine based workloads. 

## Getting Started

1. Review the [Prerequisites](prerequisites.md) to ensure your environment is ready.
2. Follow the [Installation](installation/index.md) guide step by step.
3. Reference the [Teardown](teardown.md) guide when you need to clean up.

## Tips for Success

* Make sure all the prerequisites are completed prior to POC kickoff and install. 
* Have definitive functional use cases to determine success of the POC. 
* Don't try to performance test in a POC environment with no customizations or specialized configuration. If you need performance metrics and references, Red Hat has plenty of independently verified performance metrics. 
* Running a POC is as much about learning as it is proving out the technology. Use the opportunity to spend quality time with Red Hat sales engineers who can be a wealth of information for running successful clusters in production. 
