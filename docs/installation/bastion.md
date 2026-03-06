# Step 1: Prepare the Bastion Host

The bastion host is a RHEL 9.x machine used for managing the installation and interacting with the clusters.

## Install Required Packages

```bash
sudo dnf install -y bash-completion curl jq bind-utils
```

## Download the OpenShift CLI

```bash
OCP_VERSION=stable-4.20
TMPDIR=$(mktemp -d)

curl -L "https://mirror.openshift.com/pub/openshift-v4/clients/ocp/${OCP_VERSION}/openshift-install-linux.tar.gz" | tar xzf - -C ${TMPDIR}
curl -L "https://mirror.openshift.com/pub/openshift-v4/clients/ocp/${OCP_VERSION}/openshift-client-linux.tar.gz" | tar xzf - -C ${TMPDIR}

sudo cp ${TMPDIR}/oc /usr/local/bin/
sudo cp ${TMPDIR}/kubectl /usr/local/bin/
sudo cp ${TMPDIR}/openshift-install /usr/local/bin/

rm -rf ${TMPDIR}
```

Verify:

```bash
openshift-install version
oc version --client
```

## Set Up SSH Keys

Ensure an SSH key pair exists:

```bash
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519 -N ""
```

## Pull Secret

Download the pull secret from [console.redhat.com](https://console.redhat.com/openshift/install/pull-secret) and save it to `~/.pull-secret`.
