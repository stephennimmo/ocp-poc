# Step 5: Access the Cluster

## Retrieve the Kubeconfig

Extract the POC cluster's kubeconfig from the hub cluster:

```bash
oc get secret -n ocp-poc ocp-poc-admin-kubeconfig -o jsonpath='{.data.kubeconfig}' | base64 -d > ~/ocp-installer/ocp-poc-kubeconfig
```

## Retrieve the Kubeadmin Password

```bash
oc get secret -n ocp-poc ocp-poc-admin-password -o jsonpath='{.data.password}' | base64 -d; echo
```

## Verify the POC Cluster

```bash
export KUBECONFIG=~/ocp-installer/ocp-poc-kubeconfig
oc get nodes
oc get clusteroperators
```

The web console is available at `https://console-openshift-console.apps.ocp-poc.example.com`.
