# The Amazon EKS Distro

## Small, fast, and opinionated. Get Amazon EKS anywhere

This is a single-binary package of the Amazon EKS Distro. One command to
install and a second command to cluster, the EKS snap gives you an EKS
Distro on rails anywhere you can get Ubuntu machines.

# Install

On any recent Ubuntu machine, or another Linux distribution with [snap
support](https://snapcraft.io/docs/installing-snapd):

```
sudo snap install eks
```

To form a multi-node cluster call `eks add-node` on any existing cluster
member to get a token, followed by `eks join <token>` on the new machine.

# Default components

For optimal compatibility with EKS, this Kubernetes distro will launch a
very specific set of component services automatically. These provide a
standardised K8s environment that simplifies workload compatibility with EKS
on AWS itself.

When you initialize the cluster, it will also fetch and enable:

 * `coredns` - DNS services for services on this EKS cluster
 * `metrics-server` - K8s Metrics Server for API access
 * `storage` - Storage class; allocates storage from host directory

The EKS snap will automatically detect if it is on AWS, and if so it will also
enable:

 * `aws-iam-authenticator` - login to your cluster with AWS IAM credentials

## Stripped down MicroK8s with EKS components

This EKS Distro is based on [MicroK8s](https://snapcraft.io/microk8s). It
has a stripped down set of addons and is designed to feel great for AWS
users only.  It assumes you have AWS IAM credentials for identity and
provides default services that are compatible with EKS on EC2. See `eks
status` for component information.
