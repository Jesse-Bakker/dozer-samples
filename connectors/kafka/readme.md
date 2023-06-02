Init resources
```shell
kubectl apply -f 00.namespace.yaml
```

```shell
kubectl apply -f 01.zookeeper.yaml
```

```shell
kubectl apply -f 02.kafka.yaml
```

```shell
kubectl apply -f 03.sr.yaml
```

## Create topic 
```shell
kubectl exec -it $(kubectl get pod -n kafka -l app=kafka-broker -o jsonpath="{.items[0].metadata.name}") -n kafka -- kafka-topics.sh --create --topic transactions --bootstrap-server localhost:9092
```
## Run port forwarding
```shell
kubectl port-forward $(kubectl get pod -n kafka -l app=kafka-broker -o jsonpath="{.items[0].metadata.name}") 9092 -n kafka &
kubectl port-forward $(kubectl get pod -n kafka -l app=zookeeper -o jsonpath="{.items[0].metadata.name}") 2181 -n kafka &
kubectl port-forward $(kubectl get pod -n kafka -l app=schema-registry-app -o jsonpath="{.items[0].metadata.name}") 8081 -n kafka
```

## Create schemas
```shell
curl -X POST -H "Content-Type: application/vnd.schemaregistry.v1+json" --data '{"schema": "{\"type\":\"record\",\"name\":\"transaction\",\"namespace\":\"dozer.samples\",\"fields\":[{\"name\":\"id\",\"type\":\"int\"}, {\"name\":\"customer_id\",\"type\":\"int\"},{\"name\":\"amount\",\"type\":\"float\"},{\"name\":\"location\",\"type\":\"string\"},{\"name\":\"provider\",\"type\":\"string\"}]}"}' http://localhost:8081/subjects/transactions-value/versions
curl -X POST -H "Content-Type: application/vnd.schemaregistry.v1+json" --data '{"schema": "{\"type\":\"record\",\"name\":\"transactions\",\"namespace\":\"dozer.samples\",\"fields\":[{\"name\":\"id\",\"type\":\"int\"}]}"}' http://localhost:8081/subjects/transactions-key/versions
```