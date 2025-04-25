# various stuff

## Feature gate

parameters that you can configure on your cluster components
  https://kubernetes.io/docs/reference/command-line-tools-reference/feature-gates/

## Seccomp

a bunch of various definitions that will manage the restriction of syscalls of your pods, or nodes, or anything
  https://kubernetes.io/docs/tutorials/security/seccomp/
 here an explanation of various fields and profiles  https://kubernetes.io/docs/reference/node/seccomp/

```
apiVersion: v1
kind: Pod
metadata:
  name: default-pod
  labels:
    app: default-pod
spec:
  securityContext:
    seccompProfile:
      type: RuntimeDefault
  containers:
  - name: test-container
    image: hashicorp/http-echo:1.0
    args:
    - "-text=just made some more syscalls!"
    securityContext:
      allowPrivilegeEscalation: false
```

With seccomp, you point your pods on a file or a profile type (here in the example a profile `RuntimeDefault` which is one of the default profile provided)
You can create your own profile with various syscall allowed or forbidden

myprofile.json
```
{
  "defaultAction": "SCMP_ACT_ERRNO",
  "defaultErrnoRet": 38,
  "syscalls": [
    {
      "names": [
        "adjtimex",
        "alarm",
        "bind",
        "waitid",
        "waitpid",
        "write",
        "writev"
      ],
      "action": "SCMP_ACT_ALLOW"
    }
  ]
}
```

Those json files must be stored on all nodes in a specific directory `/var/lib/kubelet/seccomp/profiles`, the good idea is to provide those files through a daemonset

```
---
kubectl create configmap seccomp-profile-config --from-file=myprofile.json
THEN
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: seccomp-distributor
spec:
  selector:
    matchLabels:
      name: seccomp-distributor
  template:
    metadata:
      labels:
        name: seccomp-distributor
    spec:
      containers:
      - name: copy-seccomp
        image: busybox
        command: ["/bin/sh", "-c"]
        args:
          - cp /src/myprofile.json /dst/myprofile.json
        volumeMounts:
        - name: seccomp-profile
          mountPath: /dst
        - name: profile-src
          mountPath: /src
        securityContext:
          privileged: true
      volumes:
      - name: seccomp-profile
        hostPath:
          path: /var/lib/kubelet/seccomp/profiles
          type: DirectoryOrCreate
      - name: profile-src
        configMap:
          name: seccomp-profile-config
```

Then you can use those seccomps profiles on a pod : 

```
apiVersion: v1
kind: Pod
metadata:
  name: pod
spec:
  containers:
  - name: container
    image: docker.io/library/debian:stable
    securityContext:
      seccompProfile:
        type: Localhost
        localhostProfile: myprofile.json

```

## Various stuff about 1.29

https://www.youtube.com/watch?v=yCkQgKVwSVU

