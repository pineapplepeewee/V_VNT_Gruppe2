# Aufgabe B - Liveness, Readiness

## Links
- [Link zu den Ressourcen im GitLab](https://gitlab.com/ch-tbz-hf/Stud/v-cnt/-/tree/main/2_Unterrichtsressourcen/A)
- [Link zur Kubernetes-Oberfläche](https://10.5.38.10:8443/#/create?namespace=default)

## Liveness Befehl
Es kann schnell einmal passieren, dass eine Applikation, welche über längere Zeit lauft, zu einem beschädigten Zustanden wechselt und sich nicht mehr erholen kann, ohne neugestartet zu werden.
Mit Kubernetes können wir uns diese Fälle etwas vereinfachen. Kubernetes bietet uns die Möglichkeit, Lebenszeichen zu senden, damit wir von solchen Situationen schnell erfahren und reagieren können.

Mit dem folgenden Befehl können wir einen Pod erstellen, der einen Container ausführt:

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-exec
spec:
  containers:
  - name: liveness
    image: registry.k8s.io/busybox
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -f /tmp/healthy; sleep 600
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5
```

Wie oben ersichtlich, haben wir folgenden command: ```yaml /bin/sh -c "touch /tmp/healthy; sleep 30; rm -f /tmp/healthy; sleep 600" ```. Dies erstellt zuerst ein leeres File ```yaml /tmp/healthy ```.
Danach definiert es, dass es 30 Sekunden warten soll, bis es mit dem nächsten Befehl weitermachen soll. Nach den 30 Sekunden löscht es das File im Verzeichnis /tmp. Das ```yaml sleep 600```können wir ignorieren. Dies bedeutet lediglich, dass es dann 10 Minuten wartet, bevor der Befehl beendet wird.
Somit haben wir kurzgefasst, ein File, welches 30 Sekunden besteht.

Da wir den Befehl ```yaml cat /tmp/healthy``` haben, erhalten wir jede 5 Sekunden (definiert im Feld "periodSecons"), einen Erfolg retourniert. Doch nach 35 Sekunden, wenn das File gelöscht wird, erhalten wir mit dem gleichen Befehl einen gescheiterten Versuch, da das File nicht mehr exisiert.

Das Ganze sieht im GUI nach ungefähr 40 Minuten wie folgt aus:

![image](https://github.com/Andreeyy/Aufgabe-B---Liveness-Readiness/assets/64062748/cc9e0c2c-4f6d-4b57-8a55-74fac1737c00)

Wir sehen hier, dass es bereits 15 Mal neugestartet hat. Ausserdem können wir mit dem Befehl ```yaml kubectl describe pod liveness-exec ``` jeweils das Geschehen sozusagen "Live" anschauen.
