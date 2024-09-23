# K8s_Networking_Cillium
_In this lab, you will independently install Cilium and the Hubble CLI. The objective is to familiarize you with the installation process and usage of these command-line interfaces (CLIs) for future tasks.

The lab will be conducted on a basic Linux machine, providing you with hands-on experience in setting up and utilizing these tools.
_


# Installing cillium and Hubble
https://docs.cilium.io/en/stable/gettingstarted/k8s-install-default/

# 1. Create the Cluster

export NAME="$(whoami)-$RANDOM"
cat <<EOF >eks-config.yaml
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: ${NAME}
  region: eu-west-1

managedNodeGroups:
- name: ng-1
  desiredCapacity: 2
  privateNetworking: true
  # taint nodes so that application pods are
  # not scheduled/executed until Cilium is deployed.
  # Alternatively, see the note below.
  taints:
   - key: "node.cilium.io/agent-not-ready"
     value: "true"
     effect: "NoExecute"
EOF
eksctl create cluster -f ./eks-config.yaml


# 2. Install the Cilium CLI

_check version_
cilium version --client

CILIUM_CLI_VERSION=$(curl -s https://raw.githubusercontent.com/cilium/cilium-cli/main/stable.txt)
CLI_ARCH=amd64
if [ "$(uname -m)" = "aarch64" ]; then CLI_ARCH=arm64; fi
curl -L --fail --remote-name-all https://github.com/cilium/cilium-cli/releases/download/${CILIUM_CLI_VERSION}/cilium-linux-${CLI_ARCH}.tar.gz{,.sha256sum}
sha256sum --check cilium-linux-${CLI_ARCH}.tar.gz.sha256sum
sudo tar xzvfC cilium-linux-${CLI_ARCH}.tar.gz /usr/local/bin
rm cilium-linux-${CLI_ARCH}.tar.gz{,.sha256sum}


cillium install --version 1.15.4 --wait 

# Run the following command to validate that your cluster has proper network connectivity:
 
cillium status --wait

#to check conectivity 
cillium connectivity test 


# Setting up Hubble Observability
_https://docs.cilium.io/en/stable/gettingstarted/hubble_setup/#install-the-hubble-client_

cilium hubble enable

#using Helm 

helm upgrade cilium cilium/cilium --version 1.16.1 \
   --namespace kube-system \
   --reuse-values \
   --set hubble.relay.enabled=true \
   --set hubble.ui.enabled=true

#Install the Hubble Client

HUBBLE_VERSION=$(curl -s https://raw.githubusercontent.com/cilium/hubble/master/stable.txt)
HUBBLE_ARCH=amd64
if [ "$(uname -m)" = "aarch64" ]; then HUBBLE_ARCH=arm64; fi
curl -L --fail --remote-name-all https://github.com/cilium/hubble/releases/download/$HUBBLE_VERSION/hubble-linux-${HUBBLE_ARCH}.tar.gz{,.sha256sum}
sha256sum --check hubble-linux-${HUBBLE_ARCH}.tar.gz.sha256sum
sudo tar xzvfC hubble-linux-${HUBBLE_ARCH}.tar.gz /usr/local/bin
rm hubble-linux-${HUBBLE_ARCH}.tar.gz{,.sha256sum}


#Validate Hubble API Access

cilium hubble port-forward&

helm list -n kube-systen --filter "cluster

helm repo add cilium https://helm.cillium.io/

cillium hubble port forward&


hubble observe

