# ArangoDB on Kubernetes

## New Project from the Mothership

As of March, 20, 2018, ArangoDB has released a Kubernetes project of their own.  It can be found at: https://github.com/arangodb/kube-arangodb  For those of you using this project, I'd encourage you to checkout their project.  While it is not yet production ready, it will inevitably surpass this project in terms of robustness and support.

## New Support for StatefulSets

There is now a new file `clusterv2.yaml`, that allows you to quickly set-up an ArangoDB cluster on Kubernetes.  This script should work with Kubernetes 1.5 and above.  If you can, I'd recommend using this method to get your cluster up and running.

## Old Method

To give it a try, just run:

    ./makeArangoDBKube.sh

and deploy the resulting file to kubernetes like so:

    kubectl create -f arangodb_cluster.yaml

Fully automatic scaling is not supported yet, but you can scale-up the arangodb-coordinator
and arangodb-dbserver deployments. The arangodb-coordinator deployment
can safely be scaled down.

See also

    ./makeArangoDBKube.sh --help

## Kubernetes Quick Start

There are many tutorials out there on how to get Kubernetes up and running on local hardware, but the method I used, and I found works best, comes straight from the source:  https://kubernetes.io/docs/getting-started-guides/kubeadm/

My local Kubernetes environment consists of one bare metal server, running nine Ubuntu 16.04 LTS VMs under VirtualBox.  I have one master, five "compute" slaves, and three "storage" slaves (I'll talk more about those below).  The installation of Kubernetes using the instructions linked to above works pretty flawlessly.

## Using GlusterFS as Persistent Volume Storage on Kubernetes

As with any database, the important part of the equation is the data.  You don't want to lose your data just because the pod running your ArangoDB instance(s) goes down.  That problem is rather easily solved by mounting a persistent volume into the pod at /var/lib/arangodb3.  Most people, at least for development/testing purposes, use host volumes (i.e. the underlying hard disk of the pod host).

The problem comes when the host itself goes down (i.e. a hardware failure).  While Kubernetes is more than happy to reschedule your pod on a new host, your data is stuck on the failed host.  This can be solved by storing your data on a resilient file system, like GlusterFS (or similar systems offered by cloud providers).  Getting GlusterFS set-up and integrated with Kubernetes is not a trivial task, fortunately, someone else has done most of the work for you:  https://github.com/gluster/gluster-kubernetes

While the script works fairly well, there are a couple of caveats to be aware of before running the script:

1.  All of the nodes on your Kubernetes cluster need to have the GlusterFS client installed.
2.  You need to `modprobe dm_thin_pool` (or be sure it is loaded) on all of the nodes that will host GlusterFS.
3.  You need to initialize the physical volumes on each GlusterFS node by running `pvcreate /dev/XXX -ff`, where XXX is the block device you are going to let GlusterFS use for storage.  Beware that GlusterFS will destroy any existing data on that disk!
4.  The script was built for OpenShift, but works fine on plain old Kubernetes.  You do need to make sure you tell it the namescape (probably "default"), otherwise it will fail right away.  My command line looked like: `./gk-deploy -vg topology.json -n default`

If the script successfully executes, you should provided with a URL for the Heketi REST API.  You'll need that to create a storage class to auto-provision volumes on GlusterFS as needed.  Here's a link to a more manual method of installing GlusterFS (they use a lot of the same yaml files), with some troubleshooting ideas as well: http://blog.lwolf.org/post/how-i-deployed-glusterfs-cluster-to-kubernetes/

At the end of the above blog post, you'll see a section on creating a storage class.  Use that example, substituting your actual Heketi REST API URL.

The last thing you (optionally) need to do is mark your new storage class as the default.  Now when a pod makes a persistent volume claim, the creation of the persistent volume will happen automagically in the background, and it will be correctly bound to the pod.

## Stateful Sets in Kubernetes

With the new script, StatefulSets are now fully supported.  You can use GlusterFS (see above) for persistent volume storage, or your favorite cloud provider's persistent storage mechanism.  Either way, every node in the cluster has it's own persistent storage, that will follow it around the cluster if/when it gets rescheduled on a different node.

You can scale your nodes down using Kubernetes, but be aware of two things:

1.  If you haven't set a replcation factor >= 2, you will lose data!
2.  The built-in ArangoDB cluster health will continue to show the node as down.

Both problems can be overcome by using the ArangoDB REST api to take a server offline first (to solve problem #1), and to remove a permanently downed server from the cluster (to solve problem #2).

The above two issues are next-up on the development roadmap for this project.
