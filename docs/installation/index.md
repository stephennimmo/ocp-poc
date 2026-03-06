# Installation Overview

The installation uses a hub-and-spoke model. A Single Node OpenShift (SNO) instance serves as the hub cluster with Red Hat Advanced Cluster Management (ACM) installed. ACM provisions the POC cluster on bare metal nodes via BMC access.

## Steps

| Step | Description                                                           |
| ---  | ---                                                                   |
| 1    | [Prepare the Bastion Host](bastion.md)                               |
| 2    | [Install Single Node OpenShift](sno.md)                              |
| 3    | [Install Advanced Cluster Management](acm.md)                       |
| 4    | [Provision the POC Cluster](provision.md)                            |
| 5    | [Access the Cluster](access.md)                                      |
