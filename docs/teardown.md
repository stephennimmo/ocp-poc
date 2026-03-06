# Teardown

## Delete the POC Cluster

From the hub cluster, delete the cluster deployment and associated resources:

```bash
export KUBECONFIG=~/ocp-installer/sno/auth/kubeconfig
oc delete clusterdeployment ocp-poc -n ocp-poc --wait=true
oc delete namespace ocp-poc --wait=true
```

## Remove ACM from the Hub

```bash
oc delete multiclusterhub multiclusterhub -n open-cluster-management --wait=true
oc delete subscription advanced-cluster-management -n open-cluster-management
oc delete operatorgroup acm-operator-group -n open-cluster-management
oc delete namespace open-cluster-management --wait=true
```

## Decommission the Hub Cluster

Power off and re-image the SNO host as needed.

## Clean Up the Bastion

```bash
rm -rf ~/ocp-installer
sudo rm -f /usr/local/bin/oc /usr/local/bin/kubectl /usr/local/bin/openshift-install
```
