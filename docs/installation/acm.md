# Step 3: Install Advanced Cluster Management

Install Red Hat Advanced Cluster Management (ACM) on the hub cluster to manage the provisioning of the POC cluster.

## Set the Kubeconfig

```bash
export KUBECONFIG=~/ocp-installer/sno/auth/kubeconfig
```

## Install the ACM Operator

Create the namespace:

```bash
oc create namespace open-cluster-management
```

Create the OperatorGroup:

```bash
cat <<EOF | oc apply -f -
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
  name: acm-operator-group
  namespace: open-cluster-management
spec:
  targetNamespaces:
    - open-cluster-management
EOF
```

Create the Subscription:

```bash
cat <<EOF | oc apply -f -
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: advanced-cluster-management
  namespace: open-cluster-management
spec:
  channel: release-2.13
  installPlanApproval: Automatic
  name: advanced-cluster-management
  source: redhat-operators
  sourceNamespace: openshift-marketplace
EOF
```

Wait for the operator to be ready:

```bash
oc wait --for=condition=Available deployment/multiclusterhub-operator \
  -n open-cluster-management --timeout=300s
```

## Create the MultiClusterHub

```bash
cat <<EOF | oc apply -f -
apiVersion: operator.open-cluster-management.io/v1
kind: MultiClusterHub
metadata:
  name: multiclusterhub
  namespace: open-cluster-management
spec: {}
EOF
```

Wait for ACM to be fully deployed:

```bash
oc wait --for=condition=Complete multiclusterhub/multiclusterhub \
  -n open-cluster-management --timeout=600s
```

## Verify ACM

```bash
oc get multiclusterhub -n open-cluster-management
```

The ACM console is available at `https://multicloud-console.apps.hub.<baseDomain>`.
