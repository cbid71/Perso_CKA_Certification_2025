
https://uklabs.kodekloud.com/topic/mock-exam-1-4/

## pour la première question

C'était assez dense, voici ce qui aurait dû être fait

```
apiVersion: v1
kind: Pod
metadata:
  name: mc-pod
  namespace: mc-namespace
spec:
  containers:
    - name: mc-pod-1
      image: nginx:1-alpine
      env:
        - name: NODE_NAME
          valueFrom:
            fieldRef:
              fieldPath: spec.nodeName
    - name: mc-pod-2
      image: busybox:1
      volumeMounts:
        - name: shared-volume
          mountPath: /var/log/shared
      command:
        - "sh"
        - "-c"
        - "while true; do date >> /var/log/shared/date.log; sleep 1; done"
    - name: mc-pod-3
      image: busybox:1
      command:
        - "sh"
        - "-c"
        - "tail -f /var/log/shared/date.log"
      volumeMounts:
        - name: shared-volume
          mountPath: /var/log/shared
  volumes:
    - name: shared-volume
      emptyDir: {}
```



## expose name 

kubectl get crd -o name | grep -i vert > /root/vpa-crds.txt


## Exposer (alias "créer un service") pour un déploiement :

kubectl expose deployment nginx-gateway --port 6379 --type=ClusterIP --name=messaging-service -n nginx-gateway

kubectl expose pod messaging --port 6379 --type=ClusterIP --name=messaging-service


## a propos d'orange

le pod avait une mauvaise cmd, une fois corrigée -> cela génère un fichier /tmp/balbalbla.yaml

cp /tmp/blablabla.yaml orange.yaml
kubectl replace --force -f orange.yaml


```
apiVersion: v1
kind: Service
metadata:
  name: hr-web-app-server
spec:
  selector:
    app.kubernetes.io/name: hr-web-app
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 30082
  type: NodePort
```

You can have the hostpath 

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-analytics
spec:
  capacity:
    storage: 100Mi
  accessModes:
    - ReadWriteMany
  hostPath:
    path: /pv/data-analytics
```

Pour les HPA, c'est bien mais le sujet ne transmettait pas toutes les infos, j'avais une bonne réponse, mais ci-dessous voici la réponse attendue, il me manquait la partie `metrics`:

```
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: webapp-hpa
  namespace: default
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: kkapp-deploy
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
```


pour les metrics cpu manquantes, il faut penser à ajouter le metrics server

Et il faut penser à passer le deployment en Insecure-tls sinon il ne fonctionne pas

```
        - --kubelet-insecure-tls
        - --kubelet-preferred-address-types=InternalIP
```

```
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
    name: analytics-vpa
spec:
    targetRef:
        apiVersion: "apps/v1"
        kind: Deployment
        name: analytics-deployment
    updatePolicy:
      updateMode: "Auto"
    resourcePolicy:
      containerPolicies:
        - containerName: '*'
```


```
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: web-gateway
  namespace: nginx-gateway
spec:
  gatewayClassName: nginx
  listeners:
  - name: http
    protocol: HTTP
    port: 80
```


Et je n'ai pas compris la dernière question sur le Helm, apparemment il fallait faire ça :

```
helm repo list
helm repo update kk-mock1 -n kk-ns
helm search repo kk-mock1/nginx -n kk-ns -l | head -n30		# search for available charts
helm upgrade kk-mock1 kk-mock1/nginx -n kk-ns --version=18.1.15
helm ls -n kk-ns
```


