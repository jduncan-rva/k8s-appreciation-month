# Lab Session One

> These documents are my working notes for each of our weekly sessions. Think of them as a Cliff's Notes version of Kubernetes the Hard Way with some of my own commentary. If we find anything substantive in our sessions, we'll be sure to open and link Pull Requests against the KTHW source of truth on Github.
>
> -jduncan

## Persistent Notes and links

* Kubernetes The Hard Way - https://github.com/kelseyhightower/kubernetes-the-hard-way
* GDoc (internal to Google) - [go/k8s-appreciation-month](go/k8s-appreciation-month)
* This project in Github - https://github.com/jduncan-rva/k8s-appreciation-month
* Session One - https://hackmd.io/6NFDYrWdQC677I-arkQWmg
* Session Two - https://hackmd.io/RIEmIlXpRouNlVKhBQ3Thw
* Session Three - https://hackmd.io/xs65EKFhRi-PnEDwBdzSYQ
* Session Four - https://hackmd.io/S62A412aQb2Q5pBsVsDqgg

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
    
    If you're setting up a new account for the first time you'll be given the option to select a default zone and region after you log in. If you do that as part of this command, you can skip doing it in steps 3-5. 
    
3. Once you're account is set up, run `gcloud auth login` to make sure you're logged in as the proper user.

    `gcloud auth login`

4. Check the currently available regions in GCP with the following command. Pick something geographically close to where you're located.

    `gcloud compute zones list`
    
    I'll be using the `us-east4-a` zone in the `us-east4` region. Note that if this is a new project, you may be prompted to enable the compute API. This is normal.
    
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

1. Download the OSX binaries
    ```
    curl -o cfssl https://storage.googleapis.com/kubernetes-the-hard-way/cfssl/1.4.1/darwin/cfssl
    curl -o cfssljson https://storage.googleapis.com/kubernetes-the-hard-way/cfssl/1.4.1/darwin/cfssljson
    ```

2. Make the downloaded files executable
    ```
    chmod +x cfssl cfssljson
    ```

3. Move the files to a directory in $PATH
    ```
    sudo mv cfssl cfssljson /usr/local/bin/
    ```

Some OS X users may experience problems using the pre-built binaries in which case [Homebrew](https://brew.sh) might be a better option:

```
brew install cfssl
```

#### Linux

1. Download the Linux binaries

    ```
    wget -q --show-progress --https-only --timestamping \
      https://storage.googleapis.com/kubernetes-the-hard-way/cfssl/1.4.1/linux/cfssl \
      https://storage.googleapis.com/kubernetes-the-hard-way/cfssl/1.4.1/linux/cfssljson
    ```

2. Make the Linux binaries executable
    ```
    chmod +x cfssl cfssljson
    ```

3. Move the files to a directory in $PATH
    ```
    sudo mv cfssl cfssljson /usr/local/bin/
    ```
    
#### GCP Cloud Shell

If you're using a GCP Cloud Shell instance for this lab, a small change to the Linux steps above will make your life easier. Steps 1 and 2 remain the same. But instead of step 3, run the following:

1. Create a `bin` directory in your `$HOME` directory
    ```
    mkdir $HOME/bin
    ```
    
2. Add your new directory to your `$PATH` variable.
    ```
    export PATH=$PATH:$HOME/bin
    ```
    
    to have this persist across Cloud Shell restarts, you'll need to add this command to your `~/.bashrc` or other shell `.rc` file. That is a simple task, but out of scope for this workshop.
    
3. Move `cfssl` and `cfssljson` to `$HOME/bin`
    ```
    mv cfssl cfssljson $HOME/bin/
    ```
    
You're now ready to move on to the next section and verify the functionality of your new tools.

#### Verifying cfssl and cfssljson

To verify `cfssl` and `cfssljson` version 1.4.1 or higher is installed.

command:
```
cfssl version
```

output:
```
Version: 1.4.1
Runtime: go1.12.12
```

command:
```
cfssljson --version
```

output:
```
Version: 1.4.1
Runtime: go1.12.12
```

### Installing kubectl

*This is taken almost verbatim from the KTHW lab link*

The `kubectl` command line utility is used to interact with the Kubernetes API Server. If you're using a Cloud Shell instance, `kubectl` is already installed. For other systems, download and install `kubectl` from the official release binaries:

#### OS X

1. Download the kubectl OSX binary
    ```
    curl -o kubectl https://storage.googleapis.com/kubernetes-release/release/v1.18.6/bin/darwin/amd64/kubectl
    ```

2. Make `kubectl` executable
    ```
    chmod +x kubectl
    ```

3. Move `kubectl` to a directory in your $PATH variable
    ```
    sudo mv kubectl /usr/local/bin/
    ```

#### Linux
1. Download the Linux binary
    ```
    wget https://storage.googleapis.com/kubernetes-release/release/v1.18.6/bin/linux/amd64/kubectl
    ```

2. Make the file excutable 
    ```
    chmod +x kubectl
    ```

3. Move `kubectl` into a directory in $PATH
    ```
    sudo mv kubectl /usr/local/bin/
    ```

#### Verification

Verify `kubectl` version 1.18.6 or higher is installed:

```
kubectl version --client
```

sample output:
```
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
    ```
    gcloud compute networks create kubernetes-the-hard-way --subnet-mode custom
    ```
    
    output:
    ```
    Created [https://www.googleapis.com/compute/v1/projects/kthw-jduncan-dev/global/networks/kubernetes-the-hard-way].
    NAME                     SUBNET_MODE  BGP_ROUTING_MODE  IPV4_RANGE  GATEWAY_IPV4
    kubernetes-the-hard-way  CUSTOM       REGIONAL

    Instances on this network will not be reachable until firewall rules
    are created. As an example, you can allow all internal traffic between
    instances as well as SSH, RDP, and ICMP by running:

    $ gcloud compute firewall-rules create <FIREWALL_NAME> --network kubernetes-the-hard-way --allow tcp,udp,icmp --source-ranges <IP_RANGE>
    $ gcloud compute firewall-rules create <FIREWALL_NAME> --network kubernetes-the-hard-way --allow tcp:22,tcp:3389,icmp
    ```

2. Create a subnet in the VPC

    ```
    gcloud compute networks subnets create kubernetes \
      --network kubernetes-the-hard-way \
      --range 10.240.0.0/24
    ```
    
    output:
    ```
    Created [https://www.googleapis.com/compute/v1/projects/kthw-jduncan-dev/regions/us-east4/subnetworks/kubernetes].
    NAME        REGION    NETWORK                  RANGE
    kubernetes  us-east4  kubernetes-the-hard-way  10.240.0.0/24
    ```
    
    This subnet can host up to 254 compute hosts. But you won't need quite that many for this lab.

3. Create a firewall rule for the VPC to allow unrestricted internal communications for all protocols.

    ```
    gcloud compute firewall-rules create kubernetes-the-hard-way-allow-internal \
      --allow tcp,udp,icmp \
      --network kubernetes-the-hard-way \
      --source-ranges 10.240.0.0/24,10.200.0.0/16
    ```
    
    output:
    ```
    Creating firewall...⠹Created [https://www.googleapis.com/compute/v1/projects/kthw-jduncan-dev/global/firewalls/kubernetes-the-hard-way-allow-internal].
    Creating firewall...done.
    NAME                                    NETWORK                  DIRECTION  PRIORITY  ALLOW         DENY  DISABLED
    kubernetes-the-hard-way-allow-internal  kubernetes-the-hard-way  INGRESS    1000      tcp,udp,icmp        False
    ```
    
    This is not a production-ready configuration. This is not a cluster that you should take into production!
    
4. Create a VPC firewall rule to allow inbound SSH, ICMP, and HTTPS

    ```
    gcloud compute firewall-rules create kubernetes-the-hard-way-allow-external \
      --allow tcp:22,tcp:6443,icmp \
      --network kubernetes-the-hard-way \
      --source-ranges 0.0.0.0/0
    ```
    
    output:
    ```
    Creating firewall...⠹Created [https://www.googleapis.com/compute/v1/projects/kthw-jduncan-dev/global/firewalls/kubernetes-the-hard-way-allow-external].
    Creating firewall...done.
    NAME                                    NETWORK                  DIRECTION  PRIORITY  ALLOW                 DENY  DISABLED
    kubernetes-the-hard-way-allow-external  kubernetes-the-hard-way  INGRESS    1000      tcp:22,tcp:6443,icmp        False
    ```
    
    Later, you'll configure a [GCP Load Balancer](https://cloud.google.com/load-balancing/docs/network) to access the Kubernetes API server.
    
5. Allocate a Static IP for the External Load Balancer. 

    ```
    gcloud compute addresses create kubernetes-the-hard-way \
      --region $(gcloud config get-value compute/region)
    ```
    
    output:
    ```
    Created [https://www.googleapis.com/compute/v1/projects/kthw-jduncan-dev/regions/us-east4/addresses/kubernetes-the-hard-way].
    ```
    
6. Verify the firewall rules are in place.
    ```
    gcloud compute firewall-rules list --filter="network:kubernetes-the-hard-way"
    ```
    
    output:
    ```
    NAME                                    NETWORK                  DIRECTION  PRIORITY  ALLOW                 DENY  DISABLED
    kubernetes-the-hard-way-allow-external  kubernetes-the-hard-way  INGRESS    1000      tcp:22,tcp:6443,icmp        False
    kubernetes-the-hard-way-allow-internal  kubernetes-the-hard-way  INGRESS    1000      tcp,udp,icmp                False
    ```

7. Verify the Static IP address has been allocated.
    ```
    gcloud compute addresses list --filter="name=('kubernetes-the-hard-way')"
    ```
    
    output:
    ```
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
    
    output: 
    
    ```
    NOTE: The users will be charged for public IPs when VMs are created.
    Instance creation in progress for [controller-0]: https://www.googleapis.com/compute/v1/projects/kthw-jduncan-dev/zones/us-east4-a/operations/operation-1601315596631-5b063574f7cf3-5af5893b-c22194db
    Use [gcloud compute operations describe URI] command to check the status of the operation(s).
    NOTE: The users will be charged for public IPs when VMs are created.
    Instance creation in progress for [controller-1]: https://www.googleapis.com/compute/v1/projects/kthw-jduncan-dev/zones/us-east4-a/operations/operation-1601315599423-5b063577a19d6-3db6dc50-475fb775
    Use [gcloud compute operations describe URI] command to check the status of the operation(s).
    NOTE: The users will be charged for public IPs when VMs are created.
    Instance creation in progress for [controller-2]: https://www.googleapis.com/compute/v1/projects/kthw-jduncan-dev/zones/us-east4-a/operations/operation-1601315601372-5b0635797d712-2b0edb32-8fed374d
    Use [gcloud compute operations describe URI] command to check the status of the operation(s).
    ```
    
2. Deploy 3 application nodes

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
    
    output:
    
    ```
    NOTE: The users will be charged for public IPs when VMs are created.
    Instance creation in progress for [worker-0]: https://www.googleapis.com/compute/v1/projects/kthw-jduncan-dev/zones/us-east4-a/operations/operation-1601315687228-5b0635cb5e63b-ff7afeaa-d8a293f0
    Use [gcloud compute operations describe URI] command to check the status of the operation(s).
    NOTE: The users will be charged for public IPs when VMs are created.
    Instance creation in progress for [worker-1]: https://www.googleapis.com/compute/v1/projects/kthw-jduncan-dev/zones/us-east4-a/operations/operation-1601315689247-5b0635cd4b352-11d440dc-e33f84b6
    Use [gcloud compute operations describe URI] command to check the status of the operation(s).
    NOTE: The users will be charged for public IPs when VMs are created.
    Instance creation in progress for [worker-2]: https://www.googleapis.com/compute/v1/projects/kthw-jduncan-dev/zones/us-east4-a/operations/operation-1601315691256-5b0635cf35be8-277d8bfd-6095f14b
    Use [gcloud compute operations describe URI] command to check the status of the operation(s).
    ```
    
    The `metadata` parameter for your application nodes defines the subnet used for the pods on each node. They must be part of the network defined by the `--cluster-cidr` for the `kubernetes-controller-manager` service. For this lab, that value will be `10.200.0.0/16`, and you'll configure it in a later lab.*
    
3. Verify these nodes are all functional: 

    ```
    gcloud compute instances list --filter="tags.items=kubernetes-the-hard-way"
    ```
    
    output:
    ```
    NAME          ZONE        MACHINE_TYPE   PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP    STATUS
    controller-0  us-east4-a  e2-standard-2               10.240.0.10  35.245.213.69  RUNNING
    controller-1  us-east4-a  e2-standard-2               10.240.0.11  34.86.61.230   RUNNING
    controller-2  us-east4-a  e2-standard-2               10.240.0.12  35.199.31.109  RUNNING
    worker-0      us-east4-a  e2-standard-2               10.240.0.20  35.245.243.21  RUNNING
    worker-1      us-east4-a  e2-standard-2               10.240.0.21  34.86.224.225  RUNNING
    worker-2      us-east4-a  e2-standard-2               10.240.0.22  34.86.19.61    RUNNING
    ```
### Accessing nodes with SSH

You'll use SSH to continue configuring your cluster from here. The `gcloud` utility can automatically store your SSH keys for hosts. To test out connectivity, use `gcloud` to connect to `controller-0` via ssh. There's not a need to enter a password for this SSH key. It will only be used for this lab.

1. Connect to controller-0 via ssh
    ```
    gcloud compute ssh controller-0
    ```
    
    output:
    ```
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

2. Close the ssh session to controller-0
    ```
    exit
    ```

Your compute and networking resources are ready to go at this point. Next you'll begin to configure the software that will be your kubernetes cluster. You'll by creating a certificate authority and a whole load of TLS certificates.

## Certificate Authority and TLS Certificates

[*KTHW Link*](https://github.com/kelseyhightower/kubernetes-the-hard-way/blob/master/docs/04-certificate-authority.md)

By design, all traffic in and out of Kubernetes control plane services are encrypted with unique TLS certificates. Those services are: 

* kube-api-server
* kube-scheduler
* kubelet
* kube-proxy

To create and manage all these keys and certificates, you'll use the Cloudflare PKI toolkit you installed in the [Prerequisites Lab](https://hackmd.io/6NFDYrWdQC677I-arkQWmg?both#Prerequisites) to bootstrap a certificate authority (CA) and generate the needed certificates. Once created, you'll distribute them to their proper cluster node.

### Creating a Certificate Authority

1. Create ca-config.json 

    ```
    cat > ca-config.json <<EOF
    {
      "signing": {
        "default": {
          "expiry": "8760h"
        },
        "profiles": {
          "kubernetes": {
            "usages": ["signing", "key encipherment", "server auth", "client auth"],
            "expiry": "8760h"
          }
        }
      }
    }
    EOF
    ```

2. Create ca-csr.json 

    ```
    cat > ca-csr.json <<EOF
    {
      "CN": "Kubernetes",
      "key": {
        "algo": "rsa",
        "size": 2048
      },
      "names": [
        {
          "C": "US",
          "L": "Raleigh",
          "O": "Kubernetes",
          "OU": "NC",
          "ST": "North Carolina"
        }
      ]
    }
    EOF
    ```
    
3. Create your certificate authority

    ```
    cfssl gencert -initca ca-csr.json | cfssljson -bare ca
    ```
    
4. Verify your certificate was created properly
    ```
    openssl x509 -text -noout -in ca.pem
    
    Certificate:
        Data:
            Version: 3 (0x2)
            Serial Number:
            ...
    ```

#### Generated files 

* `ca-config.json`
* `ca-csr.json`
* `ca-key.pem`
* `ca.pem`

This key and certificate files are used to sign the rest of the certificates you create for your cluster. They are your certificate authority.

### Creating the admin client certificate

Each kubernetes service needs a client and server certificate. Additionally, the default `admin` user needs a client certificate to be able to authenticate to the cluster's control plane.

1. Create the Certificate Signing Request (CSR) file
    ```
    cat > admin-csr.json <<EOF
    {
      "CN": "admin",
      "key": {
        "algo": "rsa",
        "size": 2048
      },
      "names": [
        {
          "C": "US",
          "L": "Raleigh",
          "O": "system:masters",
          "OU": "Kubernetes The Hard Way",
          "ST": "North Carolina"
        }
      ]
    }
    EOF
    ```

2. Generate the signed key and certificate from the CSR file
    ```
    cfssl gencert \
      -ca=ca.pem \
      -ca-key=ca-key.pem \
      -config=ca-config.json \
      -profile=kubernetes \
      admin-csr.json | cfssljson -bare admin
    ```
    
3. Verify your admin certificate
    ```
    openssl x509 -text -noout -in admin.pem
    
    Certificate:
        Data:
            Version: 3 (0x2)
            Serial Number:
            ...
    ```

#### Generated files 

* `admin-key.pem`
* `admin.pem`

### Creating kubelet client certificates

The `kubelet` service runs on each node in your cluster. Kubelet interfaces with the container runtime on each node and reports information back to the control plane. To authoritze a node to be part of a cluster, `kubelet` uses a credential for an [authentication mechanism](https://kubernetes.io/docs/admin/authorization/node/) that confirms they're part of the `system:nodes` group. 

Each worker node needs a unique certificate that you'll create in this sexction. This certificate needs to reference both the internal and exteral IP addresses for each node. This will allow for all needed communications. The snippet below does the following for each node:

* Creates a CSR file 
* Captures the internal IP address as a variable
* Captures the external IP addresses as a variable
* Uses both IPs and the CSR to create a TLS certificate 

1. Run the following small script.
    ```
    for instance in worker-0 worker-1 worker-2; do
    cat > ${instance}-csr.json <<EOF
    {
      "CN": "system:node:${instance}",
      "key": {
        "algo": "rsa",
        "size": 2048
      },
      "names": [
        {
          "C": "US",
          "L": "Raleigh",
          "O": "system:nodes",
          "OU": "Kubernetes The Hard Way",
          "ST": "North Carolina"
        }
      ]
    }
    EOF

    EXTERNAL_IP=$(gcloud compute instances describe ${instance} \
      --format 'value(networkInterfaces[0].accessConfigs[0].natIP)')

    INTERNAL_IP=$(gcloud compute instances describe ${instance} \
      --format 'value(networkInterfaces[0].networkIP)')

    cfssl gencert \
      -ca=ca.pem \
      -ca-key=ca-key.pem \
      -config=ca-config.json \
      -hostname=${instance},${EXTERNAL_IP},${INTERNAL_IP} \
      -profile=kubernetes \
      ${instance}-csr.json | cfssljson -bare ${instance}
    done
    ```
    
2. Verify one of your kubelet certificates.
    ```
    openssl x509 -text -noout -in worker-0.pem
    
    Certificate:
        Data:
            Version: 3 (0x2)
            Serial Number:
    ```

#### Generated files 

* `worker-0-key.pem`
* `worker-0.pem`
* `worker-1-key.pem`
* `worker-1.pem`
* `worker-2-key.pem`
* `worker-2.pem`

### Creating the kube-controller-manager client certificate

This certificate will be used by clients communicating with the `kube-controller-manager` service. 

1. Generate the CSR file
    ```
    cat > kube-controller-manager-csr.json <<EOF
    {
      "CN": "system:kube-controller-manager",
      "key": {
        "algo": "rsa",
        "size": 2048
      },
      "names": [
        {
          "C": "US",
          "L": "Raleigh",
          "O": "system:kube-controller-manager",
          "OU": "Kubernetes The Hard Way",
          "ST": "North Carolina"
        }
      ]
    }
    EOF
    ```
    
2. Generate the signed certificate and key file
    ```
    cfssl gencert \
      -ca=ca.pem \
      -ca-key=ca-key.pem \
      -config=ca-config.json \
      -profile=kubernetes \
      kube-controller-manager-csr.json | cfssljson -bare kube-controller-manager

    ```
    
3. Verify the `kube-controller-manager` certificate
    ```openssl x509 -text -noout -in kube-controller-manager.pem
    Certificate:
        Data:
            Version: 3 (0x2)
            Serial Number:
            ...
    ```

#### Generated files 

* kube-controller-manager-key.pem
* kube-controller-manager.pem

### Creating the kube-proxy client certificate

This certificate is used to communicate with the `kube-proxy` service.

1. Generate the CSR file 
    ```
    cat > kube-proxy-csr.json <<EOF
    {
      "CN": "system:kube-proxy",
      "key": {
        "algo": "rsa",
        "size": 2048
      },
      "names": [
        {
          "C": "US",
          "L": "Raleigh",
          "O": "system:node-proxier",
          "OU": "Kubernetes The Hard Way",
          "ST": "North Carolina"
        }
      ]
    }
    EOF
    ```
    
2. Generate the signed certificate and key file
    ```
    cfssl gencert \
      -ca=ca.pem \
      -ca-key=ca-key.pem \
      -config=ca-config.json \
      -profile=kubernetes \
      kube-proxy-csr.json | cfssljson -bare kube-proxy
    ```
    
3. Verify the `kube-proxy` certificate.
    ```
    openssl x509 -text -noout -in kube-proxy.pem
    Certificate:
        Data:
            Version: 3 (0x2)
            Serial Number:
            ...
    ```
    
#### Generated files

* kube-proxy-key.pem
* kube-proxy.pem

### Creating the kube-scheduler client certificates

This certificate is used to communicate with the `kube-scheduler` control plane service. 

1. Generate the CSR file 
    ```
    cat > kube-scheduler-csr.json <<EOF
    {
      "CN": "system:kube-scheduler",
      "key": {
        "algo": "rsa",
        "size": 2048
      },
      "names": [
        {
          "C": "US",
          "L": "Raleigh",
          "O": "system:kube-scheduler",
          "OU": "Kubernetes The Hard Way",
          "ST": "North Carolina"
        }
      ]
    }
    EOF
    ```
    
2. Generate the signed certificate and key file
    ```
    cfssl gencert \
     -ca=ca.pem \
     -ca-key=ca-key.pem \
     -config=ca-config.json \
     -profile=kubernetes \
     kube-scheduler-csr.json | cfssljson -bare kube-scheduler
    ```
    
3. Verify the `kube-scheduler` certificate
    ```
    openssl x509 -text -noout -in kube-scheduler.pem
    Certificate:
        Data:
            Version: 3 (0x2)
            Serial Number:
            ...
    ```

#### Generated files 

* kube-scheduler-key.pem
* kube-scheduler.pem

### Creating the kubernetes API server certificate

To allow for valid access to your cluster, the kubernetes-the-hard-way static IP address will be included in the list of subject alternative names for the Kubernetes API Server certificate. This will make it a valid TLS certificate.

1. Generate the CSR file 
    ```
    cat > kubernetes-csr.json <<EOF
    {
      "CN": "kubernetes",
      "key": {
        "algo": "rsa",
        "size": 2048
      },
      "names": [
        {
          "C": "US",
          "L": "Raleigh",
          "O": "Kubernetes",
          "OU": "Kubernetes The Hard Way",
          "ST": "North Carolina"
        }
      ]
    }
    EOF
    ```
    
2. Set the kubernetes-the-hard-way public IP address as a variable 
    ```
    KUBERNETES_PUBLIC_ADDRESS=$(gcloud compute addresses describe kubernetes-the-hard-way \
      --region $(gcloud config get-value compute/region) \
      --format 'value(address)')
    ```
    
    You can verify this worked by checking the value of the variable.
    ```
    echo $KUBERNETES_PUBLIC_ADDRESS
    35.194.79.116
    ```
    
3. Set all of the valid internal hostnames for kubernetes to a variable. The Kubernetes API server is automatically assigned the kubernetes internal dns name, which will be linked to the first IP address (10.32.0.1) from the address range (10.32.0.0/24) reserved for internal cluster services during the control plane bootstrapping lab.
    ```
    KUBERNETES_HOSTNAMES=kubernetes,kubernetes.default,kubernetes.default.svc,kubernetes.default.svc.cluster,kubernetes.svc.cluster.local
    ```
    
4. Generate the signed certificates and key file
    ```
    cfssl gencert \
      -ca=ca.pem \
      -ca-key=ca-key.pem \
      -config=ca-config.json \
      -hostname=10.32.0.1,10.240.0.10,10.240.0.11,10.240.0.12,${KUBERNETES_PUBLIC_ADDRESS},127.0.0.1,${KUBERNETES_HOSTNAMES} \
      -profile=kubernetes \
      kubernetes-csr.json | cfssljson -bare kubernetes
    ```
    
5. Verify the certificate was created properly. Be sure that your alternative name values are correct, or it could cause problems later.
    ```
    openssl x509 -text -noout -in kubernetes.pem
    Certificate:
        Data:
            Version: 3 (0x2)
            Serial Number:
            ...
               X509v3 Subject Alternative Name:
                    DNS:kubernetes, DNS:kubernetes.default, DNS:kubernetes.default.svc, DNS:kubernetes.default.svc.cluster, DNS:kubernetes.svc.cluster.local, IP Address:10.32.0.1, IP Address:10.240.0.10, IP Address:10.240.0.11, IP Address:10.240.0.12, IP Address:35.194.79.116, IP Address:127.0.0.1        
    ```
    
#### Generated files 

* `kubernetes-key.pem`
* `kubernetes.pem`

### Creating the service account key pair

The `kubernetes-controller-manager` service uses a key pair to generate and sign service account tokens as described in the [managing service accounts](https://kubernetes.io/docs/admin/service-accounts-admin/) documentation.

1. Generate the service account CSR file
    ```
    cat > service-account-csr.json <<EOF
    {
      "CN": "service-accounts",
      "key": {
        "algo": "rsa",
        "size": 2048
      },
      "names": [
        {
          "C": "US",
          "L": "Portland",
          "O": "Kubernetes",
          "OU": "Kubernetes The Hard Way",
          "ST": "Oregon"
        }
      ]
    }
    EOF
    ```

2. Generate the signed certificate and key file 
    ```
    cfssl gencert \
      -ca=ca.pem \
      -ca-key=ca-key.pem \
      -config=ca-config.json \
      -profile=kubernetes \
      service-account-csr.json | cfssljson -bare service-account
    ```
    
3. Verify the service account key pair was created properly.
    ```
    openssl x509 -text -noout -in service-account.pem
    
    Certificate:
        Data:
            Version: 3 (0x2)
            Serial Number:
            ...
    ```
    
#### Generated files 

* `service-account-key.pem`
* `service-account.pem`

That is the last of the 9 certificates and key files you'll need in your kubernetes cluster. In the next section, you'll place them in the proper locations on the proper servers.

### Distributing client and server certificates 

1. Distribute the application node certificates and key files
    ```
    for instance in worker-0 worker-1 worker-2; do
      gcloud compute scp ca.pem ${instance}-key.pem ${instance}.pem ${instance}:~/
    done
    ```
    
    output:
    ```
    Warning: Permanently added 'compute.1224626097461632392' (ECDSA) to the list of known hosts.
    ca.pem                                                                                                                                                                                                                     100% 1338   101.0KB/s   00:00
    worker-0-key.pem                                                                                                                                                                                                           100% 1679   125.5KB/s   00:00
    worker-0.pem                                                                                                                                                                                                               100% 1505   112.9KB/s   00:00
    Warning: Permanently added 'compute.8234720453539634566' (ECDSA) to the list of known hosts.
    ca.pem                                                                                                                                                                                                                     100% 1338    99.8KB/s   00:00
    worker-1-key.pem                                                                                                                                                                                                           100% 1675   124.7KB/s   00:00
    worker-1.pem                                                                                                                                                                                                               100% 1505   111.7KB/s   00:00
    Warning: Permanently added 'compute.4717499042523749764' (ECDSA) to the list of known hosts.
    ca.pem                                                                                                                                                                                                                     100% 1338    96.4KB/s   00:00
    worker-2-key.pem                                                                                                                                                                                                           100% 1679   125.3KB/s   00:00
    worker-2.pem
    ```
2. Distribute the certificates and key files for control plane services. In the next lab you'll use these certificates to create client configuration files. 
    ```
    for instance in controller-0 controller-1 controller-2; do
      gcloud compute scp ca.pem ca-key.pem kubernetes-key.pem kubernetes.pem \
      service-account-key.pem service-account.pem ${instance}:~/
    done
    ```
    
    output (*note that controller-0 is already in your known_hosts file*):
    ```
    ca.pem                                                                                                                                                                                                                     100% 1338    98.7KB/s   00:00
    ca-key.pem                                                                                                                                                                                                                 100% 1679   124.1KB/s   00:00
    kubernetes-key.pem                                                                                                                                                                                                         100% 1679   124.4KB/s   00:00
    kubernetes.pem                                                                                                                                                                                                             100% 1684   125.0KB/s   00:00
    service-account-key.pem                                                                                                                                                                                                    100% 1679   124.0KB/s   00:00
    service-account.pem                                                                                                                                                                                                        100% 1448   108.6KB/s   00:00
    Warning: Permanently added 'compute.557026018878004704' (ECDSA) to the list of known hosts.
    ca.pem                                                                                                                                                                                                                     100% 1338   101.5KB/s   00:00
    ca-key.pem                                                                                                                                                                                                                 100% 1679   129.4KB/s   00:00
    kubernetes-key.pem                                                                                                                                                                                                         100% 1679   128.0KB/s   00:00
    kubernetes.pem                                                                                                                                                                                                             100% 1684   129.4KB/s   00:00
    service-account-key.pem                                                                                                                                                                                                    100% 1679   128.3KB/s   00:00
    service-account.pem                                                                                                                                                                                                        100% 1448   110.2KB/s   00:00
    Warning: Permanently added 'compute.1767130885567520254' (ECDSA) to the list of known hosts.
    ca.pem                                                                                                                                                                                                                     100% 1338    99.4KB/s   00:00
    ca-key.pem                                                                                                                                                                                                                 100% 1679   126.2KB/s   00:00
    kubernetes-key.pem                                                                                                                                                                                                         100% 1679   125.5KB/s   00:00
    kubernetes.pem                                                                                                                                                                                                             100% 1684   126.8KB/s   00:00
    service-account-key.pem                                                                                                                                                                                                    100% 1679   125.9KB/s   00:00
    service-account.pem
    ```

## Wrap-Up

It may feel a little odd to be talking about kubernetes so much while you haven't even downloaded the source code from Github yyout. Don't worry. You'll get there. Think about all of this prep work the next time you use a kube cluster. There's a massive amount of prep work that is normally automated during a kube install. 

Hopefully this gives you a little more appreciation for that process, as well as a little better understanding of it. 

If you have any questions, feel free to reach out to jduncan on the [kubernetes slack](https://kubernetes.slack.com/archives/DPXNHT65T) or [twitter](https://twitter.com/jamieeduncan). 

**Next up in the series is Session Two - https://hackmd.io/RIEmIlXpRouNlVKhBQ3Thw**
