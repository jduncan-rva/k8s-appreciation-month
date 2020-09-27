# Lab Session One

> These documents are my working notes for each of our weekly sessions. Think of them as a Cliff's Notes version of Kubernetes the Hard Way with some of my own commentary. If we find anything substantive in our sessions, we'll be sure to open and link Pull Requests against the KTHW source of truth on Github.
>
> -jduncan

## Persistent Notes

* GDoc (internal to Google) - [go/k8s-appreciation-month](go/k8s-appreciation-month) 

## Weclome to Kubernetes Appreciation Month!

In our first session we'll be covering these labs from [Kubernetes The Hard Way](https://github.com/kelseyhightower/kubernetes-the-hard-way). 

* [Prerequisites](https://github.com/kelseyhightower/kubernetes-the-hard-way/blob/master/docs/01-prerequisites.md)
* [Installing the Client Tools](https://github.com/kelseyhightower/kubernetes-the-hard-way/blob/master/docs/02-client-tools.md)
* [Provisioning Compute Resources](https://github.com/kelseyhightower/kubernetes-the-hard-way/blob/master/docs/03-compute-resources.md)
* [Provisioning the CA and Generating TLS Certificates](https://github.com/kelseyhightower/kubernetes-the-hard-way/blob/master/docs/04-certificate-authority.md)

## Prerequisites

[*KTHW Link*](https://github.com/kelseyhightower/kubernetes-the-hard-way/blob/master/docs/01-prerequisites.md) for this lab content.

### Configuring Google Cloud SDK

1. Test the version of Google Cloud SDK on your system. You'll need to update if you're running anything below 301.0.0.

    `gcloud version`

2. Run `gcloud init` to make sure you're authenticated with the proper account. If you're new to the Google SDK, this will help you set up everything. If you're experienced with `gcloud`, this step may be optional.

    `gcloud init`
    
3. Once you're account is set up, run `gcloud auth login` to make sure you're logged in as the proper user.

    `gcloud auth login`

4. Check the currently available regions in GCP with the following command. Pick something geographically close to where you're located.

    `gcloud compute zones list`
    
    I'll be using the `us-east4-a` zone in the `us-east4` region.
    
5. Once you've picked a region and zone, tell `gcloud` to use them with these commands: 

```
    gcloud config set compute/region $YOUR_REGION
    gcloud config set compute/zone $YOUR_ZONE
```

With your configuration set, you're ready to decide how you want to handle multiple simultaneous SSH sessions to configure your cluster.

### Session Handling

Throughout the labs for K8s-Apprecation-Month you'll be accessing multiple hosts simultaneously to configure your Kubernetes cluster. A multiplexing tool like [`tmux`](https://github.com/tmux/tmux/wiki). There are many ways to install `tmux` on to Linux and MacOS systems. There's probably even a version that works with WSL on Windows 10, but that's far outside of our scope for this lab of labs.

If you don't want to use `tmux`, most modern terminal emulators can do multiple panes or tabs. Use whatever you're comfortable with.

With `gcloud` configured and your session management preferences ready to go, Lab 1 is complete we're ready to get some additional client tooling ready.

## Client Tools

[*KTHW Link*](https://github.com/kelseyhightower/kubernetes-the-hard-way/blob/master/docs/02-client-tools.md) for this lab content.

Kubernetes uses [TLS encryption](https://en.wikipedia.org/wiki/Transport_Layer_Security) extensively. To create the needed encryption keys and certificates we'll use Clouflare's free tools `cfssl` and `cfssljson`. 

Additionally we'll install `kubectl`, the primary command line tool to interface with kubernetes in this lab.

### Installing PKI infrastructure

*This is taken almost verbatim from the KTHW lab link*

The `cfssl` and `cfssljson` command line utilities will be used to provision a [PKI Infrastructure](https://en.wikipedia.org/wiki/Public_key_infrastructure) and generate TLS certificates.

Download and install `cfssl` and `cfssljson`:

#### OS X

```
curl -o cfssl https://storage.googleapis.com/kubernetes-the-hard-way/cfssl/1.4.1/darwin/cfssl
curl -o cfssljson https://storage.googleapis.com/kubernetes-the-hard-way/cfssl/1.4.1/darwin/cfssljson

chmod +x cfssl cfssljson

sudo mv cfssl cfssljson /usr/local/bin/
```

Some OS X users may experience problems using the pre-built binaries in which case [Homebrew](https://brew.sh) might be a better option:

```
brew install cfssl
```

#### Linux

```
wget -q --show-progress --https-only --timestamping \
  https://storage.googleapis.com/kubernetes-the-hard-way/cfssl/1.4.1/linux/cfssl \
  https://storage.googleapis.com/kubernetes-the-hard-way/cfssl/1.4.1/linux/cfssljson
  
chmod +x cfssl cfssljson

sudo mv cfssl cfssljson /usr/local/bin/
```

#### Verifying cfssl and cfssljson

To verify `cfssl` and `cfssljson` version 1.4.1 or higher is installed.

```
cfssl version

Version: 1.4.1
Runtime: go1.12.12
```

```
cfssljson --version

Version: 1.4.1
Runtime: go1.12.12
```

### Installing kubectl

*This is taken almost verbatim from the KTHW lab link*

The `kubectl` command line utility is used to interact with the Kubernetes API Server. Download and install `kubectl` from the official release binaries:

#### OS X

```
curl -o kubectl https://storage.googleapis.com/kubernetes-release/release/v1.18.6/bin/darwin/amd64/kubectl

chmod +x kubectl

sudo mv kubectl /usr/local/bin/
```

#### Linux

```
wget https://storage.googleapis.com/kubernetes-release/release/v1.18.6/bin/linux/amd64/kubectl

chmod +x kubectl

sudo mv kubectl /usr/local/bin/
```

#### Verification

Verify `kubectl` version 1.18.6 or higher is installed:

```
kubectl version --client

Client Version: version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.6", GitCommit:"dff82dc0de47299ab66c83c626e08b245ab19037", GitTreeState:"clean", BuildDate:"2020-07-15T16:58:53Z", GoVersion:"go1.13.9", Compiler:"gc", Platform:"linux/amd64"}
```

With your PKI tools and `kubectl` installed you have the tools you need to deploy your cluster. The next lab provisions your compute instances in GCP that will become your Kubernetes cluster.

## Provisioning Compute and Networking

[*KTHW Lab Link*](https://github.com/kelseyhightower/kubernetes-the-hard-way/blob/master/docs/03-compute-resources.md)

The Kubernetes cluster we're designing and deploying will exist in the single zone you configured as your default zone in the [Prerequisites Lab](https://hackmd.io/6NFDYrWdQC677I-arkQWmg?both#Prerequisites). In that zone we'll begin by deploying a VPC for the cluster. 

### Cluster Networking

Kubernetes assumes simple Layer 3 routability between all nodes in its cluster. Latency between control plane nodes is best kept under 10ms. 

Latency between application nodes or between application nodes and control plane nodes can be higher; some people have had success with latency as high as 150ms. But simple is almost always best. So we'll place all of our nodes in a single VPC in a single zone.

> There are lots of ways to implement the [networking model](https://kubernetes.io/docs/concepts/cluster-administration/networking) in Kubernetes. For our infrastructure, we'll leverage the [GCE model implementation](https://kubernetes.io/docs/concepts/cluster-administration/networking/#google-compute-engine-gce).
> 
> When you need complex restrictions between nodes in your cluster, you can leverage [Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/). While really cool, they're out of scope for our efforts.

#### VPC Configuration

To properly configure the VPC: 

1. Provision the VPC

    `gcloud compute networks create kubernetes-the-hard-way --subnet-mode custom`

2. Create a subnet in the VPC

    ```
    gcloud compute networks subnets create kubernetes \
      --network kubernetes-the-hard-way \
      --range 10.240.0.0/24
    ```
    
    This subnet can host up to 254 compute hosts. But you won't need quite that many for this lab.

3. Create a firewall rule for the VPC to allow unrestricted internal communications for all protocols.

    ```
    gcloud compute firewall-rules create kubernetes-the-hard-way-allow-internal \
      --allow tcp,udp,icmp \
      --network kubernetes-the-hard-way \
      --source-ranges 10.240.0.0/24,10.200.0.0/16
    ```
    
    This is not a production-ready configuration. This is not a cluster that you should take into production!
    
4. Create a VPC firewall rule to allow inbound SSH, ICMP, and HTTPS

    ```
    gcloud compute firewall-rules create kubernetes-the-hard-way-allow-external \
      --allow tcp:22,tcp:6443,icmp \
      --network kubernetes-the-hard-way \
      --source-ranges 0.0.0.0/0
    ```
    
    Later, you'll configure a [GCP Load Balancer](https://cloud.google.com/load-balancing/docs/network) to access the Kubernetes API server.
    
5. Allocate a Static IP for the External Load Balancer. 

    ```
    gcloud compute addresses create kubernetes-the-hard-way \
      --region $(gcloud config get-value compute/region)
    ```
    
When these steps are complete, confirm the firewall rules are in place and the static IP has been allocated.

```
gcloud compute firewall-rules list --filter="network:kubernetes-the-hard-way"

NAME                                    NETWORK                  DIRECTION  PRIORITY  ALLOW                 DENY  DISABLED
kubernetes-the-hard-way-allow-external  kubernetes-the-hard-way  INGRESS    1000      tcp:22,tcp:6443,icmp        False
kubernetes-the-hard-way-allow-internal  kubernetes-the-hard-way  INGRESS    1000      tcp,udp,icmp                False

gcloud compute addresses list --filter="name=('kubernetes-the-hard-way')"

NAME                     ADDRESS/RANGE   TYPE      PURPOSE  NETWORK  REGION    SUBNET  STATUS
kubernetes-the-hard-way  XX.XXX.XXX.XXX  EXTERNAL                    us-east4          RESERVED
```

With your firewall rules and static IP address properly configured, the networking resources for your cluster are ready. Next, you'll configure your cluster's compute instances.

### Cluster Compute Nodes

The nodes in this Kubernetes cluster will run [Ubuntu Server](https://www.ubuntu.com/server) 20.04 because it has solid support for the [containerd container runtime](https://github.com/containerd/containerd). 

Each node will be configured with a static internal IP address to make the kubernetes bootstrapping process a little simpler.

1. Deploy 3 control plane nodes

    ```
    for i in 0 1 2; do
      gcloud compute instances create controller-${i} \
    --async \
    --boot-disk-size 200GB \
    --can-ip-forward \
    --image-family ubuntu-2004-lts \
    --image-project ubuntu-os-cloud \
    --machine-type e2-standard-2 \
    --private-network-ip 10.240.0.1${i} \
    --scopes compute-rw,storage-ro,service-management,service-control,logging-write,monitoring \
    --subnet kubernetes \
    --tags kubernetes-the-hard-way,controller
    done
    ```
    
2. Deploy 2 application nodes

    ```
    for i in 0 1 2; do
      gcloud compute instances create worker-${i} \
    --async \
    --boot-disk-size 200GB \
    --can-ip-forward \
    --image-family ubuntu-2004-lts \
    --image-project ubuntu-os-cloud \
    --machine-type e2-standard-2 \
    --metadata pod-cidr=10.200.${i}.0/24 \
    --private-network-ip 10.240.0.2${i} \
    --scopes compute-rw,storage-ro,service-management,service-control,logging-write,monitoring \
    --subnet kubernetes \
    --tags kubernetes-the-hard-way,worker
    done
    ```
    
    *Note - the `metadata` parameter for your application nodes defines the subnet used for the pods on each node. They must be part of the network defined by the `--cluster-cidr` for the `kubernetes-controller-manager` service. For this lab, that value will be `10.200.0.0/16`, and you'll configure it in a later lab.*
    
Once complete, verify these nodes are all functional: 

```
gcloud compute instances list --filter="tags.items=kubernetes-the-hard-way"

NAME          ZONE        MACHINE_TYPE   PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP    STATUS
controller-0  us-west1-c  e2-standard-2               10.240.0.10  XX.XX.XX.XXX   RUNNING
controller-1  us-west1-c  e2-standard-2               10.240.0.11  XX.XXX.XXX.XX  RUNNING
controller-2  us-west1-c  e2-standard-2               10.240.0.12  XX.XXX.XX.XXX  RUNNING
worker-0      us-west1-c  e2-standard-2               10.240.0.20  XX.XX.XXX.XXX  RUNNING
worker-1      us-west1-c  e2-standard-2               10.240.0.21  XX.XX.XX.XXX   RUNNING
worker-2      us-west1-c  e2-standard-2               10.240.0.22  XX.XXX.XX.XX   RUNNING
```
### Accessing nodes with SSH

You'll use SSH to continue configuring your cluster from here. The `gcloud` utility can automatically store your SSH keys for hosts. To test out connectivity, use `gcloud` to connect to `controller-0` via ssh. There's not a need to enter a password for this SSH key. It will only be used for this lab.

```
gcloud compute ssh controller-0

WARNING: The public SSH key file for gcloud does not exist.
WARNING: The private SSH key file for gcloud does not exist.
WARNING: You do not have an SSH key for gcloud.
WARNING: SSH keygen will be executed to generate a key.
Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/$USER/.ssh/google_compute_engine.
Your public key has been saved in /home/$USER/.ssh/google_compute_engine.pub.
The key fingerprint is:
SHA256:nz1i8jHmgQuGt+WscqP5SeIaSy5wyIJeL71MuV+QruE $USER@$HOSTNAME
The key's randomart image is:
+---[RSA 2048]----+
|                 |
|                 |
|                 |
|        .        |
|o.     oS        |
|=... .o .o o     |
|+.+ =+=.+.X o    |
|.+ ==O*B.B = .   |
| .+.=EB++ o      |
+----[SHA256]-----+
Updating project ssh metadata...-Updated [https://www.googleapis.com/compute/v1/projects/$PROJECT_ID].
Updating project ssh metadata...done.
Waiting for SSH key to propagate.
```

Once this process finishes you'll be connected to `controller-0`. 

```
Welcome to Ubuntu 20.04 LTS (GNU/Linux 5.4.0-1019-gcp x86_64)
...
```

Type `exit` to close your ssh connection.

## Certificate Authority and TLS Certificates

TODO

