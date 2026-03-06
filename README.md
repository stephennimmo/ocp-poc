# ocp-poc

OpenShift 4.20 proof of concept deployment using the agent-based installer. The 3 control plane nodes run as KVM virtual machines on a Red Hat Enterprise Linux host. The 3 worker nodes are bare metal machines.

## Prerequisite Hardware

| Machine       | Count | CPU | Memory | Disk   | Description                                 |
| ------------- | ----- | --- | ------ | ------ | -----------------------------------------   |
| KVM Host      | 1     | 28  | 64 GB  | 500 GB | RHEL KVM host for install and control plane |
| Worker Node   | 3     | 16  | 64 GB  | 120 GB | Bare metal worker nodes                     |

## Prerequisites

- RHEL 9.x host with hardware virtualization support (Intel VT-x / AMD-V) (download from [Red Hat Developer](https://developers.redhat.com/products/rhel/download))
- Red Hat pull secret at `~/.pull-secret` (download from [console.redhat.com](https://console.redhat.com/openshift/install/pull-secret))
- SSH key pair at `~/.ssh/id_rsa` and `~/.ssh/id_rsa.pub`
- 3 bare metal worker nodes with iLo/iDRAC/IMM/PXE or USB boot capability
- Network connectivity between the KVM host and worker nodes on the same L2 segment

### Network Layout

The install requires a machine network subnet. For a POC, a `/28` is sufficient. 

Here's an example network address layout for a `10.0.0.0/28` subnet.

| Host        | IP Address | MAC Address        |
| ----------- | ---------- | ------------------ |
| Gateway     | 10.0.0.1   |                    |
| API VIP     | 10.0.0.2   |                    |
| Ingress VIP | 10.0.0.3   |                    |
| master-0    | 10.0.0.4   | 52:54:00:aa:bb:00  |
| master-1    | 10.0.0.5   | 52:54:00:aa:bb:01  |
| master-2    | 10.0.0.6   | 52:54:00:aa:bb:02  |
| worker-0    | 10.0.0.7   | (your MAC address) |
| worker-1    | 10.0.0.8   | (your MAC address) |
| worker-2    | 10.0.0.9   | (your MAC address) |


## Step 1: Prepare the KVM Host

Install virtualization packages:

```bash
sudo dnf install -y qemu-kvm libvirt libvirt-client virt-install virt-viewer libguestfs-tools cockpit cockpit-machines
```

Enable and start services:

```bash
sudo systemctl enable --now libvirtd
sudo systemctl enable --now cockpit.socket
```

Open firewall ports:

```bash
sudo firewall-cmd --permanent --add-service=cockpit
sudo firewall-cmd --permanent --add-service=libvirt
sudo firewall-cmd --reload
```

Verify KVM is working:

```bash
sudo virt-host-validate qemu
```

## Step 2: Create the Libvirt Network

Create a network definition file at `/tmp/ocp-poc-network.xml`:

```xml
<network>
  <name>ocp-poc</name>
  <forward mode='nat'>
    <nat>
      <port start='1024' end='65535'/>
    </nat>
  </forward>
  <bridge name='virbr-ocp' stp='on' delay='0'/>
  <ip address='10.0.0.1' netmask='255.255.255.240'>
    <dhcp>
      <host mac='52:54:00:aa:bb:00' name='master-0.ocp-poc.example.com' ip='10.0.0.4'/>
      <host mac='52:54:00:aa:bb:01' name='master-1.ocp-poc.example.com' ip='10.0.0.5'/>
      <host mac='52:54:00:aa:bb:02' name='master-2.ocp-poc.example.com' ip='10.0.0.6'/>
    </dhcp>
  </ip>
  <dns>
    <forwarder addr='9.9.9.9'/>
    <forwarder addr='149.112.112.112'/>
    <host ip='10.0.0.2'>
      <hostname>api.ocp-poc.example.com</hostname>
    </host>
    <host ip='10.0.0.3'>
      <hostname>*.apps.ocp-poc.example.com</hostname>
    </host>
  </dns>
</network>
```

Define, start, and autostart the network:

```bash
sudo virsh net-define /tmp/ocp-poc-network.xml
sudo virsh net-start ocp-poc
sudo virsh net-autostart ocp-poc
```

Add the bridge to the trusted firewall zone:

```bash
sudo firewall-cmd --permanent --zone=trusted --add-interface=virbr-ocp
sudo firewall-cmd --reload
```

## Step 3: Create the Libvirt Storage Pool

```bash
sudo mkdir -p /var/lib/libvirt/images/ocp-poc
sudo virsh pool-define-as ocp-poc dir --target /var/lib/libvirt/images/ocp-poc
sudo virsh pool-start ocp-poc
sudo virsh pool-autostart ocp-poc
```

## Step 4: Download the OpenShift Installer and CLI

```bash
OCP_VERSION=stable-4.20
INSTALLER_DIR=~/ocp-installer
mkdir -p ${INSTALLER_DIR}

curl -L "https://mirror.openshift.com/pub/openshift-v4/clients/ocp/${OCP_VERSION}/openshift-install-linux.tar.gz" | tar xzf - -C ${INSTALLER_DIR}
curl -L "https://mirror.openshift.com/pub/openshift-v4/clients/ocp/${OCP_VERSION}/openshift-client-linux.tar.gz" | tar xzf - -C ${INSTALLER_DIR}

sudo cp ${INSTALLER_DIR}/oc /usr/local/bin/
sudo cp ${INSTALLER_DIR}/kubectl /usr/local/bin/
```

Verify:

```bash
${INSTALLER_DIR}/openshift-install version
oc version --client
```

## Step 5: Prepare the Install Configuration

Create the cluster directory and copy the templates from this repo:

```bash
CLUSTER_DIR=${INSTALLER_DIR}/cluster
mkdir -p ${CLUSTER_DIR}
cp install-config.yaml ${CLUSTER_DIR}/
cp agent-config.yaml ${CLUSTER_DIR}/
```

Edit `${CLUSTER_DIR}/install-config.yaml`:

- Replace `<PULL_SECRET>` with the contents of `~/.pull-secret`
- Replace `<SSH_PUBLIC_KEY>` with the contents of `~/.ssh/id_rsa.pub`
- Update `baseDomain`, `metadata.name`, network CIDRs, VIPs, and MAC addresses to match your environment

Edit `${CLUSTER_DIR}/agent-config.yaml`:

- Update MAC addresses, IP addresses, and interface names to match your environment
- Update the `rendezvousIP` to match `master-0`'s IP address
- Update worker node MAC addresses to match your bare metal NICs

Back up the configs before generating the ISO (the installer consumes them):

```bash
cp ${CLUSTER_DIR}/install-config.yaml ${INSTALLER_DIR}/install-config.yaml.bak
cp ${CLUSTER_DIR}/agent-config.yaml ${INSTALLER_DIR}/agent-config.yaml.bak
```

## Step 6: Generate the Agent ISO

```bash
${INSTALLER_DIR}/openshift-install agent create image --dir ${CLUSTER_DIR}
```

This produces `${CLUSTER_DIR}/agent.x86_64.iso`.

Copy the ISO to the libvirt storage pool:

```bash
sudo cp ${CLUSTER_DIR}/agent.x86_64.iso /var/lib/libvirt/images/ocp-poc/
```

## Step 7: Create the Control Plane VMs

Create disk images for each control plane node:

```bash
for i in 0 1 2; do
  sudo qemu-img create -f qcow2 /var/lib/libvirt/images/ocp-poc/master-${i}.qcow2 120G
done
```

Create the VMs:

```bash
for i in 0 1 2; do
  MAC="52:54:00:aa:bb:0${i}"
  sudo virt-install \
    --name master-${i} \
    --memory 16384 \
    --vcpus 8 \
    --disk /var/lib/libvirt/images/ocp-poc/master-${i}.qcow2,bus=virtio \
    --cdrom /var/lib/libvirt/images/ocp-poc/agent.x86_64.iso \
    --network network=ocp-poc,mac=${MAC},model=virtio \
    --os-variant rhel9.4 \
    --cpu host-passthrough \
    --graphics vnc,listen=0.0.0.0 \
    --noautoconsole \
    --boot hd,cdrom
done
```

Verify the VMs are running:

```bash
sudo virsh list
```

## Step 8: Boot the Worker Nodes

Write the agent ISO to USB drives for each worker node:

```bash
sudo dd if=${CLUSTER_DIR}/agent.x86_64.iso of=/dev/sdX bs=4M status=progress
sync
```

Replace `/dev/sdX` with the correct device for each USB drive. Boot each bare metal worker node from its USB drive.

Alternatively, serve the ISO via PXE or a BMC virtual media mount if your hardware supports it.

## Step 9: Monitor the Installation

Watch the bootstrap process:

```bash
${INSTALLER_DIR}/openshift-install agent wait-for bootstrap-complete --dir ${CLUSTER_DIR} --log-level=info
```

Then wait for the full installation to complete:

```bash
${INSTALLER_DIR}/openshift-install agent wait-for install-complete --dir ${CLUSTER_DIR} --log-level=info
```

This typically takes 30-45 minutes. The installer will print the console URL and kubeadmin credentials when finished.

## Step 10: Access the Cluster

Set the kubeconfig:

```bash
export KUBECONFIG=${CLUSTER_DIR}/auth/kubeconfig
```

Verify the cluster:

```bash
oc get nodes
oc get clusteroperators
```

The kubeadmin password is at:

```bash
cat ${CLUSTER_DIR}/auth/kubeadmin-password
```

The web console is available at `https://console-openshift-console.apps.ocp-poc.example.com`.

## Teardown

Destroy and remove the control plane VMs:

```bash
for i in 0 1 2; do
  sudo virsh destroy master-${i}
  sudo virsh undefine master-${i}
done
```

Remove the storage:

```bash
sudo rm -f /var/lib/libvirt/images/ocp-poc/*.qcow2
sudo rm -f /var/lib/libvirt/images/ocp-poc/agent.x86_64.iso
```

Remove the network:

```bash
sudo virsh net-destroy ocp-poc
sudo virsh net-undefine ocp-poc
```

Remove the storage pool:

```bash
sudo virsh pool-destroy ocp-poc
sudo virsh pool-undefine ocp-poc
sudo rm -rf /var/lib/libvirt/images/ocp-poc
```

Clean up the installer directory:

```bash
rm -rf ~/ocp-installer
```
