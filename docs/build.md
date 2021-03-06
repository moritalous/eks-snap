# Building the snap from source

[Building a snap](https://snapcraft.io/docs/snapcraft-overview) is done by running `snapcraft` on the root of the project.
A VM managed by Multipass is spawned to contain the build process. If you don’t have Multipass installed,
snapcraft will first prompt for its automatic installation.

Alternatively, you can build the snap in an LXC container.
Snapcraft and LXD are needed in this case:
```
sudo snap install snapcraft --classic
sudo snap install lxd
sudo apt-get remove lxd* -y
sudo apt-get remove lxc* -y
sudo lxd init
sudo usermod -a -G lxd ${USER}
```

Build the snap with:
```
git clone http://github.com/canonical/eks-snap
cd ./eks-snap/
snapcraft --use-lxd
```

Install the newly compiled snap:
```
sudo snap install eks_*_amd64.snap --classic --dangerous
```

## Building a custom EKS package

To produce a custom build with specific component versions we cannot use the snapcraft build process on the host OS. We need to
[prepare an LXC](https://forum.snapcraft.io/t/how-to-create-a-lxd-container-for-snap-development/4658)
container with Ubuntu 16:04 and snapcraft:
```
lxc launch ubuntu:16.04 --ephemeral test-build
lxc exec test-build -- snap install snapcraft --classic
lxc exec test-build -- apt update
lxc exec test-build -- git clone https://github.com/canonical/eks-snap
```

We can then set the following environment variables prior to building:
 - KUBE_VERSION: kubernetes release to package. Defaults to latest stable.
 - CNI_VERSION: version of CNI tools.
 - KUBE_TRACK: kubernetes release series (e.g., 1.10) to package. Defaults to latest stable.
 - RUNC_COMMIT: the commit hash from which to build runc
 - CONTAINERD_COMMIT: the commit hash from which to build containerd
 - KUBERNETES_REPOSITORY: build the kubernetes binaries from this repository instead of getting them from upstream
 - KUBERNETES_COMMIT: commit to be used from KUBERNETES_REPOSITORY for building the kubernetes binaries


For building we prepend the variables we need as well as `SNAPCRAFT_BUILD_ENVIRONMENT=host` so the current LXC container is used. For example to build the MicroK8s snap for Kubernetes v1.9.6 we:
```
lxc exec test-build -- sh -c "cd eks-snap && SNAPCRAFT_BUILD_ENVIRONMENT=host KUBE_VERSION=v1.9.6 snapcraft"
```

The produced snap is inside the ephemeral LXC container, we need to copy it to the host:
```
lxc file pull test-build/root/eks/eks_v1.9.6_amd64.snap .
```

#### Installing the snap
```
snap install eks_latest_amd64.snap --classic --dangerous
```

## Assembling the Calico CNI manifest

Building the manifest is subject to the upstream calico project k8s installation process.
At the time of the v3.13.2 release. The `calico.yaml` manifest is a slightly modified version of:
`https://docs.projectcalico.org/manifests/calico.yaml`:
- CALICO_IPV4POOL_CIDR was set to "10.1.0.0/16"
- CNI_NET_DIR was set to "/var/snap/eks/current/args/cni-network"
- We set the following mount paths:
  1. var-run-calico to /var/snap/eks/current/var/run/calico
  1. var-lib-calico to /var/snap/eks/current/var/lib/calico
  1. cni-bin-dir to /var/snap/eks/current/opt/cni/bin
  1. cni-net-dir to /var/snap/eks/current/args/cni-network
  1. host-local-net-dir to /var/snap/eks/current/var/lib/cni/networks
  1. policysync to /var/snap/eks/current/var/run/nodeagent
- We enabled vxlan following the instructions in [the official docs.](https://docs.projectcalico.org/getting-started/kubernetes/installation/config-options#switching-from-ip-in-ip-to-vxlan)
- FELIX_LOGSEVERITYSCREEN was set to "error"
- we set the IP autodetection method to
```dtd
            - name: IP_AUTODETECTION_METHOD
              value: "first-found"
```

## References

- https://snapcraft.io/docs/snapcraft-overview
- https://forum.snapcraft.io/t/how-to-create-a-lxd-container-for-snap-development/4658
- https://docs.projectcalico.org/getting-started/kubernetes/installation/config-options#switching-from-ip-in-ip-to-vxlan
