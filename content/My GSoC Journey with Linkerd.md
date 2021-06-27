+++
title= "Google Summer of Code 2019 at Linkerd"
date= "2019-08-24"
tags= ["linkerd", "smi", "service-mesh", "gsoc", "traffic", "istio"]
+++

My GSoC Internship started with a different aim i.e to write a common GraphQL API to query linkerd metrics and my initial proposal was oriented towards it. I worked on the GraphQL spec along with the documentaion and relevant rfc's for it. By the end of community bonding period, we had a concrete proposal and I also got a scholarship to attend KubeCon + CloudNativeCon 2019 EU in Barcelona and get to meet my mentor and the whole Linkerd team. We had great fun discussing and clarifying my doubts regarding my internship and the linkerd project as a whole!

But then some companies/projects in the service mesh space like Linkerd, Azure, Kinvolk, solo.io etc announced [Service Mesh Interface](https://smi-spec.io) which aims to provide a common interface for whole service mesh technologies so that common tools can be built on top of it. This seemed like a great platform to build the graphql API. Then I discussed with my mentor, linkerd team and the other SMI folks and decided that it would be best to spend my time on SMI-metrics which aims to solve a similar problem but on a bigger scale i.e for service meshes on whole.

The initial work was to make the smi-metrics API Server support more service meshes like [Istio](https://istio.io) and [Consul Connect](https://learn.hashicorp.com/consul). As this work can be a bit hectic and required learning a lot of things specific to those meshes. My mentor suggested me to continue my contributions on Linkerd while also working on SMI-metrics. So, that I don't get struck on one thing.

So, My GSoC work is divided into my contributions to SMI-metrics and also the Linkerd project.

## SMI-Metrics

Intially the SMI-Metrics, had a very hard dependency on prometheus for metrics as it was still a prototype. This can be a challenge for service mesh projects that don't depend on prometheus for queries or had a drastically different structure of time-series (i.e labels,etc ). So, I worked on adding a Mesh Interface and de-coupling the prometheus client. This made the repo more extensiable i.e now service meshes can just implement the interface and reply those metrics irrespective of the presence of prometheus.

During my GSoC period, I successfully got the `istio` support for SMI-Metrics working and also created the release automation for the project.

Istio support is available from Smi-Metrics version `v0.2.0` and a demo of the same can be found [here](https://www.tarunpothulapati.com/posts/istio-metrics-smi/)

## Linkerd

Throughout the GSoC period, I also kept my contributions up to Linkerd solving issues ranging from small bug fixes to small scale features.

Contributions to Linkerd can be found [here](https://github.com/linkerd/linkerd2/pulls?q=is%3Apr+author%3APothulapati+is%3Aclosed)

## Conclusion

My contributions to both Linkerd and SMI-metric repositories can be found [here](https://github.com/Pothulapati/gsoc-meta-linkerd).

I really had a great time working with the Linkerd team especially Thomas (my mentor). Their struggle for simplicity and ultralight-weight service mesh can be seen in each and every PR. The team definitely deserves a huge win and we are already seeing that happen :wink:. They really own the project and make sure all the incoming PR's follow standards. I got to learn a lot from them. Thomas is a great mentor and always gives me long sighted suggestions, insights and also fun tech rants :p . I definitely can't ask for a better mentor. Thanks to Ivan, Alex, Alejandro, Andrew for unblocking me wherever possible and being friendly. :heart:

Special thanks to William, for the hospitality at the Team dinner where I felt a bit awkward and alone :stuck_out_tongue_closed_eyes: . I will remember that for a long time.

Linkerd team is a dream team, and it would be really sad to not continue my contributions. So, I will be continuing my contributions to Linkerd and will be taking up the work to make Tracing a first class component(like prometheus and grafana) in Linkerd. :smiley:
