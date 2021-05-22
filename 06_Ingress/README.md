# 06 - Ingress
[Ingress | Kubernetes](https://kubernetes.io/docs/concepts/services-networking/ingress/)

Ingress ist ein API Objekt, welches externen Zugriff (von ausserhalb des k8s Clusters) auf einen Service im Cluster ermöglicht. Typscherweise erfolgt dieser Zugriff ausschliesslich via HTTP/HTTPS. Manche Ingres Controller stellen Loadbalancing Funktionen, SSL Terminierung und Namensbasierte virtuelle Hosts zur Verfügung.

## 00. Ingress Controller auswählen/deployen
[Ingress Controllers | Kubernetes ](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/)

Damit die `Ingress` Ressource funktioniert, muss im Kubernetes Cluster ein ingress controller installiert wer5den. Anders als andere Controller, welche als Teil vom kube-controller-manager binary laufen werden Ingress controller nicht autmatisch mit einem Cluster gestartet.

Kubernetees als Projhekt unterstürtzt/unterhält den AWS, GCE und nginx ingress controller. Wir werden aus diesem Grund den [ingress-nginx](https://github.com/kubernetes/ingress-nginx) verwenden.

Dank metallb funktionieren in unserem Cluster Services mit Type `LoadBalancer`. Aus diesem Grund können wir die Installation welche eigentlich für AWS gedacht ist verwenden gemäss der [Anleitung](https://kubernetes.github.io/ingress-nginx/deploy/#aws). Notiere dir die *EXTERNAL-IP* vom Service *ingress-nginx-controller* im Namespace *ingress-nginx*: `kubectl -n ingress-nginx get svc ingress-nginx-controller`

## 01. DNS wildcard konfigurieren
1. Auf deinem System im `/etc/NetworkManager/NetworkManager.conf` folgende Zeile (in der Sektion `main`) als root ergänzen/auskommentieren als:
```
dns=dnsmasq
```
2. resolv.conf von NetworkManager verwalten lassen: `sudo rm /etc/resolv.conf ; sudo ln -s /var/run/NetworkManager/resolv.conf /etc/resolv.conf`
3. dnsmasq Konfiguration erstellen: **WICHTIG: Ersetze die IP Adresse mit der notierten external IP vom ingress!**
`echo 'address=/app.smartlearn.lan/192.168.210.201' | sudo tee /etc/NetworkManager/dnsmasq.d/smartlearn.lan.conf`
4. NetworkManager neu laden: `sudo systemctl reload NetworkManager`

Du solltest nun bspw. folgende DNS Namen auflösen können:

* bla.app.smartlearn.lan
* test.app.smartlearn.lan

## 02. Hello Ingress erstellen
Erstelle eine neue Datei `ingress.yaml`:

**Wichtig: Ersetze *gosteli* durch deinen Nachnamen!**
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: hello
spec:
  rules:
  - http:
    host: "hello-gosteli.app.smartlearn.lan"
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: hello
            port:
              number: 80
```

Erstelle das Ingress Objekt in deinem Namespace mit `kubectl apply -f ingress.yaml`. Prüfe anschliessend ob du via Browser mit http://hello-gosteli.app.smartlearn.lan auf deinen Hello Service zugreifen kannst.

## Weiterführende Links
* https://askubuntu.com/questions/1029882/how-can-i-set-up-local-wildcard-127-0-0-1-domain-resolution-on-18-04-20-04 
* https://kubernetes.github.io/ingress-nginx/ 
* https://medium.com/flant-com/comparing-ingress-controllers-for-kubernetes-9b397483b46b