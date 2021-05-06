# 01 - Namespace erstellen
[Namespaces | Kuberntes](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)

Namespaces werden verwendet um verschiedene Applikationen/Teams zu trennen. Es gibt auch diverse Namespaces die in einer default Installation von Kubernetes vorhanden sind.

## 0. Cluster Konnektivität prüfen
Verwende `kubectl` um einige Infos des Clusters zu erhalten:

```bash
kubectl cluster-info
kubectl get nodes
kubectl get pods --all-namespaces
```

## 1. Vorhandene Namespaces anzeigen
Verwende kubectl um die vorhanden Namespaces anzuzeigen, verwende die shell completion dazu (\<TAB\>). 

Tipp: `kubectl get ...`

Du solltest diverse Namespaces sehen mit Namen wie *kube-system* oder *default*.

## 2. Erstelle einen persönlichen Namespace
Verwende kubectl um einen persönlichen Namespace zu erstellen, nimm deinen Nachnamen (in Kleinbuchstaben) als Name für den Namespace. 

Tipp: `kubectl create ...`

Prüfe anschliessend ob der Namespace erstellt wurde, indem du noch einmal alle Namespaces anzeigen lässt.

## 3. Default Namespace festlegen
Verwende wieder kubectl um den default Namespace zu ändern auf deinen neu erstellten Namespace. 

Tipp: `kubectl config set-context --current ...`

Öffne deine KUBECONFIG (`$HOME/kube/config`) und prüfe ob du den Namespace dort findest. Prüfe abschliessend mit `kubectl config get-contexts` ob der Namespace wirklich richtig gesetzt ist.

## Weiterführende Links
* https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/
* https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/
* https://kubernetes.io/docs/tasks/extend-kubectl/kubectl-plugins/