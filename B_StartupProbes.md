# Aufgabe B - Liveness, Readiness, Startup Probes

## Links
- [Link zu den Ressourcen im GitLab](https://gitlab.com/ch-tbz-hf/Stud/v-cnt/-/tree/main/2_Unterrichtsressourcen/B)
- [Link zur Kubernetes-Oberfläche](https://10.5.38.10:8443/#/create?namespace=default)

##  Startup Probes
Bei den Startup Probes geht es darum, wie man mit älteren Anwendungen umgeht, die beim Starten mehr Zeit benötigen. Wir wollen sicherstellen, dass sie sowohl während ihres Startvorgangs als auch während ihres normalen Betriebs ordnungsgemäss überwacht werden.
Dies wird erreicht, indem eine speizelle "Startup-Probe" eingerichtet wird, die anfänglich häufiger überprüft wird, um den längeren Startvorgang zu berücksichtigen und dann in den normalen Überwachungszyklus übergeht, sobald die Anwendung ordnungsgemäss gestartet ist.
Dies ist besonders wichtig, um sicherzustellen, dass die Anwendung zuverlässig und effizient in Kubernetes betrieben werden kann.

### Startup Probes Befehl
```yaml
ports:
- name: liveness-port
  containerPort: 8080
  hostPort: 8080

livenessProbe:
  httpGet:
    path: /healthz
    port: liveness-port
  failureThreshold: 1
  periodSeconds: 10

startupProbe:
  httpGet:
    path: /healthz
    port: liveness-port
  failureThreshold: 30
  periodSeconds: 10
```
Hier ist zu sehen, wie die Startup-Probe der Anwendung genügend Zeit gibt, um sich beim Start zu initialisieren. Wenn dies innerhalb von 5 Minuten nicht gelingt, wird der Container als nicht erfolgreich betrachtet und kann gemäss den Neustartrichtlinien des Pods neu gestartet oder beendet werden.
Sobald die Startup-Probe erfolgreich war, wird die Liveness-Probe verwendet, um sicherzustellen, dass der Container während seines normalen Betriebs ordnungsgemäss funktioniert.
Wenn die Liveness-Probe fehlschlägt, wird der Container als nicht gesund betrachtet und kann neu gestartet werden.
