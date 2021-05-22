# 04 - Einen Service verwenden
[Service | Kubernetes](https://kubernetes.io/docs/concepts/services-networking/service/)

Mit Hilfe von einem Service kann eine Applikation (welche als Set von Pods läuft) im Netzwerk verfügbar gemacht werden. Ein funktioniert wie ein Loadbalancer mit einer eigenen IP bzw. DNS Namen, welcher Netzwerktaffic zu einem Set von Pods weiterleitet.

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

## 2. Service Discovery testen
Öffne erneut eine interaktive Shell in einem der hello Pods. Schaue dir im Container die Datei `/etc/resolv.conf` an, was fällt dir auf?
Notiere Einstellungen die du nicht kennst. Prüfe mit `getent hosts hello` ob der soeben erstellte Service aufgelöst werden kann. Prüfe das selbe mit folgenden Namen:

* kubernetes
* kubernetes.default
* kube-dns
* kube-dns.kube-system
* www.google.ch

Welche funktionieren? Welche nicht? Warum?

## 3. NodePort Service erstellen
Verwende wieder `kubectl create service ...` und erstelle diess mal einen Service vom Typ `nodeport` mit dem Namen `hello-np`, verwende wiederum TCP Port 80. Prüfe wiederum ob der Service erstellt wurde:

```console
# kubectl get service
NAME       TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                  AGE
hello      ClusterIP   10.107.255.82   <none>        80/TCP                   9m45s
hello-np   NodePort    10.96.171.237   <none>        80:31502/TCP             6s
```

Im Beispiel siehst du, dass Kubernetes Port 31502 ausgewählt hat, via diesem Port kannst du nun deinen hello Service auch von extern ansprechen. Du kannst das prüfen, indem du mit cURL (dieses mal vom Host, nicht in einem Pod) einen Request an diesen Port sendest. Beispiel: `curl http://localhost:31502`

## 4. LoadBalancer Service erstellen
Normalerweise wird dieser Servicetyp nur bei Cloud Providern unterstützt. Dank [metallb](https://github.com/metallb/metallb) können Services vom Typ `LoadBalancer` auch in "Bare-metal" Installationen verwendet werden.

1. Führe die Installation anhand der [Anleitung](https://metallb.universe.tf/installation/) durch.
2. Metallb benötigt noch eine [Konfiguration](https://metallb.universe.tf/configuration/), erstelle eine neue Datei `metallb-conf.yaml` mit folgendem Inhalt:
```yaml
---
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      - 192.168.210.200-192.168.210.220
```
3. Appliziere die Konfiguration mit `kubectl apply -f metallb-conf.yaml`.

Der Cluster ist nun vorberietet, damit wir einen funktionierenden Service vom Typ `LoadBalancer` erstellen können. Erstelle mit `kubectl create service loadbalancer hello-lb --tcp=80:80` einen `LoadBalancer` Service. Mit `kubectl get svc` solltest du nun sehen, dass dem Service eine externe IP Adresse zugewiesen wurde.

```console
vmadmin@vmLP1:~$ kubectl get svc
NAME       TYPE           CLUSTER-IP       EXTERNAL-IP       PORT(S)        AGE
hello-lb   LoadBalancer   10.103.216.211   192.168.210.200   80:31193/TCP   3s
```

Prüfe mit cURL ob du von deiner VM aus den Service erreichst:
```console
vmadmin@vmLP1:~$ curl http://192.168.210.200
curl: (7) Failed to connect to 192.168.210.200 port 80: Keine Route zum Zielrechner
```

Warum funktioniert es nicht? Tipp: Prüfe die Service Endpoints mit `kubectl get endpoints`. Korrigiere den Service und prüfe erneut mit cURL ob du nun den nginx Webserver erreichen kannst.

## Weiterführende Links
* https://cloud.google.com/kubernetes-engine/docs/concepts/service-discovery
* https://kubernetes.io/docs/concepts/services-networking/connect-applications-service/
* https://kubernetes.io/docs/concepts/services-networking/service/#loadbalancer