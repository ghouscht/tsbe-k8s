# 04 - Einen Service verwenden
[Service | Kubernetes](https://kubernetes.io/docs/concepts/services-networking/service/)

Mit Hilfe von einem Service kann eine Appliatkion (welche als Set von Pods läuft) im Netzwerk verfügbar gemacht werden. Ein funktioniert wie ein Loadbalancer mit einer eigenen IP bzw. DNS Namen, welcher Netzwerktaffic zu einem Set von Pods weiterleitet.

## 0. ClusterIP Service erstellen
Verwende `kubectl create service ...` um einen Service vom Typ `clusterip` zu erstllen, verwende den Namen `hello` und den TCP Port 80, der targetPort soll ebenfalls 80 sein.

Tipp: `kubectl create service -h`

Prüfe wie folgt ob der Service erstellt wurde:

```console
# kubectl get service
NAME       TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                  AGE
hello      ClusterIP   10.107.255.82   <none>        80/TCP                   2m7s

# kubectl get endpoints
NAME       ENDPOINTS                                               AGE
hello      10.0.0.121:80,10.0.0.147:80,10.0.0.190:80               4s

# kubectl get pod -o wide -l app=hello
NAME                     READY   STATUS    RESTARTS   AGE     IP           NODE                       NOMINATED NODE   READINESS GATES
hello-685657bc5b-46c6g   1/1     Running   0          6m2s    10.0.0.121   c4t-lpc01.cloud4tsbe.lan   <none>           <none>
hello-685657bc5b-6d677   1/1     Running   0          5m52s   10.0.0.190   c4t-lpc01.cloud4tsbe.lan   <none>           <none>
hello-685657bc5b-vdrxk   1/1     Running   0          5m42s   10.0.0.147   c4t-lpc01.cloud4tsbe.lan   <none>           <none>
```

Wie du nun siehst, hat der Service eine eigene IP, die IPs des Endpoints jedoch sind die der Pods. Falls das bei dir der Fall ist, ist alles ok.

## 1. ClusterIP Service verwenden
Öffne wieder eine interaktive Shell in einem der hello Pods. Prüfe ob du mit cURL einen HTTP Request auf deine Service IP ausführen kannst, in meinem Fall 10.107.255.82. Falls das funktioniert, kannst du den selben Request wiederholen aber mit dem Service Namen: `curl http://hello` - das müsste genauso funktionieren!

Services vom Typ `ClusterIP` funktionieren nur aus dem Kubernetes Cluster heraus und sind ausserhalb vom Cluster nicht ansprechbar. Dazu gibt es den Service Typ `NodePort`, diesen verwenden wir im nächsten Schritt.

## 2. NodePort Service erstellen
Verwende wieder `kubectl create service ...` und erstelle diess mal einen Service vom Typ `nodeport` mit dem Namen `hello-np`, verwende wiederum TCP Port 80. Prüfe wiederum ob der Service erstellt wurde:

```console
# kubectl get service
NAME       TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                  AGE
hello      ClusterIP   10.107.255.82   <none>        80/TCP                   9m45s
hello-np   NodePort    10.96.171.237   <none>        80:31502/TCP             6s
```

Im Beispiel siehst du, dass Kubernetes Port 31502 ausgewählt hat, via diesem Port kannst du nun deinen hello Service auch von extern ansprechen. Du kannst das prüfen, indem du mit cURL (dieses mal vom Host, nicht in einem Pod) einen Request an diesen Port sendest. Beispiel: `curl http://localhost:31502`

## Weiterführende Links
* https://cloud.google.com/kubernetes-engine/docs/concepts/service-discovery
* https://kubernetes.io/docs/concepts/services-networking/connect-applications-service/