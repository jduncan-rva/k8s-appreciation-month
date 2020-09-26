# Lab Session One

## Persistent Notes

* GDoc (internal to Google) - [go/k8s-appreciation-month](go/k8s-appreciation-month) 

## Weclome!

In our first session we'll be covering these labs from [Kubernetes The Hard Way](https://github.com/kelseyhightower/kubernetes-the-hard-way). 

* [Prerequisites](https://github.com/kelseyhightower/kubernetes-the-hard-way/blob/master/docs/01-prerequisites.md)
* [Installing the Client Tools](https://github.com/kelseyhightower/kubernetes-the-hard-way/blob/master/docs/02-client-tools.md)
* [Provisioning Compute Resources](https://github.com/kelseyhightower/kubernetes-the-hard-way/blob/master/docs/03-compute-resources.md)
* [Provisioning the CA and Generating TLS Certificates](https://github.com/kelseyhightower/kubernetes-the-hard-way/blob/master/docs/04-certificate-authority.md)

## Prerequisites

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

### Session Handling

Throughout the labs for K8s-Apprecation-Month you'll be accessing multiple hosts simultaneously to configure your Kubernetes cluster. A multiplexing tool like [`tmux`](https://github.com/tmux/tmux/wiki). There are many ways to install `tmux` on to Linux and MacOS systems. There's probably even a version that works with WSL on Windows 10, but that's far outside of our scope for this lab of labs.

If you don't want to use `tmux`, most modern terminal emulators can do multiple panes or tabs. Use whatever you're comfortable with.

With `gcloud` configured and your session management preferences ready to go, Lab 1 is complete we're ready to get some additional client tooling ready.

## Client Tools



## Provisioning Compute

## Certificate Authority and TLS Certificates

