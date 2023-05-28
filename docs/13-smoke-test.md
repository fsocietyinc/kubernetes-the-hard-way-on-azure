# Smoke Test

In this lab you will complete a series of tasks to ensure your Kubernetes cluster is functioning correctly.

## Data Encryption

In this section you will verify the ability to [encrypt secret data at rest](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/#verifying-that-data-is-encrypted).

Create a generic secret:

```shell
kubectl create secret generic kubernetes-the-hard-way \
  --from-literal="mykey=mydata"
```

Print a hexdump of the `kubernetes-the-hard-way` secret stored in etcd:

```shell
CONTROLLER="controller-0"
PUBLIC_IP_ADDRESS=$(az network public-ip show -g kubernetes \
  -n ${CONTROLLER}-pip --query "ipAddress" -otsv | tr -d '\r')

ssh kuberoot@${PUBLIC_IP_ADDRESS} \
  "sudo ETCDCTL_API=3 etcdctl get \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.pem \
  --cert=/etc/etcd/kubernetes.pem \
  --key=/etc/etcd/kubernetes-key.pem\
  /registry/secrets/default/kubernetes-the-hard-way | hexdump -C"
```

> output

```shell
00000000  2f 72 65 67 69 73 74 72  79 2f 73 65 63 72 65 74  |/registry/secret|
00000010  73 2f 64 65 66 61 75 6c  74 2f 6b 75 62 65 72 6e  |s/default/kubern|
00000020  65 74 65 73 2d 74 68 65  2d 68 61 72 64 2d 77 61  |etes-the-hard-wa|
00000030  79 0a 6b 38 73 3a 65 6e  63 3a 61 65 73 63 62 63  |y.k8s:enc:aescbc|
00000040  3a 76 31 3a 6b 65 79 31  3a d2 59 80 51 d6 89 4e  |:v1:key1:.Y.Q..N|
00000050  ce 9d f8 40 fa 43 cf 0b  f7 90 33 9f 94 24 bd 29  |...@.C....3..$.)|
00000060  6b 2b 3f 11 9f bf 29 06  fc ee 14 7f 88 c6 85 1d  |k+?...).........|
00000070  90 ae 44 d8 72 d2 12 20  63 75 bd 5d 66 d5 8a 30  |..D.r.. cu.]f..0|
00000080  14 32 fb 36 da ea 3f 02  94 96 57 27 fe 0b e2 82  |.2.6..?...W'....|
00000090  4d aa c4 24 34 f8 49 bf  e5 59 9f a3 6f 6b f5 da  |M..$4.I..Y..ok..|
000000a0  4b 2f d2 24 3c ff 7f 09  b7 d2 84 ed 43 b8 96 91  |K/.$<.......C...|
000000b0  c8 9f 74 32 31 3e 78 2b  18 ce f2 62 ed 6e 1d 1e  |..t21>x+...b.n..|
000000c0  08 23 88 92 41 1b aa 07  f4 d8 32 1d 51 b1 df ce  |.#..A.....2.Q...|
000000d0  90 81 99 6c e7 26 d2 1b  b5 26 83 3d e5 70 17 97  |...l.&...&.=.p..|
000000e0  fc cb 73 b9 90 47 65 29  bd aa 45 ef 44 4a a9 58  |..s..Ge)..E.DJ.X|
000000f0  8b 6a b3 e0 43 ed e3 04  ae 9b 03 aa 16 80 3b 0d  |.j..C.........;.|
00000100  d8 af 33 fe b6 6c 9e 3b  13 0a 91 ec c6 0d 7a d7  |..3..l.;......z.|
00000110  80 8f c6 a5 8b 2a 30 67  b2 0a db 4c a8 b7 a6 4d  |.....*0g...L...M|
00000120  6d de 47 1b be 20 1e bb  87 e1 19 8d df 25 42 ac  |m.G.. .......%B.|
00000130  e7 93 d4 ad 00 e4 db fd  a6 99 7f 16 1e 01 5f f3  |.............._.|
00000140  c1 f4 cf d7 4d 05 eb a8  a8 50 7e 48 bb 68 2c 43  |....M....P~H.h,C|
00000150  e2 2f e3 18 b4 18 dd 04  b4 0a                    |./........|
0000015a
```

The etcd key should be prefixed with `k8s:enc:aescbc:v1:key1`, which indicates the `aescbc` provider was used to encrypt the data with the `key1` encryption key.

## Deployments

In this section you will verify the ability to create and manage [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/).

Create a deployment for the [nginx](https://nginx.org/en/) web server:

```shell
kubectl create deployment nginx --image=nginx
```

List the pod created by the `nginx` deployment:

```shell
kubectl get pods -l app=nginx
```

> output

```shell
NAME                     READY   STATUS    RESTARTS   AGE
nginx-86c57db685-vj6v9   1/1     Running   0          17m
```

### Port Forwarding

In this section you will verify the ability to access applications remotely using [port forwarding](https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/).

Retrieve the full name of the `nginx` pod:

```shell
POD_NAME=$(kubectl get pods -l app=nginx -o jsonpath="{.items[0].metadata.name}")
```

Forward port `8080` on your local machine to port `80` of the `nginx` pod:

```shell
kubectl port-forward $POD_NAME 8080:80
```

> output

```shell
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80
```

In a new terminal make an HTTP request using the forwarding address:

```shell
curl --head http://127.0.0.1:8080
```

> output

```shell
HTTP/1.1 200 OK
Server: nginx/1.23.4
Date: Sun, 02 Apr 2023 22:02:28 GMT
Content-Type: text/html
Content-Length: 615
Last-Modified: Tue, 28 Mar 2023 15:01:54 GMT
Connection: keep-alive
ETag: "64230162-267"
Accept-Ranges: bytes
```

Switch back to the previous terminal and stop the port forwarding to the `nginx` pod:

```shell
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80
Handling connection for 8080
^C
```

### Logs

In this section you will verify the ability to [retrieve container logs](https://kubernetes.io/docs/concepts/cluster-administration/logging/).

Print the `nginx` pod logs:

```shell
kubectl logs $POD_NAME
```

> output

```shell
127.0.0.1 - - [02/Apr/2023:22:02:28 +0000] "HEAD / HTTP/1.1" 200 0 "-" "curl/7.87.0" "-"
```

### Exec

In this section you will verify the ability to [execute commands in a container](https://kubernetes.io/docs/tasks/debug-application-cluster/get-shell-running-container/#running-individual-commands-in-a-container).

Print the nginx version by executing the `nginx -v` command in the `nginx` container:

```shell
kubectl exec -ti $POD_NAME -- nginx -v
```

> output

```shell
nginx version: nginx/1.23.4
```

## Services

In this section you will verify the ability to expose applications using a [Service](https://kubernetes.io/docs/concepts/services-networking/service/).

Expose the `nginx` deployment using a [NodePort](https://kubernetes.io/docs/concepts/services-networking/service/#nodeport) service:

```shell
kubectl expose deployment nginx --port 80 --type NodePort
```

> The LoadBalancer service type can not be used because your cluster is not configured with [cloud provider integration](https://kubernetes.io/docs/concepts/cluster-administration/cloud-providers/#azure). Setting up cloud provider integration is out of scope for this tutorial.

Retrieve the node port assigned to the `nginx` service:

```shell
NODE_PORT=$(kubectl get svc nginx \
  --output=jsonpath='{range .spec.ports[0]}{.nodePort}')
```

Create a firewall rule that allows remote access to the `nginx` node port:

```shell
az network nsg rule create -g kubernetes \
  -n kubernetes-allow-nginx \
  --access allow \
  --destination-address-prefix '*' \
  --destination-port-range ${NODE_PORT} \
  --direction inbound \
  --nsg-name kubernetes-nsg \
  --protocol tcp \
  --source-address-prefix '*' \
  --source-port-range '*' \
  --priority 1002
```

Retrieve the external IP address of a worker instance:

```shell
EXTERNAL_IP=$(az network public-ip show -g kubernetes \
  -n worker-0-pip --query "ipAddress" -otsv | tr -d '\r')
```

Make an HTTP request using the external IP address and the `nginx` node port:

```shell
curl -I http://$EXTERNAL_IP:$NODE_PORT
```

> output

```shell
HTTP/1.1 200 OK
Server: nginx/1.23.4
Date: Sun, 02 Apr 2023 22:03:48 GMT
Content-Type: text/html
Content-Length: 615
Last-Modified: Tue, 28 Mar 2023 15:01:54 GMT
Connection: keep-alive
ETag: "64230162-267"
Accept-Ranges: bytes
```

Next: [Dashboard Configuration](14-dashboard.md)
