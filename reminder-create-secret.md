# create secret

example

login : admin
password : mygreatpassword

echo "admin | base64
YWRtaW4K
echo "mygreatpassword" | base64
bXlncmVhdHBhc3N3b3JkCg==

---

apiVersion: v1
kind: Secret
metadata:
  name: MySecret
data:
  login: YWRtaW4K
  password: bXlncmVhdHBhc3N3b3JkCg==


OU

kubectl create secret MySecret --from-literal=admin=admin --from-literal=password=mygreatpassword

---

```
kubectl create secret <name> <source>				# create standard secret
kubectl create secret tls <name> <source>			# create secret to store TLS certificate

kubectl create secret tls mon-certificat \
  --cert=chemin/vers/cert.pem \
  --key=chemin/vers/key.pem


kubectl create secret docker-registry <name> <source>		# Store elements to connect to a private docker registry

kubectl create secret docker-registry mon-registre-secret \
  --docker-username=monuser \
  --docker-password=monmotdepasse \
  --docker-email=monemail@example.com \
  --docker-server=https://index.docker.io/v1/
```

Rappel pour utiliser le secret type "docker registry"

```
apiVersion: v1
kind: Pod
metadata:
  name: mon-pod-prive
spec:
  containers:
    - name: mon-container
      image: monregistryprivé.io/monimage:tag
  imagePullSecrets:
    - name: mon-registre-secret
```
