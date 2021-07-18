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

```
[jkozik@dell2 k8sNw.net]$ kubectl get deployment,service,pod,ingress,pv,pvc
NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nwcom                 1/1     1            1           20h
deployment.apps/nwnet                 1/1     1            1           2m59s

NAME                      TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
service/nwcom             NodePort    10.100.137.46    <none>        80:31241/TCP   20h
service/nwnet             NodePort    10.108.118.250   <none>        80:30557/TCP   2m57s

NAME                                       READY   STATUS    RESTARTS   AGE
pod/nwcom-5d55cd57b6-jjc4t                 1/1     Running   0          20h
pod/nwnet-77785d54c8-kt9mm                 1/1     Running   0          2m59s

NAME                                                CLASS    HOSTS                     ADDRESS           PORTS   AGE
ingress.networking.k8s.io/nwcom-ingress             <none>   napervilleweather.com     192.168.100.174   80      20h
ingress.networking.k8s.io/nwnet-ingress             <none>   napervillesweather.net    192.168.100.174   80      2m59s

NAME                                            CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                                  STORAGECLASS   REASON   AGE
persistentvolume/nwcom-persistent-storage       1Gi        ROX            Retain           Bound    default/nwcom-persistent-storage       nfs                     20h

NAME                                                 STATUS   VOLUME                         CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/nwcom-persistent-storage       Bound    nwcom-persistent-storage       1Gi        ROX            nfs            20h
```

```
</html>[jkozik@dell2 k8sNw.net]$ curl -H "Host: napervilleweather.net" 192.168.100.173:30140 | head
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
<!DOCTYPE html>
<html lang="en">
<head>
  <title>Naperville, IL USA Smart Home Weather Station</title>
  <meta content="Smart Home weather station providing current weather conditions for Naperville, IL USA" name="description">
  <meta content="website" property="og:type">
  <meta content="7 days" name="revisit-after">
  <meta content="web" name="distribution">
  <meta content="Naperville, IL USA
100  8026    0  8026    0     0    99k      0 --:--:-- --:--:-- --:--:--  101k
curl: (23) Failed writing body (166 != 4090)
[jkozik@dell2 k8sNw.net]$
```
