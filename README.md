ArangoDB on kubernetes
======================

This is experimental.  The ultimate goal of this project is to allow for fully automatic scaling of ArangoDB in Kubernetes, using StatefulSets.  Pull requests are highly encouraged.

To give it a try, just run:

    ./makeArangoDBKube.sh

and deploy the resulting file to kubernetes like so:

    kubectl create -f arangodb_cluster.yaml

No scaling via the UI so far, but you can scale-up the arangodb-coordinator
and arangodb-dbserver deployments. The arangodb-coordinator deployment
can safely be scaled down.

See also

    ./makeArangoDBKube.sh --help
