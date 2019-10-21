# Knative Kafka Source Operator

>NOTE: POC... not more

1. Checkout this fork to `$GOPATH/src/knative.dev/eventing-operator`

1. Install the
   [KnativeEventing CRD](config/crds/eventing_v1alpha1_knativeeventing_crd.yaml)

   ```
   kubectl apply -f config/crds/eventing_v1alpha1_knativeeventing_crd.yaml
   ```

1. Install the operator

   To install run the command:

   ```
   ko apply -f config/
   ```

1. Install the
   [Eventing custom resource](#the-eventing-custom-resource)

```sh
cat <<-EOF | kubectl apply -f -
apiVersion: v1
kind: Namespace
metadata:
 name: knative-sources
---
apiVersion: operator.knative.dev/v1alpha1
kind: Eventing
metadata:
  name: knative-eventing
  namespace: knative-sources
EOF
```

Apply some config for the source, like

```yaml
apiVersion: sources.eventing.knative.dev/v1alpha1
kind: KafkaSource
metadata:
  name: kafka-source
spec:
  consumerGroup: knative-groupp
  bootstrapServers: my-cluster-kafka-bootstrap.kafka:9092
  topics: mytopic
  sink:
    apiVersion: serving.knative.dev/v1alpha1
    kind: Service
    name: event-display
```

And use the Eventing Kafka Source without Eventing but with `ksvc` or vanilla k8s `service` objects - for this see hello-display https://knative.dev/development/eventing/getting-started/#creating-event-consumers

```bash
kubectl apply --filename - << END
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-display
spec:
  replicas: 1
  selector:
    matchLabels: &labels
      app: hello-display
  template:
    metadata:
      labels: *labels
    spec:
      containers:
        - name: event-display
          # Source code: https://github.com/knative/eventing-contrib/blob/release-0.6/cmd/event_display/main.go
          image: gcr.io/knative-releases/github.com/knative/eventing-sources/cmd/event_display@sha256:37ace92b63fc516ad4c8331b6b3b2d84e4ab2d8ba898e387c0b6f68f0e3081c4

---

# Service pointing at the previous Deployment. This will be the target for event
# consumption.
  kind: Service
  apiVersion: v1
  metadata:
    name: hello-display
  spec:
    selector:
      app: hello-display
    ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
END
```


and event source that uses hello-display

```
kubectl  apply --filename - << END
apiVersion: sources.eventing.knative.dev/v1alpha1
kind: KafkaSource
metadata:
  name: kafka-source-hello
spec:
  consumerGroup: knative-groupp
  bootstrapServers: my-cluster-kafka-bootstrap.kafka:9092
  topics: mytopic
  sink:
    apiVersion: v1
    kind: Service
    name: hello-display
END
```