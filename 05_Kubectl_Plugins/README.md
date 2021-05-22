# 05 - Kubectl Plugins
[Extend kubectl with plugins | Kubernetes](https://kubernetes.io/docs/tasks/extend-kubectl/kubectl-plugins/)

Kubectl kann mit Hilfe von Plugins um zusätzliche Funktionalität erweitert werden. Plugins können mit Hilfe von [krew](https://github.com/kubernetes-sigs/krew/) gefunden und installiert werden.

## 00. krew installieren
1. Installiere krew gemäss [Anleitung](https://krew.sigs.k8s.io/docs/user-guide/setup/install/).
2. Update den krew Index mit `kubectl krew update`.

## 01. ns Plugin installieren
Installiere das Plugin `ns` zum einfachen wechseln von Namespaces.

Tipp: `kubectl krew install ...`

## 02. ns Plugin verwenden
Das `ns` Plugin ist sehr praktisch um einfach zwischen Kubernetes Namespaces hin und her zu wechseln. Verwendest du das Plugin ohne Optionen, erhälst du eine Liste aller Namespaces und dir wird farblich angezeigt in welchem Namespace du dich gerade bewegst.

```console
vmadmin@vmLP1:~$ kubectl ns
default
gosteli
kube-node-lease
kube-public
kube-system
loosli
metallb-system
```

Namespace wechseln kannst du einfach in dem du den entsprechenden Namen als Argument mitgibst:
```console
vmadmin@vmLP1:~$ kubectl ns kube-system
Context "kubernetes-admin@kubernetes" modified.
Active namespace is "kube-system".
```

## 03. Weitere Plugins suchen/testen
Installiere einige weitere Plugins und teste deren Funktinalität. Du findest eine Liste aller Plugins mit `kubectl krew search`.

Vorschläge:
* example
* doctor
* access-matrix

## Weiterführende Links
* https://krew.sigs.k8s.io/docs/
* https://krew.sigs.k8s.io/plugins/ 
* https://ahmet.im/blog/kubectl-plugins/