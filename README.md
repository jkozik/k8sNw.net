# k8sNw.net
```
[jkozik@dell2 k8sNw.net]$ docker tag jkozik/nw.net jkozik/nw.net:v1
[jkozik@dell2 k8sNw.net]$ docker push jkozik/nw.net:v1
The push refers to a repository [docker.io/jkozik/nw.net]
a1e5d4b36a6c: Pushed
199834f869d1: Pushed
4f77007a6de2: Mounted from jkozik/chw.net
8022c3879348: Mounted from jkozik/chw.net
1920409046ed: Mounted from jkozik/nw.com
1b04f74cccfe: Mounted from jkozik/nw.com
9f80175acb76: Mounted from jkozik/nw.com
4394b53db28c: Mounted from jkozik/nw.com
5b35dc81a1f5: Mounted from jkozik/nw.com
76d251b895a5: Mounted from jkozik/nw.com
7f5982d81e05: Mounted from jkozik/nw.com
ea7c1c005303: Mounted from jkozik/nw.com
df83fc87c7b6: Mounted from jkozik/nw.com
17e81ae0b70b: Mounted from jkozik/nw.com
e80100d0d319: Mounted from jkozik/nw.com
cc0f976c1376: Mounted from jkozik/nw.com
67415c00b86b: Mounted from jkozik/nw.com
ffc9b21953f4: Mounted from jkozik/nw.com
v1: digest: sha256:0bdaf68ce13c7393dc8aca9b15783b0859767c3c3ac21e8b9eb25bae9743058b size: 4082
```
```
[jkozik@dell2 ~]$ git clone https://github.com/jkozik/k8sNw.net
Cloning into 'k8sNw.net'...
remote: Enumerating objects: 11, done.
remote: Counting objects: 100% (11/11), done.
remote: Compressing objects: 100% (8/8), done.
remote: Total 11 (delta 0), reused 5 (delta 0), pack-reused 0
Unpacking objects: 100% (11/11), done.

[jkozik@dell2 ~]$ cd k8sNw.net
[jkozik@dell2 k8sNw.net]$ ls
nwnet-deploy.yml  nwnet-ingress.yml  nwnet-svc.yml  README.md
[jkozik@dell2 k8sNw.net]$ kubectl apply -f .
deployment.apps/nwnet created
ingress.networking.k8s.io/nwnet-ingress created
service/nwnet created
```

