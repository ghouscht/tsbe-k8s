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

Öffne nun eine interaktive Shell in einem anderen hello Pod und versuche teste mit cURL ob du eine Verbindung zu einem anderen Pod herstellen kannst: `curl http://10.0.0.27` **Achtung:** Du musst natürlich die IP auf eine deiner IPs anpassen.

Teste ausserdem ob in deinem hello Pod auch das folgende cURL Kommando funktioniert: `curl https://kubernetes.default.svc.cluster.local/version -k`.

Was siehst du hier? Von wem kommt diese Antwort? Verlasse die interaktive Shell im hello Pod nun wieder.

## 3. Deployment updaten
Verwende `kubectl edit deployment ...` um dein hello Deployment anzupassen. Du kannst Beispielsweise mal die `replicas` ändern oder den Tag vom nginx Image. Beobachte mit folgenden Befehlen was passiert:

* `kubectl rollout status deployment hello`
* `kubectl get pod`
* `kubectl get event`
* `kubectl get replicasets.apps`


ps. Dukannst mit `kubectl rollout restart deployment hello` auch mal alle deine Pods neu starten und das ganze beobachten.

## 4. Deployment rollback
Ändere mit `kubectl edit deployment ...` das Image von `nginx:1.20.0-alpine` auf `nginx:i-do-not-exist`. Beobachte wieder was passiert, mit den Kommandos aus Punkt 3. Mache anschliessend mit kubectl die Änderung rückgängig (rollback). 

Tipp: `kubectl rollout ...`

## 5. Erweiterte Konfiguration
Öffne dein `deployment.yaml` und ergänze eine Liveness sowie Readiness Probe am Ende des Deployments:

```yaml
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
```

Prüfe mit `kubectl get pod` den Rollout. Was stellst du fest? Siehst du etwas in den Events (`kubectl get events`)?

## 6. Ressourcen (CPU/Memory) beschränken
Konfigurire mit Hilfe der weiterführenden Links ein CPU Limit von `100m` und ein Memory Limit von `100Mi`. Was passiert, wenn du das Memory Limit auf bspw. `5m` senkst?

## Weiterführende Links
* https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/
* https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/