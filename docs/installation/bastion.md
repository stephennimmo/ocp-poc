# Step 1: Prepare the Bastion Host

The bastion host is a RHEL 9.x machine used for managing the installation and interacting with the clusters.

## Install Required Packages

```bash
sudo dnf install -y bash-completion curl jq bind-utils
```

## Download the OpenShift CLI

```bash
OCP_VERSION=stable-4.20
INSTALLER_DIR=~/ocp-installer
mkdir -p ${INSTALLER_DIR}

curl -L "https://mirror.openshift.com/pub/openshift-v4/clients/ocp/${OCP_VERSION}/openshift-install-linux.tar.gz" | tar xzf - -C ${INSTALLER_DIR}
curl -L "https://mirror.openshift.com/pub/openshift-v4/clients/ocp/${OCP_VERSION}/openshift-client-linux.tar.gz" | tar xzf - -C ${INSTALLER_DIR}

sudo cp ${INSTALLER_DIR}/oc /usr/local/bin/
sudo cp ${INSTALLER_DIR}/kubectl /usr/local/bin/
sudo cp ${INSTALLER_DIR}/openshift-install /usr/local/bin/
```

Verify:

```bash
openshift-install version
oc version --client
```

## Set Up SSH Keys

Ensure an SSH key pair exists:

```bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa -N ""
```

## Pull Secret

Download the pull secret from [console.redhat.com](https://console.redhat.com/openshift/install/pull-secret) and save it to `~/.pull-secret`.
