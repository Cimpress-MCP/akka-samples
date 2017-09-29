# Trying to down an inaccessible node from the cluster

Start a cluster with multiple nodes:

    sbt "runMain sample.cluster.simple.SimpleClusterApp 2551"
    sbt "runMain sample.cluster.simple.SimpleClusterApp 2552"

Find the pid for the second node and kill it:

    ps aux | grep 2552
    kill -9 <pid>

Query the cluster status:
    
    curl localhost:19999/members

The cluster status should have one unreachable node:
````json
{
  "selfNode": "akka.tcp://ClusterSystem@127.0.0.1:2551",
  "leader": "akka.tcp://ClusterSystem@127.0.0.1:2551",
  "oldest": "akka.tcp://ClusterSystem@127.0.0.1:2551",
  "unreachable": [
    {
      "node": "akka.tcp://ClusterSystem@127.0.0.1:2552",
      "observedBy": [
        "akka.tcp://ClusterSystem@127.0.0.1:2551"
      ]
    }
  ],
  "members": [
    {
      "node": "akka.tcp://ClusterSystem@127.0.0.1:2551",
      "nodeUid": "-1097307584",
      "status": "Up",
      "roles": []
    },
    {
      "node": "akka.tcp://ClusterSystem@127.0.0.1:2552",
      "nodeUid": "-1898009298",
      "status": "Up",
      "roles": []
    }
  ]
}
````

Try to down the node:

    curl -X DELETE localhost:19999/members/akka.tcp://ClusterSystem@127.0.0.1:2552
    
Query again, and the node will appear as leaving, but it will not leave the cluster:
````json
{
  "selfNode": "akka.tcp://ClusterSystem@127.0.0.1:2551",
  "leader": "akka.tcp://ClusterSystem@127.0.0.1:2551",
  "oldest": "akka.tcp://ClusterSystem@127.0.0.1:2551",
  "unreachable": [
    {
      "node": "akka.tcp://ClusterSystem@127.0.0.1:2552",
      "observedBy": [
        "akka.tcp://ClusterSystem@127.0.0.1:2551"
      ]
    }
  ],
  "members": [
    {
      "node": "akka.tcp://ClusterSystem@127.0.0.1:2551",
      "nodeUid": "1952225788",
      "status": "Up",
      "roles": []
    },
    {
      "node": "akka.tcp://ClusterSystem@127.0.0.1:2552",
      "nodeUid": "-1309925996",
      "status": "Leaving",
      "roles": []
    }
  ]
}
````

Logs for this example can be found [here](DowningUnreachableNode.md)