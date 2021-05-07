# 02 - Einen Pod starten
[Pods | Kubernetes](https://kubernetes.io/docs/concepts/workloads/pods/)

Ein Pod ist eine Gruppe von einem oder mehreren Container. Mehrere Container in einem Pod teilen sich Netzwerk und Storage. Container in einem Pod sind immer co-located und co-scheduled und laufen in einem sog. "shared context".

## 0. Namespace prüfen
Prüfe mit kubectl, das du dich in deinem persönlichen Namespace befindest. Entsprechende Kommandos hast du bereits in Aufgabe 1 verwendet.

## 1. Pod starten
Verwende das Dockerimage `busybox` um einen Pod zu starten mit dem Namen `mypod`. 

Tipp: `kubectl run ...`

## 2. Pod inspizieren
Verwende die folgenden kubectl Kommandos um den Pod zu inspizieren:

* `kubectl get pod`
* `kubectl describe pod mypod`
* `kubectl logs mypod`

Wie du sicher gemerkt hast stimmt etwas nicht mit dem Pod, er sollte im Zustand `CrashLoopBackOff` (eventuell auch kurz im `Completed` Zustand) sein. Wir erfahren später was das Problem ist. Lösche den Pod mit `kubectl delete pod ...`.

## 3. Pod erneut starten
Verwende erneut `kubectl run` um einen Busybox Pod zu starten. Verwende aber zusätzlich die Optionen `--tty`, `--stdin` und `--restart=Never`. Wenn du alles richtig machst erhälst du eine Shell im Busybox Container. 

Führe in dieser Shell folgende Kommandos aus und mache dir Gedanken zu den Ausgaben der Befehle. Was fällt dir auf? Falls du Fragen hast, wäre das der richtige Zeitpunkt deinen Dozenten zu fragen.

* `hostname`
* `df -h`
* `ps -ef`
* `ip addr show`

Beende anschliessend die Shell im Busybox Pod mit `exit`.

## 4. Pod erneut inspizieren
Dein Pod sollte nun im Status `Completed` sein. Verwende erneut die kubectl Kommandos aus Punkt 2 um den Pod zu inspizieren. Siehst du einen Unterschied? Lösche den Pod anschliessend erneut.

## 5. Pod im Hintergrund starten
Nun möchten wir gerne noch den Busybox Pod im Hintergrund starten, also ohne interaktive Shell. Wir haben bereits gelernt, dass sich der Pod aber immer wieder beendet wenn wir das möchten. Warum ist das so? 

Starte mit `kubectl run` erneut einen Busybox Pod, dieses mal soll der Pod aber das Kommando `sleep infinity` ausführen. 

Tipp: `kubectl run --help`

Wenn du alles richtig machst, sollte es am Ende so aussehen:
```console
$ kubectl get po
NAME    READY   STATUS    RESTARTS   AGE
mypod   1/1     Running   0          3m53s
```

Verwende nun erneut die Kommandos um den Pod zu inspizieren. Findest du unterschiede?

## 6. Shell an laufenden Pod "attachen"
Starte nun noch eine interaktive Shell im laufenden Pod. Verwende `kubectl exec ...` um `/bin/sh` auszuführen im Pod. 

Tipp: `kubectl exec` benötigt zwei Optionen, die du bereits einmal verwendet hast. 

Wenn du die Shell starten konntest, führe erneut das Kommando `ps -ef` im Pod aus, findest du einen Unterschied zu der Ausgabe aus Punkt 3? 
Beende nun die interaktive Shell im Pod, prüfe anschliessend ob der Pod noch läuft.

## 7. Wo läuft mein Pod? Welche IP hat er?
Verwende `kubectl get po -o wide` um dir auf übersichtliche Art und Weise mehr Infos zu deinem Pod anzeigen zu lassen. Lass dir das Pod Objekt zusätzlich auch als YAML anzeigen mit: `kubectl get pod mypod -o yaml` - ziemlich viele Infos, nicht?

## 8. Kubectl Hilfe zu k8s Objekten
Verwende `kubectl explain ...` um Hilfe zu Kubernetes Objekten zu erhalten.
Kubernetes Objekte sind immer identisch aufgebaut mit folgenden Feldern:

* apiVersion
* kind
* metadata
* spec
* status

Lies die Infos zum Pod Objekt (`kubectl explain pod`) aufmerksam durch. Falls du beispielsweise weitere Infos zum `spec` Objekt eines Pods suchst kannst du `kubectl explain pod.spec` verwenden.