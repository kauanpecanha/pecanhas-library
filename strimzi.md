# Strimzi Kafka Operator

## 1. Create the Kafka namespace

```bash
kubectl create namespace kafka && kubens kafka
```

---

## 2. Install Strimzi 0.51.0

```bash
export STRIMZI_VERSION="0.51.0"
wget https://github.com/strimzi/strimzi-kafka-operator/releases/download/$STRIMZI_VERSION/strimzi-$STRIMZI_VERSION.tar.gz
tar -xzf strimzi-$STRIMZI_VERSION.tar.gz
rm strimzi-$STRIMZI_VERSION.tar.gz
```

Patch the operator RoleBinding namespace and install:

```bash
sed -i 's/namespace: .*/namespace: kafka/' strimzi-$STRIMZI_VERSION/install/cluster-operator/*RoleBinding*.yaml
kubectl apply -f strimzi-$STRIMZI_VERSION/install/cluster-operator/ -n kafka
kubectl get pods -n kafka -w
```

> Wait until the Strimzi operator pods are Ready before continuing.

---

## 3. Deploy a Kafka cluster with Strimzi 0.51.0

```bash
kubectl apply -f strimzi-$STRIMZI_VERSION/examples/kafka/kafka-persistent.yaml -n kafka
```

> Use `kubectl get pods -n kafka` to verify the Kafka cluster pods are running.

---

## 4. Install Strimzi 1.1.0

```bash
export STRIMZI_VERSION="1.1.0"
wget https://github.com/strimzi/strimzi-kafka-operator/releases/download/$STRIMZI_VERSION/strimzi-$STRIMZI_VERSION.tar.gz
tar -xzf strimzi-$STRIMZI_VERSION.tar.gz
rm strimzi-$STRIMZI_VERSION.tar.gz
```

Patch and install the operator:

```bash
sed -i 's/namespace: .*/namespace: kafka/' strimzi-$STRIMZI_VERSION/install/cluster-operator/*RoleBinding*.yaml
kubectl apply -f strimzi-$STRIMZI_VERSION/install/cluster-operator/ -n kafka
kubectl get pods -n kafka -w
```

> Wait until the Strimzi 1.1.0 operator pods are Ready.

---

## 5. Detect deployed Strimzi and Kafka versions

Set the installed Strimzi operator version, Kafka CR version, and Kafka cluster name from the cluster:

```bash
export STRIMZI_VERSION=$(kubectl -n kafka get deployment strimzi-cluster-operator -o jsonpath='{.spec.template.spec.containers[0].image}' | awk -F ':' '{print $NF}')
export KAFKA_CLUSTER_NAME=$(kubectl -n kafka get kafka -o jsonpath='{.items[0].metadata.name}')
export KAFKA_VERSION=$(kubectl -n kafka get kafka -o jsonpath='{.items[0].spec.kafka.version}')
```

> If your Kafka cluster CR has a different name or you manage multiple Kafka CRs, adjust the `jsonpath` selector accordingly.

Verify values:

```bash
echo "STRIMZI_VERSION=$STRIMZI_VERSION"
echo "KAFKA_CLUSTER_NAME=$KAFKA_CLUSTER_NAME"
echo "KAFKA_VERSION=$KAFKA_VERSION"
```

---

## 6. Run Kafka client pods for testing

Create a consumer pod:

```bash
kubectl run kafka-consumer \
  -n kafka \
  --image=quay.io/strimzi/kafka:${STRIMZI_VERSION}-kafka-${KAFKA_VERSION} \
  --restart=Never \
  -- sleep infinity
```

Start the consumer:

```bash
kubectl exec -it kafka-consumer -n kafka -- \
  bin/kafka-console-consumer.sh \
  --bootstrap-server ${KAFKA_CLUSTER_NAME}-kafka-bootstrap.kafka.svc:9092 \
  --topic upgrade-test \
  --from-beginning
```

Create a producer pod:

```bash
kubectl run kafka-producer \
  -n kafka \
  --image=quay.io/strimzi/kafka:${STRIMZI_VERSION}-kafka-${KAFKA_VERSION} \
  --restart=Never \
  -- sleep infinity
```

Produce test messages:

```bash
kubectl exec -it kafka-producer -n kafka -- bash -c '
  i=0
  while true; do
    echo "upgrade-test-message-$i $(date -Iseconds)" | \
      bin/kafka-console-producer.sh \
        --bootstrap-server ${KAFKA_CLUSTER_NAME}-kafka-bootstrap.kafka.svc:9092 \
        --topic upgrade-test
    i=$((i+1))
    sleep 1
done
'
```

---

## 7. Alternative: use the broker pod directly

If you want to avoid creating extra client pods, you can run the console tools inside an existing broker pod.

Create the topic first:

```bash
kubectl -n kafka exec -it kafka-broker-0 -- \
  bin/kafka-topics.sh \
  --bootstrap-server localhost:9092 \
  --create \
  --topic upgrade-test \
  --partitions 1 \
  --replication-factor 1
```

Produce messages:

```bash
kubectl -n kafka exec -it kafka-broker-0 -- \
  bin/kafka-console-producer.sh \
  --bootstrap-server localhost:9092 \
  --topic upgrade-test
```

> Type messages line by line in the producer shell. Press Ctrl+C to stop it.

Consume from the topic:

```bash
kubectl -n kafka exec -it kafka-broker-0 -- \
  bin/kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic upgrade-test \
  --from-beginning
```

---

## Notes

- Update the `STRIMZI_VERSION` variable when installing a different version.
- The example uses `kafka-persistent.yaml`; adjust if you need a different Kafka cluster topology.
- Use `kubectl get pods -n kafka` and `kubectl logs` to troubleshoot operator or Kafka pod issues.
