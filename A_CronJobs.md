# CronJobs
## Link Unterrichtsressourcen

https://gitlab.com/ch-tbz-hf/Stud/v-cnt/-/tree/main/2_Unterrichtsressourcen/A

## CronJob erstellen

https://10.5.38.10:8443/#/create?namespace=default


**Jobs**

- [Link zu den Ressourcen im GitLab](https://gitlab.com/ch-tbz-hf/Stud/v-cnt/-/tree/main/2_Unterrichtsressourcen/A)
- [Link zur Kubernetes-Oberfläche](https://10.5.38.10:8443/#/create?namespace=default)

**CronJob erstellen (zum Beispiel, dieser wartet jeweils 30 Sekunden)**

**YAML-Code:**

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pi
spec:
  parallelism: 2
  completions: 6
  template:
    spec:
      containers:
      - name: pi
        image: perl:5.34.0
        command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: OnFailure
  backoffLimit: 4
```

Unser Skript führt einen Perl-Befehl aus, um die Zahl Pi zu berechnen und den Wert dann auszugeben. Nach dem Job wartet er 30 Sekunden, bevor der Job gestartet wird. Insgesamt werden 6 Pods erstellt, wobei jeweils 2 parallel laufen.

Am Anfang sieht man links "1 Job Running" (Unser CronJob "pi") und rechts "Running 2" (Momentan).

Beim dritten Durchlauf der Pods sieht es dann so aus:

[Hier kann ein Screenshot eingefügt werden]

Am Ende wurde der Job erfolgreich ausgeführt, und alle Pods sind auf "Succeeded" gesetzt.