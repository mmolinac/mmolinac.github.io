---
layout: post
status: publish
published: true
title: Using Ceph RBD as Container Storage Interface in Proxmox
author: Manuel Molina
about_author: "/us/manuel/readme.html"
author_email: mmc@pocosmhz.org
date: '2025-07-02 22:50:00 +0200'
tags:
- hypervisor
- home
- ha
- budget
- kubernetes
comments: true
---
This is the third post about Proxmox, after my [previous post]({% post_url 2025-04-15-proxmox-home-cluster-ii %}) in which I discussed the basic steps for Ceph installation.

In that, I mentioned about using CephFS. However, right here we're going to implement persistent storage through Ceph [RBD](https://docs.ceph.com/en/reef/rbd/)

I'll use both the documentation about [block devices and Kubernetes](https://docs.ceph.com/en/reef/rbd/rbd-kubernetes/) from Ceph and [this](https://fabreur.medium.com/kubernetes-using-ceph-rbd-as-container-storage-interface-csi-6ab4177a0fc3) Medium post from [Fabio Reis](https://fabreur.medium.com).

# Storage pool creation
We already have a RBD pool named `pool1` in which we allocate storage volumes for Proxmox VMs. 

{% highlight bash %}
root@pve01:~# ceph osd pool ls detail
pool 1 '.mgr' replicated size 3 min_size 2 crush_rule 0 object_hash rjenkins pg_num 1 pgp_num 1 autoscale_mode on last_change 19 flags hashpspool stripe_width 0 pg_num_max 32 pg_num_min 1 application mgr read_balance_score 3.00
pool 2 'pool1' replicated size 3 min_size 2 crush_rule 0 object_hash rjenkins pg_num 32 pgp_num 32 autoscale_mode on last_change 479 lfor 0/479/477 flags hashpspool,selfmanaged_snaps stripe_width 0 application rbd read_balance_score 1.31
{% endhighlight %}

Thus, we're going to create a separate one for our Kubernetes cluster.

From the same shell in a Proxmox cluster node, let's create a new pool:

{% highlight bash %}
root@pve01:~# ceph osd pool create kubernetes
pool 'kubernetes' created
{% endhighlight %}

We need to initialize a newly created pool prior to use:

{% highlight bash %}
root@pve01:~# rbd pool init kubernetes
{% endhighlight %}

We need to create a new user for Kubernetes and ceph-csi. Execute the following and **record the generated key**:

{% highlight bash %}
root@pve01:~# ceph auth get-or-create client.kubernetes mon 'profile rbd' osd 'profile rbd pool=kubernetes' mgr 'profile rbd pool=kubernetes'
[client.kubernetes]
    key = AQD9o0Fd6hQRChAAt7fMaSZXduT3NWEqylNpmg==
{% endhighlight %}

For configuring `ceph-csi`, we also require a ConfigMap object stored in Kubernetes to define the the Ceph monitor addresses for the Ceph cluster. Collect both the Ceph cluster unique **fsid** and the **monitor addresses**:

{% highlight bash %}
root@pve01:~# ceph mon dump
epoch 3
fsid b9127830-b0cc-4e34-aa47-9d1a2e9949a8
last_changed 2025-04-16T01:29:22.792421+0200
created 2025-04-15T20:21:39.833395+0200
min_mon_release 19 (squid)
election_strategy: 1
0: [v2:192.168.18.131:3300/0,v1:192.168.18.131:6789/0] mon.pve01
1: [v2:192.168.18.132:3300/0,v1:192.168.18.132:6789/0] mon.pve02
2: [v2:192.168.18.133:3300/0,v1:192.168.18.133:6789/0] mon.pve03
dumped monmap epoch 3
{% endhighlight %}

**Note**: `ceph-csi` currently only supports the legacy V1 protocol. Hence, we'll use the v1 addresses with port 6789.

# Kubernetes connection to Ceph storage
With all this information, we can now continue with the Ceph document from [this point](https://docs.ceph.com/en/latest/rbd/rbd-kubernetes/#generate-ceph-csi-configmap) in our Kubernetes cluster, and create the required assets manually.

Or better than that, we can use the [ceph-csi-rbd Helm chart](https://artifacthub.io/packages/helm/ceph-csi/ceph-csi-rbd) together with Terraform and apply something like this:

`variables.tf`:
{% highlight HCL %}
variable "k8s_clusters" {
  description = "Kubernetes clusters configuration"
  type = map(object({
    nodes = map(object({
      ip_address = string
      ip_gateway = string
    }))
    ceph = object({
      username      = string
      key           = string
      mon_hosts     = list(string)
      cluster_fsid  = string
      rbd_pool      = string
    })
  }))
  default = {
    k8s01 = {
      nodes = {
        k8s01cp01 = {
          ip_address = "192.168.1.5"
          ip_gateway = "192.168.1.1"
        }
        k8s01cp02 = {
          ip_address = "192.168.1.6"
          ip_gateway = "192.168.1.1"
        }
      }
      ceph = {
        # we omit the client. prefix !
        username = "kubernetes"
        key          = "AQD9o0Fd6hQRChAAt7fMaSZXduT3NWEqylNpmg=="
        mon_hosts    = [
            "192.168.18.131:6789",
            "192.168.18.132:6789",
            "192.168.18.133:6789"
        ]
        cluster_fsid = "b9127830-b0cc-4e34-aa47-9d1a2e9949a8"
        rbd_pool    = "kubernetes"
      }
    }
  }
}
{% endhighlight %}

`k8s_storage.tf`:
{% highlight HCL %}
resource "kubernetes_namespace" "ceph_csi_rbd" {
  metadata {
    name = "ceph-csi-rbd"
  }
}

resource "helm_release" "ceph_csi_rbd" {
  name       = "ceph-csi-rbd"
  repository = "https://ceph.github.io/csi-charts"
  chart      = "ceph-csi-rbd"
  version    = "3.14.1"
  namespace = kubernetes_namespace.ceph_csi_rbd.metadata[0].name
  values = [
    templatefile("${path.module}/source/helm/ceph/ceph-csi-rbd-values.tpl.yml", {
      ceph_conf = var.k8s_clusters["k8s01"].ceph
    })
  ]
}
{% endhighlight %}

The aforementioned `ceph-csi-rbd-values.tpl.yml` is the default values template you can get from [here](https://artifacthub.io/packages/helm/ceph-csi/ceph-csi-rbd?modal=values) on that Helm chart.

The keys I've customized are:

- `csiConfig`
- `storageClass`
- `secret`

With that, you apply the chart and all resources cited in Ceph docs are created.

# Kubernetes storage test
Let's create a storage volume from inside our Kubernetes cluster:

{% highlight bash %}
root@k8s01cp01:~# cat raw-block-pvc.yaml 
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: raw-block-pvc
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Block
  resources:
    requests:
      storage: 1Gi
  storageClassName: csi-rbd-sc

root@k8s01cp01:~# kubectl apply -f raw-block-pvc.yaml
persistentvolumeclaim/raw-block-pvc created

root@k8s01cp01:~# kubectl get pvc
NAME            STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
raw-block-pvc   Bound    pvc-18ef4c64-caaf-44e5-a123-6502238a2a1e   1Gi        RWO            csi-rbd-sc     <unset>                 19s
{% endhighlight %}

and now we see the volume inside Ceph:

{% highlight bash %}
root@pve01:~# rbd ls -p kubernetes
csi-vol-e5fa9aa4-13b9-4f09-a366-91621a34c264
{% endhighlight %}

Now, if we remove the volume from Kubernetes:

{% highlight bash %}
root@k8s01cp01:~# kubectl delete pvc/raw-block-pvc
persistentvolumeclaim "raw-block-pvc" deleted

{% endhighlight %}

... we can confirm that Ceph does not have that anymore:

{% highlight bash %}
root@pve01:~# rbd ls -p kubernetes
root@pve01:~# 
{% endhighlight %}
