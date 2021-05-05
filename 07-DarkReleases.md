# Dark Releases

Dark Releases is a way to release software without it become public accessiable. Becomming possible to test it within the real world.

## Setup Demo
---
```bash
kubectl apply -f ./data/4\ Dark\ Releases/1-istio-init.yaml
kubectl apply -f ./data/4\ Dark\ Releases/2-istio-minikube.yaml
kubectl apply -f ./data/4\ Dark\ Releases/3-kiali-secret.yaml
kubectl apply -f ./data/4\ Dark\ Releases/4-label-default-namespace.yaml
kubectl apply -f ./data/4\ Dark\ Releases/5-application-no-istio.yaml

```

## Header Base Routing
---

To routing based on header it is necessary to have the following resources in place: (All magic are inside the Virtual Service Resource)
```yaml
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: ingress-gateway-configuration
spec:
  selector:
    istio: ingressgateway # use Istio default gateway implementation
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "*"   # Domain name of the external website
---
kind: VirtualService
apiVersion: networking.istio.io/v1alpha3
metadata:
  name: fleetman-webapp
  namespace: default
spec:
  hosts:
    - "*"
  gateways:
    - ingress-gateway-configuration
  http:
    - match: # <-- Routing based on headers 
      - headers:  # IF
          my-header: # <-- custom header
            exact: canary # <-- header value
      route: # THEN
      - destination:
          host: fleetman-webapp
          subset: experimental # <-- experimental release
    - route: # CATCH ALL
      - destination:
          host: fleetman-webapp
          subset: original
---
kind: DestinationRule
apiVersion: networking.istio.io/v1alpha3
metadata:
  name: fleetman-webapp
  namespace: default
spec:
  host: fleetman-webapp
  subsets:
    - labels:
        version: original
      name: original
    - labels:
        version: experimental
      name: experimental
```

The result of this configuration is that now the requests are routed based on a header value so:

```bash
curl -s http://$(minikube ip):31380/ | grep title 
# Returns 
# <title>Fleet Management</title>

curl -H "my-header:canary" -s http://$(minikube ip):31380/ | grep title
# Returns
# <title>Fleet Management Istio Premium Enterprise Edition</title>
```

## Dark Demos
---

Also, Istio allows to bring dark release to another level: `The Dark Demo`. Through it is possible to navigate to a secret experience in the application:

So, together with the dark fleetman-webapp we will release the dark `staff-service`.

```yaml
kind: VirtualService
apiVersion: networking.istio.io/v1alpha3
metadata:
  name: fleetman-staff-service
  namespace: default
spec:
  hosts:
    - fleetman-staff-service.default.svc.cluster.local
  http:
    - match: 
      - headers:
          x-my-header:
            exact: canary 
      route: # THEN
      - destination:
          host: fleetman-staff-service
          subset: risky
    - route: # CATCH ALL
      - destination:
          host: fleetman-staff-service
          subset: safe
---
kind: DestinationRule
apiVersion: networking.istio.io/v1alpha3
metadata:
  name: fleetman-staff-service
  namespace: default
spec:
  host: fleetman-staff-service.default.svc.cluster.local
  subsets:
    - labels:
        version: safe
      name: safe
    - labels:
        version: risky
      name: risky
```

As a result, we have a Dark Demo based on headers!

![Consistent Hash Limitation](./artifacts/07-DarkReleases.gif)