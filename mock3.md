Mock test 3

https://uklabs.kodekloud.com/topic/mock-exam-3-3/

Sur chaque node :

sysctl net.ipv4.ip_forward=1
sysctl  net.bridge.bridge-nf-call-iptables=1


```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: pvviewer
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: pvviewer-role
rules:
- apiGroups: [""]
  resources: ["persistentvolumes"]
  verbs: ["get", "watch", "list"]
# Apparemment pour le test, seul list est pertinent
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: pvviewer-role-binding
subjects:
- kind: ServiceAccount
  name: pvviewer # Name is case sensitive
  namespace: default
roleRef:
  kind: ClusterRole
  name: pvviewer
---
apiVersion: v1
kind: Pod
metadata:
  name: pvviewer
spec:
  serviceAccountName: pvviewer
  containers:
  - name: redis
    image: redis:latest
```

```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: rancher-sc
provisioner: rancher/local-path
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: true
```

Sur la question suivante il fallait éditer un déploiement déjà existant pour ajouter deux nouvelles variables d'env

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: cm-namespace
data:
  ENV: production
  LOG_LEVEL: info
```


```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: cm-webapp
  name: cm-webapp
  namespace: cm-namespace
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: cm-webapp
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: cm-webapp
    spec:
      containers:
      - image: nginx
        imagePullPolicy: Always
        name: nginx
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
        env:
        - name: ENV
          valueFrom:
           configMapKeyRef:
              name: app-config
              key: ENV
        - name: LOG_LEVEL
          valueFrom:
           configMapKeyRef:
              name: app-config
              key: LOG_LEVEL
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
```

```
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: low-priority
value: 50000
```


Ensuite il fallait ajouter le champs `spec.Priorityclass: low-priority` dans un déploiement préexistant


Puis à présent créer un netpol

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: ingress-to-nptest
  namespace: default
spec:
  podSelector:
    matchLabels:
      run: np-test-1
  policyTypes:
  - Ingress
  ingress:
  - ports:
    - protocol: TCP
      port: 80
```


 A présent on parle de node,

il faut taint node01 pour qu'il soit unschedulable, on pourrait uncordon, mais ce n'est pas le but ici

```
kubectl taint nodes node01 env_type=production:NoSchedule

and for the example to untaint
kubectl taint nodes node1 key1=value1:NoSchedule
kubectl taint nodes node01 key1:NoSchedule-

```

à partir de là : les pods qui n'ont pas la toleration ne seront pas schedulés dessus.


```
apiVersion: v1
kind: Pod
metadata:
  name: dev-redis
spec:
  containers:
  - name: redis
    image: redis:alpine
---
apiVersion: v1
kind: Pod
metadata:
  name: prod-redis
spec:
  tolerations:
  - key: "env_type"
    operator: "Equal"
    value: "production"
    effect: "NoSchedule"
  containers:
  - name: redis
    image: redis:alpine
```

Ensuite un souci de PVC, c'était un problème avec l'accessmode  --> de readwritemany à readwriteonce

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"PersistentVolumeClaim","metadata":{"annotations":{},"name":"app-pvc","namespace":"storage-ns"},"spec":{"accessModes":["ReadWriteMany"],"resources":{"requests":{"storage":"1Gi"}}}}
  creationTimestamp: "2025-04-23T14:20:16Z"
  finalizers:
  - kubernetes.io/pvc-protection
  name: app-pvc
  namespace: storage-ns
  resourceVersion: "8553"
  uid: 4bc01031-efe8-4cc6-90c9-d6c390f5da9d
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  volumeMode: Filesystem
```


Ensuite un problème de config de fichier kuke/super.config

`diff ~/.kube/config /root/CKA/super.kubeconfig` expose bien la difference


Ensuite un souci de scaling :

kubectl scale deploy nginx-deploy --replicas=3
Faudra pas oublier de penser à mettre à jour le déploiement qui plante de base


Ensuite un HPA basé sur une custom Metric

ATTENTION : bien lire le sujet, un fichier de base a été donné
```
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: api-hpa
  namespace: api
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api-deployment
  minReplicas: 1
  maxReplicas: 20
  metrics:
  - type: Pods
    pods:
      metric:
        name: requests_per_second
      target:
        type: AverageValue
        averageValue: "1000"
```

Pour la demande de couper le flux en deux

```
apiVersionm: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: web-route
  namespace: default
spec:
  parentRefs:
    - name: web-gateway
      namespace: default
  rules:
    - matches:
        - path:
            type: PathPrefix
            value: /
      backendRefs:
        - name: web-service
          port: 80
          weight: 80
        - name: web-service-v2
          port: 80
          weight: 20
```


Pour les questions Helm

```
helm ls -n default
helm lint ./new-version
helm install --generate-name ./new-version
helm uninstall webpage-server-01
# après c'est un peu bête, j'aurais tendance à monter de version, puisque c'est comme ça que c'est censé fonctionner m'enfin...
```



Pour la question 14 que je n'arriverai pas parce que le jsonpath m'ennuie : 

```
kubectl get node -o jsonpath='{.items[0].spec.podCIDR}' > /root/pod-cidr.txt
```
