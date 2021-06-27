
+++
title= "Traffic Splitting in Linkerd using SMI"
date= "2019-08-13"
tags= ["linkerd", "smi", "service-mesh", "gsoc", "traffic"]
+++

# Traffic Splitting in Linkerd

Linkerd is CNCF service mesh project for Kubernetes. Linkerd 2.4 introduces *traffic splitting* functionality, which allows users to split traffic destined for a particular service across multiple services. Traffic can be split based on the weight given for a particular service, or by percentage. This traffic splitting is useful for performing canary deploys and blue/green deploys, which can be useful for reducing risk when publishing new code to production — the push can be done gradually, and potentially rolled back if things go wrong.

Without a service mesh, using just Kubernetes, you can partially accomplish traffic shifting with pure label flipping. However, the granularity of this appraoch is pretty coarse, especially for small deployments, since this must be done on a per-pod basis. Using Linkerd, the service mesh gives you fine-grained control by allowing you shift arbitrary portions of the traffic to whichever service you want, without changing any code.

In this post, I'll walk you through an example of using Linkerd for traffic splitting on Istio's BookInfo example app.

## How is it done in Linkerd?

A service mesh is divided into control plane and data plane (proxy) components. In Linkerd, whenever the data plane proxies have to perform a request, they get the config from the **destination** component in the control plane.

![](/images/control-plane-5c1c76a5-5431-4134-b08c-0f3bef57fcac.png)

The destination component of the control plane watches for changes in service mesh configuration (implemented as Kubernetes Custom Resource Definitions) and then pushes the right config for proxies to follow. Rather than introducing its own configuration format for traffic splitting, Linkerd follows the [SMI spec](https://www.smi-spec.io), which aims to provide a unified, generalized configuration model for service meshes (just like ingress, CRI, etc in Kubernetes).

## Demo

It's easy to try this out for yourself. First, download the Linkerd `2.4` by running `curl https://run.linkerd.io/install | sh` . Once that is done, Linkerd control plane can be installed in the Kubernetes cluster by running 

`linkerd install | kubectl apply -f -`

For this demo, let's use [Istio's BookInfo sample application](https://github.com/istio/istio/tree/master/samples/bookinfo), The bookinfo sample can be retrieved [here](https://istio.io/docs/setup/kubernetes/#downloading-the-release).

Next, the Linkerd data plane proxies have to be injected into each pod's manifest, and this has to be applied to the cluster. This can be done by running 

`linkerd inject ./samples/bookinfo/platform/kube/bookinfo.yaml | kubectl apply -f -`

You can see the the following pods running in your cluster: 3 review page pods, 1 product page pod, and 1 details page pod.

```
    kubectl get pods
    NAME                              READY   STATUS    RESTARTS   AGE
    details-v1-9f59fd579-ncgkg        2/2     Running   0          11m
    productpage-v1-66df8b7cd5-nrzwv   2/2     Running   0          11m
    ratings-v1-f5d68559-qjh95         2/2     Running   0          11m
    reviews-v1-64c8bc6778-bzlf8       2/2     Running   0          11m
    reviews-v2-756495d66-brfrr        2/2     Running   0          11m
    reviews-v3-7cdb64bdfd-z2m4q       2/2     Running   0          11m

```
with the following services.

```python
 kubectl get svc
    NAME          TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
    details       ClusterIP   10.96.115.62    <none>        9080/TCP   13m
    kubernetes    ClusterIP   10.96.0.1       <none>        443/TCP    7h56m
    productpage   ClusterIP   10.109.31.20    <none>        9080/TCP   13m
    ratings       ClusterIP   10.101.57.168   <none>        9080/TCP   13m
    reviews       ClusterIP   10.105.52.139   <none>        9080/TCP   13m
```

The productpage is the homepage of the application and it can be viewed by running 

```
    kubectl port-forward svc/productpage 9080:9080
```
Now, checking `localhost:9080` would show you a product page, with a list of reviews on the right. Those reviews are being loaded from the `reviews` service which is backed by the 3 reviews pods. The requests to the reviews service are randomly sent to one of the 3 review pods, as they represent different versions of this service. The three different versions provide different output:

 v1 (with no stars) 

![](/images/v1-972e7ce4-8e3a-43b6-85f7-e0df617b2724.png)

v2 (with orange stars)

![](/images/v2-c3bcd0df-5d6e-4e4e-b4ba-f19e2cda762b.png)

v3 (with black stars)

![](/images/v3-9b0b281a-a442-43db-bf7e-ab49a0d8862b.png)

Now, let's have the reviews service only split traffic to v2 and v3 versions of the application. 


In Linkerd's approach to traffic splitting, services are used as the core primitives. This, we need to create two new review services that correspond to the pods. In the following manifest, two new services `reviews-v2` and `reviews-v3` are created that correspond to the v2 and v3 pods respectively by using label selectors:

<script src="https://gist.github.com/Pothulapati/e94acb41d2e489561ae366bb2340552e.js"></script>

There are two new services created

```
    kubectl get svc
    NAME          TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
    details       ClusterIP   10.96.115.62     <none>        9080/TCP   35m
    kubernetes    ClusterIP   10.96.0.1        <none>        443/TCP    8h
    productpage   ClusterIP   10.109.31.20     <none>        9080/TCP   35m
    ratings       ClusterIP   10.101.57.168    <none>        9080/TCP   35m
    reviews       ClusterIP   10.105.52.139    <none>        9080/TCP   35m
    reviews-v2    ClusterIP   10.106.174.219   <none>        9080/TCP   7s
    reviews-v3    ClusterIP   10.96.125.224    <none>        9080/TCP   7s
```

Now, let's apply the SMI TrafficSplit CRD, which makes the requests to reviews service split between reviews-v3 and reviews-v2 : 

<script src="https://gist.github.com/Pothulapati/c2b8c33c19f1d945f511c2d6ce16b28d.js"></script>

This tells Linkerd's control plane that whenever there are requests to the `reviews` service, to split them across the `reviews-v2` and `reviews-v3` based on the weights provided. (In this case, a 50-50 split.)

If we now go back to our product page, we can only see the reviews with red or black stars appear on each refresh, and they appear equally—which means the traffic is split equally, as equal weights are provided in the config. Success!

## Conclusion

Linkerd uses Kubernetes's *Service* abstraction as the core mechanism for traffic splitting. I think is a good choice, as services are not physical resources but a routing mechanism. By comparison, Istio's approach introduces a new level of abstraction based on VirtualServices which requires some learning! By keeping it to the Kubernetes primitives that we know and love, Linkerd makes this experience very smooth and simple.

In practice, we wouldn't want to shift traffic without also monitoring success rate—if we're sending new traffic to a service that is failing, we probably want to shift traffic back to the original version! This is where tools like [Flagger](https://github.com/weaveworks/flagger) come in! Flagger combines traffic shifting and L7 metrics to do canary deployments, etc. It will slowly increase the weight to the newer version, based on the metrics, and if there is any problem (e.g. failed requests), it would roll back. If not it will continue increasing the weight until all the requests are routed to the newer version. A demo of Linkerd with Flagger can be seen [here](https://www.youtube.com/watch?v=nlg3yiCNmY8). This is the promise of SMI: tools like Flagger can be built on top of [SMI](https://smi-spec.io/) and they work on all the meshes that implement it.

Special thanks to William Morgan for helping with this post. :)