# Step 4: Provision the POC Cluster

Use ACM to provision the POC cluster on bare metal nodes via BMC access.

## Set the Kubeconfig

```bash
export KUBECONFIG=~/ocp-installer/sno/auth/kubeconfig
```

## Create the Cluster Namespace

```bash
oc create namespace ocp-poc
```

## Create the Pull Secret

```bash
oc create secret generic pull-secret \
  -n ocp-poc \
  --from-file=.dockerconfigjson=${HOME}/.pull-secret \
  --type=kubernetes.io/dockerconfigjson
```

## Create the BMC Credentials

Create a secret for each bare metal host's BMC credentials:

```bash
for host in control-plane-0 control-plane-1 control-plane-2 worker-0 worker-1 worker-2; do
  oc create secret generic ${host}-bmc-secret \
    -n ocp-poc \
    --from-literal=username=<BMC_USERNAME> \
    --from-literal=password=<BMC_PASSWORD>
done
```

!!! note
    Update `<BMC_USERNAME>` and `<BMC_PASSWORD>` with the actual BMC credentials for each host. If hosts have different credentials, create each secret individually.

## Register the Bare Metal Hosts

Create a `BareMetalHost` resource for each node. Example for `control-plane-0`:

```yaml
apiVersion: metal3.io/v1alpha1
kind: BareMetalHost
metadata:
  name: control-plane-0
  namespace: ocp-poc
  labels:
    infraenvs.agent-install.openshift.io: ocp-poc
  annotations:
    bmac.agent-install.openshift.io/hostname: control-plane-0
    bmac.agent-install.openshift.io/role: control-plane
spec:
  online: true
  bootMACAddress: <MAC_ADDRESS>
  bmc:
    address: idrac-virtualmedia://<BMC_IP>/redfish/v1/Systems/System.Embedded.1
    credentialsName: control-plane-0-bmc-secret
    disableCertificateVerification: true
```

!!! note
    The `bmc.address` format depends on your hardware. Common formats include `idrac-virtualmedia://`, `ilo5-virtualmedia://`, and `redfish-virtualmedia://`.

Repeat for all control plane and worker nodes, updating the name, MAC address, BMC IP, credentials secret, and role (`control-plane` or `worker`) accordingly.

## Create the AgentClusterInstall

```bash
cat <<EOF | oc apply -f -
apiVersion: extensions.hive.openshift.io/v1beta1
kind: AgentClusterInstall
metadata:
  name: ocp-poc
  namespace: ocp-poc
spec:
  clusterDeploymentRef:
    name: ocp-poc
  imageSetRef:
    name: openshift-4.20
  networking:
    clusterNetwork:
      - cidr: 10.128.0.0/14
        hostPrefix: 23
    serviceNetwork:
      - 172.30.0.0/16
    machineNetwork:
      - cidr: 10.0.0.0/28
  provisionRequirements:
    controlPlaneAgents: 3
    workerAgents: 3
  apiVIPs:
    - 10.0.0.2
  ingressVIPs:
    - 10.0.0.3
  sshPublicKey: '<SSH_PUBLIC_KEY>'
EOF
```

## Create the ClusterDeployment

```bash
cat <<EOF | oc apply -f -
apiVersion: hive.openshift.io/v1
kind: ClusterDeployment
metadata:
  name: ocp-poc
  namespace: ocp-poc
spec:
  baseDomain: example.com
  clusterInstallRef:
    group: extensions.hive.openshift.io
    kind: AgentClusterInstall
    name: ocp-poc
    version: v1beta1
  clusterName: ocp-poc
  platform:
    agentBareMetal:
      agentSelector:
        matchLabels:
          infraenvs.agent-install.openshift.io: ocp-poc
  pullSecretRef:
    name: pull-secret
EOF
```

## Create the InfraEnv

```bash
cat <<EOF | oc apply -f -
apiVersion: agent-install.openshift.io/v1beta1
kind: InfraEnv
metadata:
  name: ocp-poc
  namespace: ocp-poc
spec:
  clusterRef:
    name: ocp-poc
    namespace: ocp-poc
  pullSecretRef:
    name: pull-secret
  sshAuthorizedKey: '<SSH_PUBLIC_KEY>'
  nmStateConfigLabelSelector:
    matchLabels:
      infraenvs.agent-install.openshift.io: ocp-poc
EOF
```

## Monitor the Provisioning

Watch the agents register:

```bash
oc get agents -n ocp-poc -w
```

Monitor the cluster installation progress:

```bash
oc get agentclusterinstall ocp-poc -n ocp-poc -o jsonpath='{.status.conditions}' | jq .
```

Wait for the cluster deployment to complete:

```bash
oc wait --for=condition=Installed clusterdeployment/ocp-poc \
  -n ocp-poc --timeout=3600s
```
