# Deploying the DNS Cluster Add-on

In this lab you will deploy the [DNS add-on](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/) which provides DNS based service discovery, backed by [CoreDNS](https://coredns.io/), to applications running inside the Kubernetes cluster.

## The DNS Cluster Add-on

Deploy the `coredns` cluster add-on:

```shell
kubectl apply -f https://github.com/fsocietyinc/kubernetes-the-hard-way-on-azure/blob/master/deployments/coredns.yaml
```

> output

```shell
serviceaccount/coredns created
clusterrole.rbac.authorization.k8s.io/system:coredns created
clusterrolebinding.rbac.authorization.k8s.io/system:coredns created
configmap/coredns created
deployment.extensions/coredns created
service/kube-dns created
```

List the pods created by the `kube-dns` deployment:

```shell
kubectl get pods -l k8s-app=kube-dns -n kube-system
```

> output

```shell
NAME                       READY   STATUS    RESTARTS   AGE
coredns-59845f77f8-b4x76   1/1     Running   0          15s
coredns-5677dc4cdb-m8n69   1/1     Running   0          13s
```

## Verification

Create a `busybox` deployment:

```shell
kubectl run busybox --image=busybox:1.28 --command -- sleep 3600
```

List the pod created by the `busybox` deployment:

```shell
kubectl get pods -l run=busybox
```

> output

```shell
NAME                      READY   STATUS    RESTARTS   AGE
busybox                   1/1     Running   0          10s
```

Retrieve the full name of the `busybox` pod:

```shell
POD_NAME=$(kubectl get pods -l run=busybox -o jsonpath="{.items[0].metadata.name}")
```

Execute a DNS lookup for the `kubernetes` service inside the `busybox` pod:

```shell
kubectl exec -ti $POD_NAME -- nslookup kubernetes
```

> output

```shell
Server:    10.32.0.10
Address 1: 10.32.0.10 kube-dns.kube-system.svc.cluster.local

Name:      kubernetes
Address 1: 10.32.0.1 kubernetes.default.svc.cluster.local
```

Next: [Smoke Test](13-smoke-test.md)
