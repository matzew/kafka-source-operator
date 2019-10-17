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
   kubectl apply -f config/
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

And use the Eventing Kafka Source without Eventing but with `ksvc` or vanilla k8s `service` objects....

