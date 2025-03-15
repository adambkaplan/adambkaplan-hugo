---
title: "Plex on MicroShift"
date: 2022-02-16T09:06:15-05:00
draft: false
---

How I deployed Plex Media Server on Microshift!

----
<!--more-->

## Plex on MicroShift

TL;DR if you want to deploy this yourself!

1. Follow the [Getting Started](https://microshift.io/docs/getting-started/) guide to install
   MicroShift on Fedora (34+), CentOS 8 Stream, or RHEL 8.
2. Install Metallb using the [official helm chart](https://metallb.universe.tf/installation/#installation-with-helm),
   reserving a set of IP addresses on the local network:

   ```yaml
   # values.yaml
   configInline:
     address-pools:
       - name: default
         protocol: layer2
         addresses:
           - <MY_RESERVED_IPS>
   psp:
     create: false
   ```

3. Install the NFS CSI Driver using the [official helm chart](https://github.com/kubernetes-csi/csi-driver-nfs/tree/master/charts#install-csi-driver-with-helm-3):

   ```yaml
   # values.yaml
   controller:
     replicas: 1
   ```

4. Configure a PersistentVolume and PersistentVolumeClaim for Plex's media. I had previously set
   this up to sort/organize my media from various devices through the years:

   ```yaml
   apiVersion: v1
   kind: PersistentVolume
   metadata:
     name: plex-data
     labels:
       app: plex
   spec:
     capacity:
     storage: 196Gi
     accessModes:
       - ReadWriteMany
     persistentVolumeReclaimPolicy: Retain
     mountOptions:
       - hard
       - nfsvers=4.1
     csi:
       driver: nfs.csi.k8s.io
       readOnly: false
       volumeHandle: <MY_VOLUME_HANDLE>
       volumeAttributes:
         server: <nfs-hostname>
         share: <nfs-share-path>
   ```

   ```yaml
   apiVersion: v1
   kind: PersistentVolumeClaim
   metadata:
     labels:
       app: plex
     name: plex-data
   spec:
     accessModes:
       - ReadWriteMany
     resources:
       requests:
         storage: 196Gi
     volumeName: plex-data
   ```

5. Deploy Plex Media Server using the [k8s@home helm chart](https://artifacthub.io/packages/helm/k8s-at-home/plex),
   configuring the following values:

   1. The local time zone
   2. A reserved IP address on my home network
   3. A provisioned volume for Plex's database
   4. My NFS PVC for Plex's media

   ```yaml
   env:
     TZ: "America/New_York"
     ADVERTISE_IP: "http://<RESERVED_IP_ADDRESS>:32400/"
   resources:
     requests:
       cpu: 200m
       memory: 256Mi
     limits:
       memory: 2048Mi
   persistence:
     # config directory is the database, we want this to persist
     # Use the hostpath provisioner with microshift
     config:
       enabled: true
       type: pvc
       accessMode: ReadWriteOnce
       size: 10Gi
       storageClass: kubevirt-hostpath-provisioner
     # data directory is where music, movies, photos are stored
     data:
       enabled: true
       type: pvc
       existingClaim: plex-data
   service:
     # service - for security reasons, we want to configure metallb loadbalancers
     main:
       type: LoadBalancer
       loadBalancerIP: <RESERVED_IP_ADDRESS>
   ```

## The Full Story

My homelab consists of an old Dell Inspiron laptop and a [Synology DS220+ NAS](https://www.synology.com/en-us/products/DS220+)
for file sharing.
Ever since I set it up, I wanted to deploy [OpenShift/OKD](https://www.okd.io/) to run applications.
OpenShift packs a lot of features on top of standard Kubernetes, and until recently running a small single node cluster was impossible.

The launch of [MicroShift](https://microshift.io/) changed this limitation, as it promises to
deliver an OpenShift-like experience in a form factor suitable for edge devices.
My old Inspiron certainly fits the description of an "edge device":

* Dual-core 2.2 GHz Intel Pentium (!) processor
* 4Gb RAM
* 16Gb HDD storage

Over the past week I decided to kick the tires and rebuild my home cluster with MicroShift!

### Setup and Install

The first thing I did was install Fedora 35 on the laptop.
I has previously set up [k3s](https://k3s.io/) with Fedora 32 on that machine, and Plex running on
top.
Over time the system deteriorated to the point that I couldn't log in via SSH.
Rather than do a typical upgrade, I did a clean install.

After updating the OS, it was time to install MicroShift.
The [Getting Started](https://microshift.io/docs/getting-started/) guide has very easy instructions
to get the cluster up and running.
An important thing I found is that on RHEL/Fedora, you really need to update the firewall settings.
Otherwise the pods in your cluster won't be able to access the host's DNS services (and thus, the
rest of the Internet).

### Networking

Folks who have used Kubernetes know that workloads run in Pods that have their own
[network space](https://kubernetes.io/docs/concepts/services-networking/) and IP address that is
separate from the host system's network.
To get network traffic into Kubernetes Pods, you need to tell Kubernetes how to route the network
packets.
This can be done in a few ways:

- Create a `NodePort` service, which opens a port on the host system.
- Create a `LoadBalancer` service, which relies on an external application to establish an IP
  address that accepts traffic and routes it to the specified Pods.
- Create an `Ingress`, which paired with a standard `ClusterIP` service allows HTTP/S traffic to
  be routed to pods via a domain name and/or path.

Which one to choose?

I decided to use a `LoadBalancer` service because Plex has a unique
[secure server connections](https://support.plex.tv/articles/206225077-how-to-use-secure-server-connections/) feature.
When you sign up for Plex and then register your server with the same account, Plex can establish
trusted communication over HTTPS between your server, local Plex TV apps, and even remote mobile
devices.
There is a bit of DNS hacking that makes this all work, and a key piece of the puzzle comes from Plex
Media Server sending its local network IP address to Plex's centralized servers.

But which IP address gets sent to Plex central?
Under normal circumstances, the local Plex Media Server will send the IP address of the network it
is deployed on.
As I mentioned earlier, when Plex is run on Kubernetes it will run in its own network IP space, and
therefore will send its cluster-internal IP address by default.
To send the local network's IP address, I set an appropriate value for the `ADVERTISE_IP`
environment variable.
This is documented in the official Plex Media Server
[container image documentation](https://hub.docker.com/r/plexinc/pms-docker/).

Using a `LoadBalancer` service allows me to fix the value `ADVERTISE_IP` for Plex.
When deploying the Plex helm chart, I can ask the load balancer to assign a specific local IP
address for Plex.
If I had chosen a `NodePort` service, I would have needed to know in advance the IP of the node
I'm deploying Plex.
For now I have only one node in my homelab, but I hope to add additional nodes in the future.

I use [Metallb](https://metallb.universe.tf) as the provider of my load balancer services.
Metallb is designed to provide load balancers for on-premises Kubernetes deployments.
I configured mine to use a [Layer 2](https://metallb.universe.tf/configuration/#layer-2-configuration)
address pool - it is the simplest to set up and works with my home network's router.

In theory I could have tried to provision an `Ingress` object, which I think would have provisioned
an OpenShift `Route` on Microshift.
I decided this was a bit of overkill, plus most of the documentation for Plex assumes the
application is accessed directly on user machines.
I didn't want to push the limits of what the `ADVERTISE_IP` environment variable actually does.

### Plex Database Storage

Plex is a media server which lets you access your music, movies, photos and more from your computer.
It also has integrations that let you stream content to Roku TVs as well as mobile devices (the
latter requires a subscription for full access). 
The containerized version of Plex Media Server can have three potential volumes for storage:

1. `/config` - to store Plex's database
2. `/transcode` - scratch space for media transcoding
3. `/data` - to store media

For my home deployment, I decided to use persistent local storage for `/config`, temporary disk
space for `/transcode`, and my Synology NAS for the media in `/data`.

#### Config Storage

MicroShift comes wit the `kubevirt-hostpath-provisioner`, a CSI driver which supports dynamic
provisioning of PersistentVolumes backed by local disk storage on the host node.
This is an ideal use case for my `config` volume, as there is only node on my cluster and its
network bandwidth is limited to 100MB/s.
Using networked storage for the database seemed like a risk for poor performance.
To use the provisioner, I specified the `kubevirt-hostpath-provisioner` as the StorageClass for
the `config` volume.

#### Data Storage

For the `/data` volume, I used my music and photos collection backed up on the Synology NAS.
NFS felt like the best way to configure this, since I also wanted to update the collection while
Plex was running.
My first attempt to set up a normal NFS `PersistentVolume` failed because MicroShift doesn't
know how to mount a volume with NFS!
This was a surprise to me, since NFS is a first-class volume source in Kubernetes.

I soon learned the reason why - MicroShift runs as a container with `ubi8-minimal` as its base image.
Today UBI only contains a subset of packages available in a typical RHEL system.
NFS is one such package that is excluded from UBI, and thus MicroShift can't mount NFS drives natively.

Fortunately there is a simple solution to this predicament.
The upstream Kubernetes SIG-Storage has created a [CSI driver](https://github.com/kubernetes-csi/csi-driver-nfs)
for provisioning NFS storage, using much of the same logic that allows Kubernetes to mount NFS volumes.
In the long run, CSI drivers will be the
[supported way to configure volumes in Kubernetes](https://kubernetes.io/blog/2021/12/10/storage-in-tree-to-csi-migration-status-update/).
The driver has a Helm chart that was very simple to set up - the only default I needed to change
was the number of controller replicas to deploy.

### Pulling It All Together

I deployed Metallb, the NFS CSI Driver, and Plex using the Helm charts referenced at the beginning
of this post, with the provided configuration values.
I had previously learned a lot about deploying Plex by writing an operator for my first deployment.
I decided to use Helm this go around for its simplicity - the operator is overkill for my needs,
and was more of an educational project for me than anything else.

After deploying Plex, there were a few "day 2" matters to deal with once I had access to the Plex
web console.
The first bit of business was to sign in with my Plex account and set up the libraries for my music
and photo collection.
Next, I claimed my server and linked it to my Plex account.
Finally, I set up the "Remote Access" feature so that I could access my media from anywhere -
perhaps with some lag, as I won't have a CDN to help with latency.

Next up:

1. Actually reorganize my music collection, which is sitting on various device backups.
2. Fix up any mislabeled tags on the various songs downloaded from the peak piracy era.
3. Try and minimize the privileges available to Plex, using PodSecurity namespace labels.
