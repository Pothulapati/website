+++
title= "SMI Metrics with Istio"
date= "2019-08-23"
tags= ["istio", "smi", "service-mesh", "gsoc", "traffic"]
+++

As a part of my GSoC 2019 internship with Linkerd, I was tasked to add support for Istio on SMI-Metrics and also to add basic things to the repository like release automation, etc.

[SMI-Metrics](https://github.com/deislabs/smi-metrics) aims to have a common API to consume metrics from Service Meshes. Tools like flagger can be built on top of SMI, without having to use mesh specific API's. This creates a standardisation for Service Mesh metrics.

The SMI-Metrics is an [extension API Server](https://kubernetes.io/docs/tasks/access-kubernetes-api/setup-extension-api-server/) that users can install on their k8s cluster which then talks to the service mesh specific API's and returns metrics in a standard format as per the [SMI spec](https://github.com/deislabs/smi-spec/blob/master/traffic-metrics.md).

As of `v0.2.0`, the repository supports both Linkerd and Istio.

## Istio Support

Istio has a pluggable way of metrics i.e metrics can be added dynamically by submitting custom resources i.e Handler, Instances and Rules and Istio sends those metrics to prometheus in real time. More on this [here](https://istio.io/docs/tasks/telemetry/metrics/collecting-metrics/)

Istio support of SMI works by installing some handler and isntances that make Istio send metrics that involve the labels i.e kind, etc needed by SMI and then reading those metrics from prometheus whenever query requests come in.

### Demo

First, make sure the kubernetes cluster has a working installation of Istio, by following this [doc](https://istio.io/docs/setup/kubernetes/)

Once this is done, let's run a demo app which has the Istio side-cars installed whose metrics we will query.

First, Let's create the `emojivoto` namespace.

```bash
kubectl apply -f https://gist.githubusercontent.com/Pothulapati/c8789307ec24fd73bb5ba5b892154ffd/raw/3386da4bec1472c11b1e0222c8143ca0d8c2a60b/namespace.yaml
```

Let's add the injection label, that makes Istio to inject sidecars automatically.

```bash
kubectl label namespace emojivoto istio-injection=enabled
```

Now, Let's deploy the Emojivoto services on to the same namespace.

```bash
kubectl apply -f https://gist.githubusercontent.com/Pothulapati/abed2e3f638fa0c16947b76191f89353/raw/8a891cf5d228a386bfc59cb7a1ce4c5af2f11db5/emojivoto.yaml
```

Now, If we check the pods in the emojivoto namespacc.

```bash
tarun@tarun-Inspiron-5559 ~/w/smi-metrics> kubectl get pods -n emojivoto
NAME                        READY   STATUS    RESTARTS   AGE
emoji-58c9579849-rp7nc      2/2     Running   0          28h
vote-bot-774764fd7f-blhzx   2/2     Running   0          28h
voting-66d5cdc46d-4fzqh     2/2     Running   0          28h
web-7f8455487f-rw9h5        2/2     Running   0          28h

```

We can see that all pods have two containers running i.e one application container and the Istio proxy (i.e envoy).

Regarding the emojivoto sample, `vote-bot` service keeps sending requests to `web`, which further talks to `emoji` and `voting`.

Now, Let's install SMI-Metrics Extension API Server and see the metrics of these application services.

The `Smi-metrics v0.2.0` helm package can be downloaded from the [link](https://github.com/deislabs/smi-metrics/releases/tag/v0.2.0)

Once it is unpacked. The package can be installed by running.

```bash
tarun@tarun-Inspiron-5559 ~/w/smi-metrics> helm install . --set adapter=istio --name dev
NAME:   dev
LAST DEPLOYED: Sat Aug 24 18:50:53 2019
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/ConfigMap
NAME             DATA  AGE
dev-smi-metrics  1     4s

==> v1/Deployment
NAME             READY  UP-TO-DATE  AVAILABLE  AGE
dev-smi-metrics  1/1    1           1          4s

==> v1/Pod(related)
NAME                              READY  STATUS   RESTARTS  AGE
dev-smi-metrics-74df88c5fc-stxbk  1/1    Running  0         4s

==> v1/RoleBinding
NAME             AGE
dev-smi-metrics  4s

==> v1/Secret
NAME             TYPE               DATA  AGE
dev-smi-metrics  kubernetes.io/tls  2     4s

==> v1/Service
NAME             TYPE       CLUSTER-IP   EXTERNAL-IP  PORT(S)  AGE
dev-smi-metrics  ClusterIP  10.0.19.175  <none>       443/TCP  4s

==> v1/ServiceAccount
NAME             SECRETS  AGE
dev-smi-metrics  1        4s

==> v1alpha2/handler
NAME            AGE
smi-prometheus  4s

==> v1alpha2/instance
NAME                AGE
smirequestcount     4s
smirequestduration  4s

==> v1alpha2/rule
NAME     AGE
smiprom  3s

==> v1beta1/APIService
NAME                          AGE
v1alpha1.metrics.smi-spec.io  4s
```

As we can see, all the parts of the SMI installation have been installed successfully. Now, queries on the emojivoto resources can be done by running.

```bash
tarun@tarun-Inspiron-5559 ~/w/smi-metrics> kubectl get --raw /apis/metrics.smi-spec.io/v1alpha1/namespaces/emojivoto/deployments/web | jq
```

```json
{
  "kind": "TrafficMetrics",
  "apiVersion": "metrics.smi-spec.io/v1alpha1",
  "metadata": {
    "name": "web",
    "namespace": "emojivoto",
    "selfLink": "/apis/metrics.smi-spec.io/v1alpha1/namespaces/emojivoto/deployments/web",
    "creationTimestamp": "2019-08-24T13:22:52Z"
  },
  "timestamp": "2019-08-24T13:22:52Z",
  "window": "30s",
  "resource": {
    "kind": "Deployment",
    "namespace": "emojivoto",
    "name": "web"
  },
  "edge": {
    "direction": "from",
    "resource": null
  },
  "metrics": [
    {
      "name": "p99_response_latency",
      "unit": "ms",
      "value": "9m"
    },
    {
      "name": "p90_response_latency",
      "unit": "ms",
      "value": "7m"
    },
    {
      "name": "p50_response_latency",
      "unit": "ms",
      "value": "3m"
    },
    {
      "name": "success_count",
      "value": "108"
    },
    {
      "name": "failure_count",
      "value": "4"
    }
  ]
}
```

These are the golden metrics of inbound requests that `web` deployment is recieving.

```bash
tarun@tarun-Inspiron-5559 ~/w/smi-metrics> kubectl get --raw /apis/metrics.smi-spec.io/v1alpha1/namespaces/emojivoto/deployments/web/edges | jq
```

```json
{
  "kind": "TrafficMetricsList",
  "apiVersion": "metrics.smi-spec.io/v1alpha1",
  "metadata": {
    "selfLink": "/apis/metrics.smi-spec.io/v1alpha1/namespaces/emojivoto/deployments/web/edges"
  },
  "resource": {
    "kind": "Deployment",
    "namespace": "emojivoto",
    "name": "web"
  },
  "items": [
    {
      "kind": "TrafficMetrics",
      "apiVersion": "metrics.smi-spec.io/v1alpha1",
      "metadata": {
        "name": "web",
        "namespace": "emojivoto",
        "selfLink": "/apis/metrics.smi-spec.io/v1alpha1/namespaces/emojivoto/deployments/web/edges",
        "creationTimestamp": "2019-08-24T13:22:26Z"
      },
      "timestamp": "2019-08-24T13:22:26Z",
      "window": "30s",
      "resource": {
        "kind": "Deployment",
        "namespace": "emojivoto",
        "name": "web"
      },
      "edge": {
        "direction": "from",
        "resource": {
          "kind": "Deployment",
          "namespace": "emojivoto",
          "name": "vote-bot"
        }
      },
      "metrics": [
        {
          "name": "p99_response_latency",
          "unit": "ms",
          "value": "22m"
        },
        {
          "name": "p90_response_latency",
          "unit": "ms",
          "value": "9m"
        },
        {
          "name": "p50_response_latency",
          "unit": "ms",
          "value": "5m"
        },
        {
          "name": "success_count",
          "value": "116"
        },
        {
          "name": "failure_count",
          "value": "4"
        }
      ]
    },
    {
      "kind": "TrafficMetrics",
      "apiVersion": "metrics.smi-spec.io/v1alpha1",
      "metadata": {
        "name": "web",
        "namespace": "emojivoto",
        "selfLink": "/apis/metrics.smi-spec.io/v1alpha1/namespaces/emojivoto/deployments/web/edges",
        "creationTimestamp": "2019-08-24T13:22:26Z"
      },
      "timestamp": "2019-08-24T13:22:26Z",
      "window": "30s",
      "resource": {
        "kind": "Deployment",
        "namespace": "emojivoto",
        "name": "web"
      },
      "edge": {
        "direction": "to",
        "resource": {
          "kind": "Deployment",
          "namespace": "emojivoto",
          "name": "voting"
        }
      },
      "metrics": [
        {
          "name": "p99_response_latency",
          "unit": "ms",
          "value": "4m"
        },
        {
          "name": "p90_response_latency",
          "unit": "ms",
          "value": "4m"
        },
        {
          "name": "p50_response_latency",
          "unit": "ms",
          "value": "2m"
        },
        {
          "name": "success_count",
          "value": "30"
        },
        {
          "name": "failure_count",
          "value": "0"
        }
      ]
    },
    {
      "kind": "TrafficMetrics",
      "apiVersion": "metrics.smi-spec.io/v1alpha1",
      "metadata": {
        "name": "web",
        "namespace": "emojivoto",
        "selfLink": "/apis/metrics.smi-spec.io/v1alpha1/namespaces/emojivoto/deployments/web/edges",
        "creationTimestamp": "2019-08-24T13:22:26Z"
      },
      "timestamp": "2019-08-24T13:22:26Z",
      "window": "30s",
      "resource": {
        "kind": "Deployment",
        "namespace": "emojivoto",
        "name": "web"
      },
      "edge": {
        "direction": "to",
        "resource": {
          "kind": "Deployment",
          "namespace": "emojivoto",
          "name": "emoji"
        }
      },
      "metrics": [
        {
          "name": "p99_response_latency",
          "unit": "ms",
          "value": "4m"
        },
        {
          "name": "p90_response_latency",
          "unit": "ms",
          "value": "4m"
        },
        {
          "name": "p50_response_latency",
          "unit": "ms",
          "value": "2m"
        },
        {
          "name": "success_count",
          "value": "60"
        },
        {
          "name": "failure_count",
          "value": "0"
        }
      ]
    }
  ]
}
```

The above response shows the `edges` from `web`, i.e inbound and outbound requests from deployments and the respective golden metrics for the corresponding edges.

The same queries can be done for different kinds of resources i.e pods, namespaces, daemonsets, etc as specified in the [smi-spec](https://github.com/deislabs/smi-spec/blob/master/traffic-metrics.md)

Isito support was added pretty recently. Feel free to try out and submit feedback/issues. :smiley:
