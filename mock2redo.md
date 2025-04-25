1: 
```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-sc
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
provisioner: kubernetes.io/no-provisioner
allowVolumeExpansion: true
volumeBindingMode: WaitForFirstConsumer
```



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
  verbs: ["create", "list", "get", "update", "delete"]
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
  name: john-developer
  namespace: developer
spec:
  signerName: example.com/mysigner
  request: <base64csr>
  usages:
    - "client auth"
```

Et en théorie derriere il aurait fallu faire ça : 

```
kubectl get csr
kubectl certificate approve <certificate-signing-request-name>
kubectl certificate approve john
```

Et pour vérifier 

```
kubectl get pods -n development --as='john'
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

kubectl exec busybox -- sh -c "nslookup nginx-resolver"


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

Puis penser à installer metrics server
Puis penser à régler le metrics server en non sécurisé

```
    - --kubelet-insecure-tls
    - --kubelet-preferred-address-types=InternalIP
```


9:

penser à se ssh sur le bon controlplane avant
Il a juste fallu redémarrer le service kubelet sur le controlplane

Et tant qu'à faire, l'api server était pété donc je l'ai réparé :

`kubectl describe pod kube-apiserver-cluster2-controlplane -n kube-system`

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

Hey ! c'est bon là ! de l'extraction via jq, et la solution ne fonctionne pas :
```
kubectl get deploy -n <NAMESPACE> <DEPLOYMENT-NAME> -o json | jq -r '.spec.template.spec.containers[].image'
helm uninstall <RELEASE-NAME> -n <NAMESPACE>
```

EN QUI MARCHE mais difficile à sortir en exam : 

```
kubectl get deployments --all-namespaces -o json | jq -r '.items[] | "\(.metadata.namespace):\(.metadata.name) -> \(.spec.template.spec.containers[]?.image)"' | grep -i "kodekloud/webapp-color:v1"
helm list -n atlanta-page-04
helm uninstall atlanta-page-apd -n atlanta-page-04
```


12 :

appliquer la netpol3

kubectl apply -f /root/net-pol-3.yaml

13 :

Voir le replicaset 

kubectl describe replicasets backend-api-7977bfdbd5

Et aligner le déploiement sur les éléments du replicaset

---> Il faut que la somme des requests soit strictement inférieur à la limite imposée par le replicaset

```
      resources:
          requests:
            cpu: 100m
            memory: 100Mi
```


14 :

Il faut récupérer Calico/Tigera

curl https://raw.githubusercontent.com/projectcalico/calico/v3.29.3/manifests/tigera-operator.yaml -O

Et déployer le custom resource

curl https://raw.githubusercontent.com/projectcalico/calico/v3.29.2/manifests/custom-resources.yaml -O

Puis installer Tigera

kubectl create -f tigera-operator.yaml    # Attention ! pas apply sinon ça plante

Puis installer les custom resources

kubectl create -f custom-resources.yaml

Apparemment il manque une histoire de CIDR, la correction a planté sur ce point mais en fouillant un peu je tombe sur ce résultat qui pourrait fonctionner :

```
apiVersion: crd.projectcalico.org/v1
kind: IPPool
metadata:
  name: custom-pool-172-17
spec:
  cidr: 172.17.0.0/16
  ipipMode: Always
  natOutgoing: true
  disabled: false
``` 

Problème, je ne peux pas installer la CRD sur le yaml tigera-operator.yaml 
The CustomResourceDefinition "installations.operator.tigera.io" is invalid: metadata.annotations: Too long: may not be more than 262144 bytes


Et... je sais pas.
