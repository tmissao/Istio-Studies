# Fault Injection

With Istio it is possible to deliberately inject faults into the system. Thus, it is possible to analyse and test how the application will behavioured and check if it is `fault tolerant`. 

So it is possible to configure a microservice to throw an error or add a delay in the response.

## Injecting Failures

In this case we will inject failures on the service `fleetman-vehicle-telemetry`, for that will be necessary a virtual service.

- `Injecting Failures`
```yaml
kind: VirtualService
apiVersion: networking.istio.io/v1alpha3
metadata:
  name: fleetman-vehicle-telemetry
spec:
  hosts:
    - fleetman-vehicle-telemetry.default.svc.cluster.local
  http:
    - fault: # <-- Adds a Fault
        abort: # <-- Defines the type of the fault (Error)
          httpStatus: 503 # <-- Sets the HTTP Code
          percentage:
            value: 50 # <-- Defines how often the fault will occur
      route:
        - destination:
            host: fleetman-vehicle-telemetry.default.svc.cluster.local
---
kind: DestinationRule
apiVersion: networking.istio.io/v1alpha3
metadata:
  name: fleetman-vehicle-telemetry
spec:
  host: fleetman-vehicle-telemetry.default.svc.cluster.local
  subsets: []
```

As a result, the application still works. However, it is degraded not showing the vehicle telemetry data (Speed/mph)

![Failure Injection](./artifacts/08-FailureInjection.gif)

- `Injecting Delays`
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
      fault: # <-- Injects an Fault just on requests with canary header
        delay: # <-- the type of the fault is a delay
          percentage:
            value: 100 # <-- This fault will happen on 100% of the requests
          fixedDelay: 5s #<-- Will be introduzed a delay of 5 seconds
      route: # THEN
      - destination:
          host: fleetman-staff-service
          subset: risky
    - route: # CATCH ALL
      - destination:
          host: fleetman-staff-service
          subset: safe
```

![Delay Injection](./artifacts/08-DelayInjection.gif)
