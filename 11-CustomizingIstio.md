# Customizing Istio

## Instaling Istioclt
---
```bash
wget https://github.com/istio/istio/releases/download/1.9.4/istio-1.9.4-linux-arm64.tar.gz

tar -zxvf ./istio-1.9.4-linux-arm64.tar.gz
mv istio-1.9.4-linux-arm64.tar istio
cd ./istio/bin/
export PATH=$(pwd):$PATH

istioctl version;
```

## Installing Istio
---

- `Basic Instalation`
```bash
# ["Istio core" "Istiod" "Ingress gateways"]
istioctl install
``` 

- `Profile Instalation`
```bash
istioctl install --set profile=demo
```

- `Getting Istio Profile`
```
istioctl profile dump <profile-name> > profile.yaml
```
## Istio Integrations

Istio integration or addons provides usuful third parties to run alonside with Istio like: `Kiali`, `Prometheus`, `Grafana` and `Jaeger`. All instalation could be archived by the yamls in the addonn folder of istioclt

```bash
cd istio/samples/addons;
kubectl apply -f prometheus.yaml
kubectl apply -f grafana.yaml
kubectl apply -f kiali.yaml
kubectl apply -f jaeger.yaml
```
## Generating Istio Yaml

```bash
istioctl profile dump default > profile-tuned.yaml
# edit your tunes in profile-tuned.yaml
istioctl manifest generate -f ./profile-tuned.yaml > istio-install.yaml
```