ArangoDB on Kubernetes
======================

This is experimental.  The ultimate goal of this project is to allow for fully automatic scaling of ArangoDB in Kubernetes, using StatefulSets.  Pull requests are highly encouraged.

To give it a try, just run:

    ./makeArangoDBKube.sh

and deploy the resulting file to kubernetes like so:

    kubectl create -f arangodb_cluster.yaml

Fully automatic scaling is not supported yet, but you can scale-up the arangodb-coordinator
and arangodb-dbserver deployments. The arangodb-coordinator deployment
can safely be scaled down.

See also

    ./makeArangoDBKube.sh --help

Kubernetes Quick Start
======================

There are many tutorials out there on how to get Kubernetes up and running on local hardware, but the method I used, and I found works best, comes straight from the source:  https://kubernetes.io/docs/getting-started-guides/kubeadm/

My local Kubernetes environment consists of one bare metal server, running nine Ubuntu 16.04 LTS VMs under VirtualBox.  I have one master, five "compute" slaves, and three "storage" slaves (I'll talk more about those below).  The installation of Kubernetes using the instructions linked to above works pretty flawlessly.

Using GlusterFS as Persistent Volume Storage on Kubernetes
==========================================================

As with any database, the important part of the equation is the data.  You don't want to lose your data just because the pod running your ArangoDB instance(s) goes down.  That problem is rather easily solved by mounting a persistent volume into the pod at /var/lib/arangodb3.  Most people, at least for development/testing purposes, use host volumes (i.e. the underlying hard disk of the pod host).

The problem comes when the host itself goes down (i.e. a hardware failure).  While Kubernetes is more than happy to reschedule your pod on a new host, your data is stuck on the failed host.  This can be solved by storing your data on a resilient file system, like GlusterFS (or similar systems offered by cloud providers).  Getting GlusterFS set-up and integrated with Kubernetes is not a trivial task, fortunately, someone else has done most of the work for you:  https://github.com/gluster/gluster-kubernetes

While the script works fairly well, there are a couple of caveats to be aware of before running the script:

1.  All of the nodes on your Kubernetes cluster need to have the GlusterFS client installed.
2.  You need to modprobe dm_thin_pool (or be sure it is loaded) on all of the nodes that will host GlusterFS.
3.  You need to initialize the physical volumes on each GlusterFS node by running "pvcreate /dev/sdX -ff", where X is the block device you are going to let GlusterFS use for storage.  Beware that GlusterFS will destroy any existing data on that disk!

If the script successfully executes, you should provided with a URL to use as the Heketi REST API.  You'll need that to create a storage class to auto-provision volumes on GlusterFS as needed.  Here's a link to a more manual method of installing GlusterFS (they use a lot of the same yaml files), with some troubleshooting ideas as well: http://blog.lwolf.org/post/how-i-deployed-glusterfs-cluster-to-kubernetes/

At the end of the above blog post, you'll see a section on creating a storage class.  Use that example, substituting your actual Heketi REST API URL.

The last thing you (optionally) need to do is mark your new storage class as the default.  Now when a pod makes a persistent volume claim, the creation of the persistent volume will happen automagically in the background, and it will be correctly bound to the pod.

Stateful Sets in Kubernetes
===========================

Now that we have the foundation properly laid, it'd be really nice to use StatefulSets with ArangoDB, so that if for example, a database server is rescheduled to a different pod, it has access to the same persistent volume as before.  That will be the focus of this project over the next few weeks.
