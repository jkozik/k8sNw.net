# k8sNw.net

Setup napervilleweather.net in a kubernetes cluster as deployment named nwnet; expose as service nwnet; connect to the internet as napervilleweather.net through an ingress called nwnet-ingress. Get realtime weather data from an NFS mount on my local host through the PV/PVC named nwcom-persistent-storage -- note, this is the same PV/PVC used by [k8sNw.com](https://github.com/jkozik/k8sNw.com). 

Source image [InstallNW.net](https://github.com/jkozik/InstallNW.net)

# Build image, put on docker hub
NapervilleWeather.net is running in a docker container on my host, directly -- not in a VM.  To make it run in my kubernetes cluster, I need to push the image to my dockerhub repository.  The kubernetes deployment resource has an "image" field that triggers a pull from dockerhub.
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
# Clone github repository, apply yaml files

Get the yaml manifest files and apply them.  Verify that the pod, deployment, service and ingress are successfully running.
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
# Verify that pod, service, deployment and ingress resources running
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
## HomeLAN NATing from external IP address / port 80 to cluster's LAN address and ingress controller's port number.
So, on my home LAN http://192.168.100.174:30410 is where incoming web traffic enters.  The ingress controller parses for napervilleweather.com and redirects the traffic to the nwcom service. I have an external IP address for my home LAN that gets NAT'd by my home router to 192.168.100.174.  On that NAT box, I map port 80 to port 30410. As you can see from the get ingress command, I also am running a wordpress application with another URL, but also mapped to the same IP address and port number.  The ingress control does a reverse proxy function and splits the wordpress traffic off to the nginx-wordpress-ingress resource.
```
[jkozik@dell2 k8sNw.net]$ kubectl -n ingress-nginx get service
NAME                                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller             NodePort    10.97.70.29     <none>        80:30140/TCP,443:30023/TCP   22d
ingress-nginx-controller-admission   ClusterIP   10.111.250.10   <none>        443/TCP
```
## Check the web page http://napervilleweather.net
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
```
## napervilleweather.net/weewx
In the repository [weewx](https://github.com/jkozik/weewx), I added an ingress to permit the weewx based weather software run as a separate path.  Thus [napervilleweather.net/weewx](https://napervilleweather.net/weewx) will display the same weather data but under the weewx weather software tool and graphing mechanism.  

Note: to do this, I updated the [nwnet-weewx-ingress.yaml](nwnet-weewx-ingress.yaml) file in the weewx repository to replace the one in this repository.  See the contents of it below:
```
[jkozik@dell2 weewx]$ cat nwnet-weewx-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nwnet-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$1
    nginx.ingress.kubernetes.io/use-regex: "true"
    nginx.ingress.kubernetes.io/configuration-snippet: |
      rewrite ^(/weewx)$ $1/ redirect;
      rewrite ^(/nwnet)$ $1/ redirect;
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  tls:
  - hosts:
    - napervilleweather.net
    secretName: napervilleweather-net-tls
  rules:
  - host: napervilleweather.net
    http:
      paths:
      - path: /nwnet/(.*)
      #- path: /nwnet(/|$)(.*)
        pathType: Prefix
        backend:
          service:
            name: nwnet
            port:
              number: 80
     # - path: /weewx(/|$)(.*)
      #- path: /weewx(/|$)(.*)
      - path: /weewx/(.*)
        pathType: Prefix
        backend:
          service:
            name: weewx
            port:
              number: 80
      - path: /(.*)
        pathType: Prefix
        backend:
          service:
            name: nwnet
            port:
              number: 80

[jkozik@dell2 weewx]$
```
## Ingress --> HttpRoute
I have installed the Gateway API on this cluster.  It is called ` Gateway running in `envoy-gateway-system` namespace.

The HTTPRoute yaml points napervilleweather.net and www.naperweather.net from my Cloudflare tunnel to port 30458.

Ingress has been depricated.
