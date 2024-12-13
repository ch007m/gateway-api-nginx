# How to play with Nginx Gateway API

## Introduction

https://blog.nashtechglobal.com/hands-on-kubernetes-gateway-api-with-nginx-gateway-fabric/
https://docs.nginx.com/nginx-gateway-fabric/get-started/

The following scenario which has been tested successfully use the following port configuration between the `host`, the `container` and pod running the `nginx-fabric-gateway` or `ngf`:

* **HTTP traffic**

  localhost port: `8080` <-> podman container port: `31437`
  kubernetes Service nodePort: `31437` <-> Pod container port: `80`

* **HTTPS traffic**

  localhost port: `8443` <-> podman container port: `31438`
  kubernetes Service nodePort: `31438` <-> Pod container port: `433`

## Instructions

- Create the kind cluster
```bash
set -x KIND_EXPERIMENTAL_PROVIDER podman
kind delete cluster
kind create cluster --config kind.cfg
```
- Deploy the `Gateway API` CRDs
```bash
kubectl kustomize "https://github.com/nginxinc/nginx-gateway-fabric/config/crd/gateway-api/standard?ref=v1.5.0" | kubectl apply -f -
```
- Install `nginx fabric gateway` using helm and the configuration defined using the `values.yaml` file
```bash
helm uninstall ngf -n nginx-gateway
helm install ngf oci://ghcr.io/nginxinc/charts/nginx-gateway-fabric --create-namespace -n nginx-gateway -f values.yaml
...
NAMESPACE            NAME                                         READY   STATUS    RESTARTS   AGE
nginx-gateway        ngf-nginx-gateway-fabric-7667ddbc9d-4fq7f    1/2     Running   0          10s
```
- Create the application resources:
```bash
kubectl delete -f microservices.yaml
kubectl apply -f microservices.yaml
```
**Remark**: github project: https://github.com/nginxinc/NGINX-Demos/tree/master/nginx-hello

- Create the `Gateway` resource

```bash
kubectl delete -f gateway.yaml
kubectl apply -f gateway.yaml
```
- Create the `HTTPRoutes` resources
```bash
kubectl delete -f httproutes.yaml
kubectl apply -f httproutes.yaml
```
- curl the service
```bash
curl http://cafe.localtest.me:8080/coffee
Server address: 10.244.0.11:8080
Server name: coffee-6db967495b-gv7q4
Date: 13/Dec/2024:13:18:58 +0000
URI: /coffee
Request ID: 158cafccd8e407bb1dd5ebe3f9a0abf4

curl http://tea.localtest.me:8080/tea
Server address: 10.244.0.10:8080
Server name: tea-7b7d6c947d-q8gjr
Date: 13/Dec/2024:13:18:58 +0000
URI: /tea
Request ID: 562a46952c985b0d71d2ac7b1edc47b9
```