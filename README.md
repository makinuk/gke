# RabbitMQ Setup on GKE


```
###Install the RabbitMQ operator 
kubectl apply -f https://github.com/rabbitmq/cluster-operator/releases/latest/download/cluster-operator.yml
###Check if the components are healthy in the rabbitmq-system namespace
kubectl get all -o wide -n rabbitmq-system
```

Once the rabbitmq-cluster-operator is healthy and all the related components are created let us now proceed with the installation of a RabbitMQ Cluster in our Kubernetes.

```
kubectl apply -f rabbitmqcluster.yaml 
```

Let us now check the status of our RabbitmqCluster.

```
kubectl describe RabbitmqCluster production-rabbitmqcluster 
```

Now, in the above RabbitMQ Cluster manifest, we have specified the default username and password to be guest/guest. However, the RabbiMQ Cluster operator also creates a Kubernetes secret for us so that the username and the password can be extracted from there as well.

```
###Username
kubectl get secret production-rabbitmqcluster-default-user -o jsonpath='{.data.username}' | base64 --decode
###Password
kubectl get secret production-rabbitmqcluster-default-user -o jsonpath='{.data.password}' | base64 --decode
```

Now let us explore the resources created by the RabbitMqCluster

```shell
kubectl get all -l app.kubernetes.io/part-of=rabbitmq
```

You should now see a stateful set ( RabbitMQ always uses stateful sets to create a cluster ), headless service ( for the cluster nodes discovery ), And a Kubernetes LoadBalancer ( To access the RabbitMQ WebUI ).


## Replication / Cluster Management

Now that we have seen all the components created by RabbitMQ, let us understand some important aspects of the RabbitMQ cluster. Let us check the status of the cluster using ( You can exec into any of the stateful set pod )

```shell
kubectl exec production-rabbitmqcluster-server-0 -- /bin/sh -c "rabbitmqctl cluster_status --formatter json" | jq
```

Let us now check the nodes in the cluster.

```shell
kubectl exec production-rabbitmqcluster-server-0 -- /bin/sh -c "rabbitmqctl cluster_status --formatter json" | jq -r .running_nodes
```

Alright, enough of the black and green terminal. Let us navigate to something colorful now. RabbitMQ has a Cluster Management Web UI exposed at port 15672. Let us get the IP of the LoadBalancer by

```shell
kubectl get svc production-rabbitmqcluster -o jsonpath='{.status.loadBalancer.ingress[0].ip}'
```

192.168.29.166 ( Since I am using a local cluster my LoadBalancer Ip start with 192* ) 
Let us access the WEBUI from http://192.168.29.166:15672 and login using guest/guest.


---
This is a really basic scenario. RabbitMQ supports many other scenarios like Complex Work Queues, Selective Routing, Publish / Subscribe, etc. There might be many other complex use cases and scenarios.