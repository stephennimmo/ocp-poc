# OpenShift POC

Red Hat OpenShift Container Platform (OCP) 4.20 proof of concept deployment using [Red Hat Advanced Cluster Management](https://www.redhat.com/en/technologies/management/advanced-cluster-management) (ACM) to provision and manage the POC cluster.

A Single Node OpenShift (SNO) instance acts as the hub cluster with ACM installed. ACM uses BMC access to provision the POC cluster on bare metal nodes, mimicking real-world patterns for cluster deployments.

## Architecture

|                    | Count | CPU | Memory | Disk   | Description                            |
| ---                | ---   | --- | ---    | ---    | ---                                    |
| RHEL Bastion       | 1     | 4   | 16 GB  | 40 GB  | Management host, can be VM or physical |
| SNO Hub            | 1     | 16  | 64 GB  | 120 GB | Hub cluster with ACM, VM or physical   |
| Control Plane Node | 3     | 16  | 64 GB  | 120 GB | Bare metal control plane nodes         |
| Worker Node        | 3     | 16  | 64 GB  | 120 GB | Bare metal worker nodes                |

## Getting Started

1. Review the [Prerequisites](prerequisites.md) to ensure your environment is ready.
2. Follow the [Installation](installation/index.md) guide step by step.
3. Reference the [Teardown](teardown.md) guide when you need to clean up.
