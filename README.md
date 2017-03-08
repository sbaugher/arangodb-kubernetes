ArangoDB on kubernetes
======================

This is experimental.

Use like this:

    ./makeArangoDBKube.sh

and deploy the resulting file to kubernetes as in:

    kubectl create -f arangodb_cluster.yaml

No UI scaling so far, but you can scale up the arangodb-coordinator
and arangodb-dbserver deployments. The arangodb-coordinator deployment
can safely be scaled down.

See also

    ./makeArangoDBKube.sh --help
