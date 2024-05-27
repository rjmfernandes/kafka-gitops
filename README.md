This project is based on: https://github.com/osodevops/kafka-gitops-examples 

# Kafka GitOps Example

A Kafka / Confluent GitOps workflow example for multi-env deployments with Flux, Kustomize, Helm and Confluent Operator

Check the details under original repository: https://github.com/osodevops/kafka-gitops-examples 

---

## Demo

Once your local kubernetes cluster is running you can execute:

```shell
cd ./flux-system
kubectl apply -f gotk-components.yaml
kubectl apply -f gotk-sync.yaml
cd ..
```

It should first deploy Flux which will be monitoring your git repository (configured in `flux-system/gotk-sync.yaml`).

You can check the namespaces:

```shell
k get namespaces
```

It will be useful checking the Flux source pod:

```shell
k get pods -n flux-system
```

And do the log tail of your source pod with something like this (replace the name of your pod accordingly):

```shell
k logs -n flux-system -f source-controller-7bbff746cd-glc6h
```

In another terminal you should probably see the CFK operator created under the namespace `confluent` and sometime after the two environments `production` and `sandbox`:

```shell
k get namespaces
```

You can now check the pods of one your environments:

```shell
k get pods -n sandbox 
```

Once both pods for kafka and zookeeper are running and ready you can forward the port:

```shell
kubectl -n sandbox port-forward kafka-0 9092:9092
```

Now in another terminal you can list the topics (make sure to map in your /etc/hosts, `kafka-0.kafka.sandbox.svc.cluster.local` to 127.0.0.1):

```shell
kafka-topics --bootstrap-server kafka-0.kafka.sandbox.svc.cluster.local:9092 --list
```

You can create one topic:

```shell
kafka-topics --bootstrap-server kafka-0.kafka.sandbox.svc.cluster.local:9092 --topic test --create --partitions 1 --replication-factor 1
```

Now let's change in the file `kustomize/base/confluent/zookeeper.yaml` replicas for zookeeper to 3. Commit and push the changes to your git repository. With something like this:

```shell
git add .
git commit -m "testing change of zk replicas"
git push -u origin main
```

Once there is a new Flux sync (you can see by the log of your Flux source controller pod) you can check if there are any changes:

```shell
k get pods -n sandbox 
```

Once all new 3 ZK replicas are ready we can run again:

```shell
kafka-topics --bootstrap-server kafka-0.kafka.sandbox.svc.cluster.local:9092 --list
```
