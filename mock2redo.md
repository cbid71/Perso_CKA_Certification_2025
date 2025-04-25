1: 
Très basique


2 :

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: logging-deployment
  namespace: logging-ns
  labels:
    app: logging
spec:
  replicas: 1
  selector:
    matchLabels:
      app: logging
  template:
    metadata:
      labels:
        app: logging
    spec:
      containers:
      - name: app-container
        image: busybox:latest
        command: 
        - sh
        - -c
        - "while true; do echo 'Log entry' >> /var/log/app/app.log; sleep 5; done" 
        volumeMounts:
        - mountPath: /var/log/app
          name: log-volume
      - name: log-agent
        image: busybox
        command:
        - tail
        - -f
        - /var/log/app/app.log
        volumeMounts:
        - mountPath: /var/log/app
          name: log-volume
      volumes:
      - name: log-volume
        emptyDir:
          sizeLimit: 100Mi
```


kubectl -n logging-ns exec logging-deployment-79996f5bd9-hg6v6 app-container -- cat /var/log/app/app.log

3:

Ensuite il faut rajouter un ingress pour renvoyer vers un service
Il ne faut pas oublier l'ingressclass

`kubectl get ingressclass : ` retourne `nginx`

Puis le ingress en question :

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: webapp-ingress
  namespace: ingress-ns
spec:
  ingressClassName: nginx
  rules:
  - host: kodekloud-ingress.app
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: webapp-svc
            port:
              number: 80
```



4 : 


```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.16
```

Puis kubectl get deploy --> ce qui permet de passer de nginx:1.16 à 1.17

--> normalement le sujet se suffit à lui-même et le deployement est redémarré en douceur


5 : PAS FINI

```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
  namespace: development
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods"]
  verbs: ["create", "list", "update", "delete"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: developer-rb
  namespace: development
subjects:
- kind: User
  name: john # "name" is case sensitive
  apiGroup: rbac.authorization.k8s.io
roleRef:
  # "roleRef" specifies the binding to a Role / ClusterRole
  kind: Role #this must be Role or ClusterRole
  name: developer # this must match the name of the Role or ClusterRole you wish to bind to
  apiGroup: rbac.authorization.k8s.io
---
## cat  /root/CKA/john.csr | base64 | tr -d '\n' >> 5.yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: john
  namespace: developer
spec:
  signerName: example.com/mysigner
  request: <base64csr>
  usages:
    - "client auth"
```

Et en théorie derriere il aurait fallu faire ça : 

```
kubectl certificate approve <certificate-signing-request-name>
kubectl certificate approve john
```

Et pour vérifier 

```
kubectl auth can-i update pods --as=john --namespace=development
```

6 : 

Marche pas, je suis censé avec une résolution DNS depuis busybox vers nginx-resolver, mais non

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-resolver
spec:
  hostname: nginx-resolver
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-resolver-service
spec:
  selector:
    app.kubernetes.io/name: nginx-resolver
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
---
apiVersion: v1
kind: Pod
metadata:
  name: busybox
spec:
  containers:
  - name: bsb
    image: busybox:1.28
    command:
    - sh
    - -c
    - "sleep 200000"
```


7:

Pod static sur un certain node

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-critical
spec:
  containers:
  - name: nginx-critical
    image: nginx
    ports:
    - containerPort: 80
```
Puis on met sur le node
```
scp 7.yaml cluster1-node01:/etc/kubernetes/manifests/pod.yaml
```


8:

```
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: webapp-hpa
  namespace: backend
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: backend-deployment
  minReplicas: 3
  maxReplicas: 15
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 65
```


9:

penser à se ssh sur le bon controlplane avant
Il a juste fallu redémarrer le service kubelet sur le controlplane

10:

CHanger la gateway pour prendre les infos du port 443
Ne pas oublier le certificat
```
# web-gateway.yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: web-gateway
  namespace: cka5673
spec:
  gatewayClassName: kodekloud
  listeners:
    - name: https
      protocol: HTTPS
      port: 443
      hostname: kodekloud.com
      tls:
        certificateRefs:
          - name: kodekloud-tls
```



11:

Hey ! c'est bon là ! de l'extraction via jq

kubectl get deploy -n <NAMESPACE> <DEPLOYMENT-NAME> -o json | jq -r '.spec.template.spec.containers[].image'

helm uninstall <RELEASE-NAME> -n <NAMESPACE>

12 :

appliquer la netpol3

kubectl apply -f /root/net-pol-3.yaml

13 :

Voir le replicaset 

kubectl describe replicasets backend-api-7977bfdbd5

Et aligner le déploiement sur les éléments du replicaset


14 :

Il faut déployer Calico

https://raw.githubusercontent.com/projectcalico/calico/v3.29.3/manifests/tigera-operator.yaml

Et déployer le custom resource

curl https://raw.githubusercontent.com/projectcalico/calico/v3.29.2/manifests/custom-resources.yaml -O


Et... je sais pas.
