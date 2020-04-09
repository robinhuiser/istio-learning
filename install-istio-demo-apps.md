# Install Istio and demo app

Notes for sections 1-3.

## Install Istio 1.5

~~~bash
# Create the Istio CRDs
$ kubectl apply -f ./github.com/istio-fleetman/_course_files/warmup-exercise/1-istio-init.yaml

# Deploy Istio (works fine on Docker Desktop)
$ kubectl apply -f ./github.com/istio-fleetman/_course_files/warmup-exercise/2-istio-minikube.yaml

# Wait until all pods are up
$ kubectl get pods -n istio-system
NAME                                   READY   STATUS    RESTARTS   AGE
grafana-78bc994d79-89q5q               1/1     Running   0          34m
istio-egressgateway-7c9f7d5bd6-mgpwj   1/1     Running   0          33m
istio-ingressgateway-f9b47d445-69h2g   1/1     Running   0          33m
istio-tracing-c7b59f68f-tjqm5          1/1     Running   0          34m
istiod-5745bd5f6b-tr22v                1/1     Running   0          33m
kiali-57fb5bb5c6-rxnnt                 1/1     Running   0          34m
prometheus-78f785fc6b-97njv            2/2     Running   0          33m

# Set password for Kiali (admin/admin)
$ kubectl apply -f ./github.com/istio-fleetman/_course_files/warmup-exercise/ls 3-kiali-secret.yaml
~~~

## Enable sidecar injection

We are going to tag the namespace where we want Istio to automatically inject the sidecar (proxy) into every pod we deploy.

> There is an alternative which modifies our yaml file to create definitions for Istio, but this "damages" the deployment descriptor since we no longer can use it without deploying Istio...

~~~bash
# Inspect the (default) namespace
$ kubectl describe ns default
Name:         default
Labels:       <none>
Annotations:  <none>
Status:       Active

No resource quota.

No resource limits.

# Set the label istio-injection to enabled on our namespace
$ kubectl label namespace default istio-injection=enabled

# In case we want to save this namespace config in yaml:
$ kubectl get ns default -o yaml
~~~

...which outputs:

~~~yaml
apiVersion: v1
kind: Namespace
metadata:
  creationTimestamp: "2020-04-07T19:52:49Z"
  labels:
    istio-injection: enabled
  name: default
  resourceVersion: "32711"
  selfLink: /api/v1/namespaces/default
  uid: 31a76573-0c47-4ed8-8881-265edf098c1f
spec:
  finalizers:
  - kubernetes
status:
  phase: Active
~~~

## Deploy the demo app

~~~bash
# Deploy the apps
$ kubectl apply -f ./github.com/istio-fleetman/_course_files/warmup-exercise/4-application-full-stack.yaml

# Check with watch until the pods are installed
$ watch kubectl get pods

NAME                                  READY   STATUS    RESTARTS   AGE
api-gateway-7fcd9cf875-f8d59          2/2     Running   0          3m49s
photo-service-66595b6747-7h2sq        2/2     Running   0          3m49s
position-simulator-57f5bf77c8-wfcgd   2/2     Running   0          3m49s
position-tracker-7848c69fb-c88j5      2/2     Running   0          3m49s
staff-service-7d78f9d785-276st        2/2     Running   0          3m49s
vehicle-telemetry-5bdc596c84-fxd5c    2/2     Running   0          3m49s
webapp-685777b869-2jqr8               2/2     Running   0          3m49s
~~~

The app is now running on [http://localhost:30080](http://localhost:30080).

## Troubleshoot the app

The Kiali dashboard can be found on [http://localhost:31000](http://localhost:31000) - this has been prepared by the course deployment descriptor by enabling the `NodePort` for the `kiali` service running in the `istio-system` namespace:

~~~bash
kubectl get svc -n istio-system | grep kiali
kiali                       NodePort       10.100.122.48    <none>        20001:31000/TCP     137m
~~~

## Temporarily patch the app

While the instructor explained the application deployment YAML had no Istio definitions, it seems the section below was an exception:

~~~yaml
# See case #23 - this is a legacy service that we're integrating with
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: fleetman-driver-monitoring
spec:
  hosts:
  - 2oujlno5e4.execute-api.us-east-1.amazonaws.com
  http:
  - match:
    - port: 80
    route:
    - destination:
        host: 2oujlno5e4.execute-api.us-east-1.amazonaws.com
        port:
          number: 443
~~~

By Adding a timeout to the service we could implement some simple circuit breaker pattern:

~~~yaml
    - port: 80
    timeout: 1s
    route:
~~~
