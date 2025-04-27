CKA-2025

**COURSE** https://github.com/kodekloudhub/certified-kubernetes-administrator-course


Disclaimer : I'm not the creator of thoses tests, I'm working for the certification myself with them.

Certified Kubernetes Administrator: https://www.cncf.io/certification/cka/

Exam Curriculum (Topics): https://github.com/cncf/curriculum

Candidate Handbook: https://www.cncf.io/certification/candidate-handbook

Exam Tips: 
- http://training.linuxfoundation.org/go//Important-Tips-CKA-CKAD
- https://docs.linuxfoundation.org/tc-docs/certification/tips-cka-and-ckad
- https://github.com/kodekloudhub/certified-kubernetes-administrator-course

Get certification :
- https://training.linuxfoundation.org/certification/certified-kubernetes-administrator-cka/

K3S : https://k3s.io/

Other labs :
- https://uklabs.kodekloud.com/courses/labs-certified-kubernetes-administrator-with-practice-tests/

- killersh lab : https://killer.sh/dashboard

# Allowed sources

`https://docs.linuxfoundation.org/tc-docs/certification/certification-resources-allowed#certified-kubernetes-administrator-cka-and-certified-kubernetes-application-developer-ckad`

Note : `http://discuss.kubernetes.io|discuss.kubernetes.io` is NOT ALLOWED and might lead to a warning

# Other source ?

https://www.udemy.com/course/certified-kubernetes-administrator/

# To be done

- read the exam tips
- read the candidate handbook
- get other mocks exams by udemy
- get killerSH tests
- read the candidate handbook
- read all my notes in this repo
- linux foundation trainings ?

# know those two parameters for metrics-server, they are useful !
```
    - /metrics-server
    - --kubelet-insecure-tls
    - --kubelet-preferred-address-types=InternalIP
```

# know about

- admission controller and admission API
	**they are Kubernetes components that intercept requests to the Kubernetes API server after authentication/authorization but before persistence (saving to etcd).**
- imperative commands like kubectl run, kubectl create -> convert ingress to gateway API /!\
- know about api gateway which replaced ingress  + ingress to api gateway
- mutating webhook
	** a type of Admission Controller that can modify (mutate) an object before it is saved to Kubernetes.**
- Validating webhook
	**Can only accept or reject the request, but cannot change it.**
- do exams on KillerSH (via PSIExam)
- liveness probe
- readiness probe
- gateway API and TLS /!\
- know to properly investigate kubelet
- know role, and role bindings
- Being able to deploy a network add (CNI) as calico for example
- Doing mock exams
	`https://www.udemy.com/course/real-cka-exam-questions-certified-kubernetes-administrator/`


# Command to be known
```
kubectl create namespace new-namespace
kubectl create secret
kubectl create configmap
kubectl run nginx --image=nginx  # to create a pod
kubectl scale...
kubectl create clusterrole storage-admin --resource=persistentvolumes,storageclasses --verb=list,create,get,watch
kubectl api-resources # to know the group in apiGroup, for example for clusterrole
kubectl create clusterrolebinding michelle-storage-admin --user=michelle --clusterrole=storage-admin
kubectl get storageclass --as michelle
kustomize create --automatic  # not sure if kustomize really need to be purely known
```
