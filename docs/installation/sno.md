# Step 2: Install Single Node OpenShift

The SNO instance acts as the hub cluster. ACM will be installed on this cluster to manage the provisioning of the POC cluster.

## Prepare the Install Configuration

Create the cluster directory:

```bash
SNO_DIR=~/ocp-installer/sno
mkdir -p ${SNO_DIR}
```

Create `${SNO_DIR}/install-config.yaml`:

```yaml
apiVersion: v1
baseDomain: example.com
metadata:
  name: hub
networking:
  networkType: OVNKubernetes
  clusterNetwork:
    - cidr: 10.128.0.0/14
      hostPrefix: 23
  serviceNetwork:
    - 172.30.0.0/16
compute:
  - name: worker
    replicas: 0
controlPlane:
  name: control-plane
  replicas: 1
  architecture: amd64
platform:
  none: {}
pullSecret: '<PULL_SECRET>'
sshKey: '<SSH_PUBLIC_KEY>'
```

- Replace `<PULL_SECRET>` with the contents of `~/.pull-secret`
- Replace `<SSH_PUBLIC_KEY>` with the contents of `~/.ssh/id_ed25519.pub`
- Update `baseDomain` and `metadata.name` to match your environment

## Generate the Agent ISO

Create `${SNO_DIR}/agent-config.yaml` with the SNO host's network details:

```yaml
apiVersion: v1alpha1
kind: AgentConfig
metadata:
  name: hub
rendezvousIP: <SNO_IP>
hosts:
  - hostname: hub
    role: control-plane
    interfaces:
      - name: <INTERFACE_NAME>
        macAddress: <MAC_ADDRESS>
    networkConfig:
      interfaces:
        - name: <INTERFACE_NAME>
          type: ethernet
          state: up
          mac-address: <MAC_ADDRESS>
          ipv4:
            enabled: true
            address:
              - ip: <SNO_IP>
                prefix-length: <PREFIX_LENGTH>
            dhcp: false
      dns-resolver:
        config:
          server:
            - 9.9.9.9
            - 149.112.112.112
      routes:
        config:
          - destination: 0.0.0.0/0
            next-hop-address: <GATEWAY_IP>
            next-hop-interface: <INTERFACE_NAME>
```

Back up the configs and generate the ISO:

```bash
cp ${SNO_DIR}/install-config.yaml ${SNO_DIR}/install-config.yaml.bak
cp ${SNO_DIR}/agent-config.yaml ${SNO_DIR}/agent-config.yaml.bak
openshift-install agent create image --dir ${SNO_DIR}
```

## Boot the SNO Host

Write the ISO to a USB drive or mount it via BMC virtual media, then boot the SNO host from the ISO.

## Monitor the Installation

```bash
openshift-install agent wait-for bootstrap-complete --dir ${SNO_DIR} --log-level=info
openshift-install agent wait-for install-complete --dir ${SNO_DIR} --log-level=info
```

## Verify the Hub Cluster

```bash
export KUBECONFIG=${SNO_DIR}/auth/kubeconfig
oc get nodes
oc get clusteroperators
```
