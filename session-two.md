# Lab Session Two

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

In our second livestream lab, we'll cover the following KTHW labs: 

* [Generating Kubernetes Configuration Files for Authentication](https://github.com/kelseyhightower/kubernetes-the-hard-way/blob/master/docs/05-kubernetes-configuration-files.md)
* [Generating the Data Encryption Config and Key](https://github.com/kelseyhightower/kubernetes-the-hard-way/blob/master/docs/06-data-encryption-keys.md)
* [Bootstrapping the etcd Cluster](https://github.com/kelseyhightower/kubernetes-the-hard-way/blob/master/docs/07-bootstrapping-etcd.md)

## Generating Kubernetes Configuration Files

The goal of this session is to take the certificates and keys generated in the previous lab and create the [configuration files](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/) (kubeconfigs) needed for your cluster.

*Note - these commands should be run in the same directory where you created the TLS certificates and key files.*

### Getting your Kubernetes Public IP Address

Each kubeconfig you create needs to reference the IP address of the kubernetes API server. To make your control plane highly available, the IP address will be the public IP address you created earlier. You'll retrieve this value using the `gcloud` SDK.

```
KUBERNETES_PUBLIC_ADDRESS=$(gcloud compute addresses describe kubernetes-the-hard-way \
  --region $(gcloud config get-value compute/region) \
  --format 'value(address)')
```

To verify this worked, check the value of the variable.

```
echo $KUBERNETES_PUBLIC_ADDRESS
```

### kubelet

Each node requries a kubeconfig for the `kubelet` service. Like we discussed in the previous lab, the [Node Authorizer](https://kubernetes.io/docs/admin/authorization/node/) code in kubernetes references each node by its hostname when its authorized. This command creates kubeconfigs for each application node. 

```
for instance in worker-0 worker-1 worker-2; do
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://${KUBERNETES_PUBLIC_ADDRESS}:6443 \
    --kubeconfig=${instance}.kubeconfig

  kubectl config set-credentials system:node:${instance} \
    --client-certificate=${instance}.pem \
    --client-key=${instance}-key.pem \
    --embed-certs=true \
    --kubeconfig=${instance}.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:node:${instance} \
    --kubeconfig=${instance}.kubeconfig

  kubectl config use-context default --kubeconfig=${instance}.kubeconfig
done
```

#### Created Files 

* `worker-0.kubeconfig`
* `worker-1.kubeconfig`
* `worker-2.kubeconfig`

### kube-proxy

This is the kubeconfig for the `kube-proxy` control plane service. Run the following `kubectl` commands:

```
kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://${KUBERNETES_PUBLIC_ADDRESS}:6443 \
    --kubeconfig=kube-proxy.kubeconfig

kubectl config set-credentials system:kube-proxy \
    --client-certificate=kube-proxy.pem \
    --client-key=kube-proxy-key.pem \
    --embed-certs=true \
    --kubeconfig=kube-proxy.kubeconfig
    
kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:kube-proxy \
    --kubeconfig=kube-proxy.kubeconfig

kubectl config use-context default --kubeconfig=kube-proxy.kubeconfig
```

#### Created File

* `kube-proxy.kubeconfig`

### kube-controller-manager

This is the kubeconfig for the `kube-controller-manager` control plane service. Run the following `kubectl` commands: 

```
kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=kube-controller-manager.kubeconfig

kubectl config set-credentials system:kube-controller-manager \
    --client-certificate=kube-controller-manager.pem \
    --client-key=kube-controller-manager-key.pem \
    --embed-certs=true \
    --kubeconfig=kube-controller-manager.kubeconfig

kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:kube-controller-manager \
    --kubeconfig=kube-controller-manager.kubeconfig

kubectl config use-context default --kubeconfig=kube-controller-manager.kubeconfig
```

#### Created File

* `kube-controller-manager.kubeconfig`

### kube-scheduler

This is the kubeconfig for the `kube-scheduler` control plane service. Run the following `kubectl` commands:

```
kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=kube-scheduler.kubeconfig

kubectl config set-credentials system:kube-scheduler \
    --client-certificate=kube-scheduler.pem \
    --client-key=kube-scheduler-key.pem \
    --embed-certs=true \
    --kubeconfig=kube-scheduler.kubeconfig

kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:kube-scheduler \
    --kubeconfig=kube-scheduler.kubeconfig

kubectl config use-context default --kubeconfig=kube-scheduler.kubeconfig
```

#### Created File

* `kube-scheduler.kubeconfig`

### admin user

This is the kubeconfig for the `admin` user. 

```
kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=admin.kubeconfig

kubectl config set-credentials admin \
    --client-certificate=admin.pem \
    --client-key=admin-key.pem \
    --embed-certs=true \
    --kubeconfig=admin.kubeconfig

kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=admin \
    --kubeconfig=admin.kubeconfig

kubectl config use-context default --kubeconfig=admin.kubeconfig
```

#### Created file 

* `admin.kubeconfig`

### Data Encrycption Configuration and Key

The last configuration file you'll create in this lab will help encrypt data at rest in your kubernetes cluster. It leverages a base-64 encoded random string. Run the following commands: 

```
ENCRYPTION_KEY=$(head -c 32 /dev/urandom | base64)

cat > encryption-config.yaml <<EOF
kind: EncryptionConfig
apiVersion: v1
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: ${ENCRYPTION_KEY}
      - identity: {}
EOF
```

#### Created file

* `encryption-config.yaml`

### Distributing the Created Files

1. Copy the `kubelet` and `kube-proxy` kubeconfigs to each application node.

    ```
    for instance in worker-0 worker-1 worker-2; do
      gcloud compute scp ${instance}.kubeconfig kube-proxy.kubeconfig encryption-config.yaml ${instance}:~/
    done
    ```
    
2. Copy the control plane kubeconfigs to the control plane nodes.
    ```
    for instance in controller-0 controller-1 controller-2; do
      gcloud compute scp admin.kubeconfig kube-controller-manager.kubeconfig kube-scheduler.kubeconfig ${instance}:~/
    done
    ```
    
## Bootstrapping the etcd Cluster 

Kubernetes stores all of its state information in [etcd](https://github.com/etcd-io/etcd). You'll bootstrap a 3-node `etcd` cluster on your control plane, configuring it to be highly available with secure remote access.

### Prerequisites

All of these commands must be run on all three control plane nodes. Your `ssh` keys are stored in the `gcloud` SDK. For example, you can connect to `controller-0` by typing: 

```
gcloud compute ssh controller-0
```

### Setting up tmux

`tmux` is an effective tool to, among other things, execute the same command on multiple servers in parrallel. It'll save you a lot of time and prevent a lot of typos during the rest of this lab. Your environment my vary a little, but the general steps should be the same. If you choose to use another method to handle multiple servers, ignore this section. If you're using [GCP Cloud Shell](https://cloud.google.com/shell) to deploy this lab, `tmux` is already installed. 

#### Open up three tmux panes 

1. The hot key for `tmux` is typically `ctrl-b`. To open up a new tab in `tmux`, hit `ctrl-b` followed by the double-quote key. This will open up a new pane. Do this twice to open up a total of 3 panes. 
2. At this point you have 3 panes open, but they're not evenly spaced. That's just annoying. Hit `ctrl-b` again, followed by `:select-layout even-vertical` and hit `Enter`. 

You should now have 3 nice evenly spaced panes.

#### Connect to your control plane nodes 

To move between panes to make the initial connections, use `ctrl-b` and the up/down errors. 
1. If you're using Cloud Shell, you will have to run `gcloud init` in each `tmux` pane to connect back to your account.
2. With this done, connect to one of your control plane nodes in each pane. For example: 
    ```
    gcloud compute ssh controller-0
    ```

Once you're connected to all 3 control plane nodes, you can synchronize your panes to type in all of them simultaneously.

#### Synchronizing your tmux panes 

1. Hit `ctrl-b` on your keyboard, followed by `:setw synchronize-panes on`.

This will synchronize the 3 `tmux` panes to enter the same commands across all 3 control plane nodes simultaneously. Next, you'll bootstrap your `etcd` cluster.

### Configuring each cluster

1. Download the `etcd` binaries from the [Github](https://github.com/etcd-io/etcd) project.
    ```
    wget -q --show-progress --https-only --timestamping \
      "https://github.com/etcd-io/etcd/releases/download/v3.4.10/etcd-v3.4.10-linux-amd64.tar.gz"
    ```
    
2. Extract and install the `etcd` and `etcdctl` binaries. `etcdctl` is the command line utility to manage an `etcd` deployment.
    ```
    tar -xvf etcd-v3.4.10-linux-amd64.tar.gz
    sudo mv etcd-v3.4.10-linux-amd64/etcd* /usr/local/bin/
    ```
    
3. Apply etcd configurations
    ```
    sudo mkdir -p /etc/etcd /var/lib/etcd
    sudo chmod 700 /var/lib/etcd
    sudo cp ca.pem kubernetes-key.pem kubernetes.pem /etc/etcd/
    ```
    
4. Get the host internal IP address 
    ```
    INTERNAL_IP=$(curl -s -H "Metadata-Flavor: Google" \
      http://metadata.google.internal/computeMetadata/v1/instance/network-interfaces/0/ip)
    ```
    
5. Each host needs a unique name for `etcd`. Save this value as a variable.
    ```
    ETCD_NAME=$(hostname -s)
    ```
    
6. Create the `etcd.service` systemd unit file.
    ```
    cat <<EOF | sudo tee /etc/systemd/system/etcd.service
    [Unit]
    Description=etcd
    Documentation=https://github.com/coreos

    [Service]
    Type=notify
    ExecStart=/usr/local/bin/etcd \\
      --name ${ETCD_NAME} \\
      --cert-file=/etc/etcd/kubernetes.pem \\
      --key-file=/etc/etcd/kubernetes-key.pem \\
      --peer-cert-file=/etc/etcd/kubernetes.pem \\
      --peer-key-file=/etc/etcd/kubernetes-key.pem \\
      --trusted-ca-file=/etc/etcd/ca.pem \\
      --peer-trusted-ca-file=/etc/etcd/ca.pem \\
      --peer-client-cert-auth \\
      --client-cert-auth \\
      --initial-advertise-peer-urls https://${INTERNAL_IP}:2380 \\
      --listen-peer-urls https://${INTERNAL_IP}:2380 \\
      --listen-client-urls https://${INTERNAL_IP}:2379,https://127.0.0.1:2379 \\
      --advertise-client-urls https://${INTERNAL_IP}:2379 \\
      --initial-cluster-token etcd-cluster-0 \\
      --initial-cluster controller-0=https://10.240.0.10:2380,controller-1=https://10.240.0.11:2380,controller-2=https://10.240.0.12:2380 \\
      --initial-cluster-state new \\
      --data-dir=/var/lib/etcd
    Restart=on-failure
    RestartSec=5

    [Install]
    WantedBy=multi-user.target
    EOF
    ```
    
7. Start the `etcd` service 
    ```
    sudo systemctl daemon-reload
    sudo systemctl enable etcd
    sudo systemctl start etcd
    ```
    
8. Verify `etcd` is running properly
    ```
    sudo ETCDCTL_API=3 etcdctl member list \
      --endpoints=https://127.0.0.1:2379 \
      --cacert=/etc/etcd/ca.pem \
      --cert=/etc/etcd/kubernetes.pem \
      --key=/etc/etcd/kubernetes-key.pem
    ```
    
    sample output:
    ```
    3a57933972cb5131, started, controller-2, https://10.240.0.12:2380, https://10.240.0.12:2379, false
    f98dc20bce6225a0, started, controller-0, https://10.240.0.10:2380, https://10.240.0.10:2379, false
    ffed16798470cab5, started, controller-1, https://10.240.0.11:2380, https://10.240.0.11:2379, false
    ```
    
### Wrap Up

You now have a functional `etcd` cluster! In the next session you'll bootstrap the control plane and application node services and configure `kubectl` to communicate with your kubernetes API.