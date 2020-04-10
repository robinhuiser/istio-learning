# Istio architecture

Notes for section 4.

## Components

> Please note all components below are since v1.5 bundled inside the `istiod` pod ([Istio Daemon](https://istio.io/blog/2020/istiod/)).

* **Galley** - reads in the Kubernetes YAML and translates this into a structure Istio understands; this means Istio can work with other container management platform then Kubernetes
* **Pilot** - converts the Istio configuration into Envoy format and pushes this configuration to the proxies 
* **Citadel** - manages certificates (TLS, mTLS)
* **Mixer** - to be replaced soon, but handles policy checks (like rate limiting) and telemetry data from the proxies

Additional components deployed along with Istio:

* **Grafana** - UI for viewing metrics (`grafana`)
* **Kiali** - UI for viewing proxy telemetry (`kiali`)
* **Jaeger** - UI for viewing service traces (`tracing`)
* **Prometheus** - metrics scraper (`prometheus`)

~~~bash
# Get the Istio pods
$ kubectl get pods -n istio-system
NAME                                   READY   STATUS    RESTARTS   AGE
grafana-78bc994d79-89q5q               1/1     Running   0          2d
istio-egressgateway-7c9f7d5bd6-mgpwj   1/1     Running   0          2d
istio-ingressgateway-f9b47d445-69h2g   1/1     Running   0          2d
istio-tracing-c7b59f68f-tjqm5          1/1     Running   0          2d
istiod-5745bd5f6b-tr22v                1/1     Running   0          2d
kiali-57fb5bb5c6-rxnnt                 1/1     Running   0          2d
prometheus-78f785fc6b-97njv            2/2     Running   0          2d
~~~

What about `istio-ingressgateway` and `istio-egressgateway`?

From the docs:

> Istio uses ingress and egress gateways to configure load balancers executing at the edge of a service mesh. An ingress gateway allows you to define entry points into the mesh that all incoming traffic flows through. Egress gateway is a symmetrical concept; it defines exit points from the mesh. Egress gateways allow you to apply Istio features, for example, monitoring and route rules, to traffic exiting the mesh.

## Playing with Envoy (optional)

> Def. A **proxy** acts on behalf of the client(s), while a **reverse proxy** acts on behalf of the server(s)

Envoy is only provided as a Docker image, but we can run it locally by [tapping into Homebrew for envoy](https://www.getenvoy.io/install/envoy/macos/).

~~~bash
# Tap
$ brew tap tetratelabs/getenvoy

# Deploy
$ brew install envoy

# Run the proxy with the course provided YAML
$ envoy -c ./github.com/istio-fleetman/_course_files/envoy_demo/config.yaml

# Test Google
$ curl -XGET http://localhost:10000/google
<!doctype html><html
...
...
 </body></html>
~~~

Yup - this works.

> Definition `upstream vs downstream`; An upstream system is any system that sends data to the Collaboration Server system. A downstream system is a system that receives data from the Collaboration Server system. Given the terminology 'upstream' and 'downstream' it may help to make an analogy with a river.
