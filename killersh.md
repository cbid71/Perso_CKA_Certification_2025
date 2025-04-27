https://killer.sh/course/preview/e84d0e31-4fff-4c42-8afd-be1bdbc0d994

Export iptable rules related to a service

```
kubectl get services -A
...
my-service
...
#on each node/worker
iptable-save | grep -i my-service
```

create pod, service and change cidr for services to 11.96.0.0/12

--> change the CIDR

```
# on the controlplane
nano /etc/kubernetes/manifests/kube-apiserver.yaml
```

```
# cka9412:/etc/kubernetes/manifests/kube-apiserver.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    component: kube-apiserver
    tier: control-plane
  name: kube-apiserver
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-apiserver
    - --advertise-address=192.168.100.21
...
    - --service-account-key-file=/etc/kubernetes/pki/sa.pub
    - --service-cluster-ip-range=11.96.0.0/12 			            # change
    - --tls-cert-file=/etc/kubernetes/pki/apiserver.crt
    - --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
...
```

--> create pod and service

```
kubectl run check-ip --image=httpd:2-alpine
kubectl expose pod check-ip --name check-ip-service --port=80 --targetport=80	# note : targetport ip optional since port and targetport have same value
```
