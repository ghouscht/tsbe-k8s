# 03 - Ein Deployment erstellen
[Deployments | Kubernetes](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)

Deployment, das wohl wichtigste Kubernetes Objekt ermöglich deklarative updates von Pods bzw. ReplicaSets. In einem Deployment wird der gewünschte Zustand beschrieben. Der Deployment Controller von Kubernetes verändert den aktuellen Zustand um den gewünschten Zustand zu erreichen in einer geregelten Geschwindigkeit.

## 0. Deployment YAML erstellen
Erstelle eine neue Datei `deployment.yaml` mit folgendem Inhalt:
```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello
  labels:
    app: hello
spec:
  replicas: 3
  selector:
    matchLabels:
      app: hello
  template:
    metadata:
      labels:
        app: hello
    spec:
      containers:
      - name: nginx
        image: nginx:1.20.0-alpine
        ports:
        - containerPort: 80
```
Verwnede nun kubectl um das Deployment zu applizieren.

Tipp: `kubectl apply ...`

Verwende nun `kubectl explain deployment` um dir die einzelnen Felder des Deployments erklären zu lassen.

## 1. Deployment prüfen
Verwende die folgenden Kommandos um den Status des Deployments bzw. der einzelnen Pods zu prüfen:

* `kubectl get events`
* `kubectl get deployment`
* `kubectl describe deployment hello`
* `kubectl get pod -o wide`
* `kubectl get replicasets`
* `kubectl describe replicasets`

Lösche mit `kubectl delete pod ...` einen der 3 hello Pods, was passiert danach (prüfe mit `kubectl get pod`)?

## 2. Verbindung zum *hello* Webserver herstellen
Suche mit `kubectl get pod -l app=hello -o wide` eine IP von einem der drei *hello* Pods. Bspw. *10.127.0.141*.

Falls noch ein Busybox Pod in deinem Namespace läuft öffne eine interaktive Shell in diesem Pod, falls nicht starte einen neuen Busybox Pod mit einer Shell. Führe in der Shell folgendes aus: `wget http://10.127.0.141:80`. **Achtung:** Du musst natürlich die IP auf eine deiner IPs anpassen.