# Prerequisites

## Preferred Option

The preferred option for executing a POC for Red Hat OpenShift Container Platform (OCP) is to be able to demonstrate the capabilities of [Red Hat Advanced Cluster Management](https://www.redhat.com/en/technologies/management/advanced-cluster-management)(ACM) to be able to provision a new Red Hat OpenShift cluster. This preferred setup for this would be to run a single node instance (SNO) of OCP to act as a "hub" cluster, install ACM on the hub cluster, provide BMC access to ACM and then allow ACM to provision and manage the POC cluster. There are other options for installation, but this configuration mimics real-world patterns for cluster deployments and thus will provide the best environment for determining technical success. 

## What resources are needed?

|                       | Count | CPU | Memory | Disk   | Description                                 |
| ---                   | ---   | --- | ---    | ---    | ---                                         |
| RHEL Bastion Host     | 1     | 4   | 16 GB  | 40  GB | This can be a VM or a bare metal host *     |
| SNO Host              | 1     | 16  | 64 GB  | 120 GB | This can be a VM or a bare metal host *     |
| Control Plane Host    | 3     | 16  | 64 GB  | 120 GB | Bare metal control plane nodes              |
| Worker Host           | 3     | 16  | 64 GB  | 120 GB | Bare metal worker nodes                     |

\* Technically, the bastion host could also host the SNO instance using KVM

> The CPU and Memory and Disk are bare minimum for install and basic POC work. As with anything, the more resources, the better. 

### What else is needed?

* DNS - You will need to create multiple new DNS entries for the cluster API and ingress. 
    * The ingress is a wildcard DNS. 
* Storage provider with CSI driver
* A [Red Hat account](https://www.redhat.com/wapps/ugc/register.html) associated with your organization. This is needed for the cluster evaluation subscriptions. 
    * RHEL 9.x ISO (download from [Red Hat Developer](https://developers.redhat.com/products/rhel/download)
    * Red Hat pull-secret (download from [console.redhat.com](https://console.redhat.com/openshift/install/pull-secret))
* An SSH key pair
* Control plane and bare metal worker nodes with BMC connectivity
    * BMC credentials and IPs for all control plane and worker node hosts
* Network connectivity between the bastion, sno host and control plane/worker nodes on the same L2 segment
* A machine subnet with at least a `/28` available. See table below. 

## Network Layout

The install requires a machine network subnet. For a POC, a `/28` is sufficient.

Here's an example network address layout for a `10.0.0.0/28` subnet.

| Host            | IP Address | MAC Address       |
| ---             | ---        | ---               |
| Gateway         | 10.0.0.1   |                   |
| bastion         | 10.0.0.2   | (your MAC address) |
| sno             | 10.0.0.3   | (your MAC address) |
| API VIP         | 10.0.0.7   |                   |
| Ingress VIP     | 10.0.0.8   |                   |
| control-plane-0 | 10.0.0.9   | (your MAC address) |
| control-plane-1 | 10.0.0.10  | (your MAC address) |
| control-plane-2 | 10.0.0.11  | (your MAC address) |
| worker-0        | 10.0.0.12  | (your MAC address) |
| worker-1        | 10.0.0.13  | (your MAC address) |
| worker-2        | 10.0.0.14  | (your MAC address) |

## Are you ready?

- [ ] Have you logged into all the hosts using BMC?
- [ ] Does your DNS records resolve? 
    - `dig api.my-cluster.ocp.example.com +short`
    - `dig random-app.apps.my-cluster.ocp.example.com +short`