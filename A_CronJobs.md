# Aufgabe A - CronJobs

## Links
- [Link zu den Ressourcen im GitLab](https://gitlab.com/ch-tbz-hf/Stud/v-cnt/-/tree/main/2_Unterrichtsressourcen/A)
- [Link zur Kubernetes-Oberfläche](https://10.5.38.10:8443/#/create?namespace=default)

## CronJob erstellen

CronJobs sind etwa wie Scheduled Tasks. Der folgende CronJob berechnet die Zahl PI und gibt sie aus, danach wartet er 30 Sekunden.


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
        actions: "Sleep well, darling"
        type: "SLEEP"
        trigger_time: "ON_EVERY_EXECUTION"
        sleep_in_sec: 30
      restartPolicy: OnFailure
  backoffLimit: 4
```

Unser Skript führt einen Perl-Befehl aus, um die Zahl Pi zu berechnen und den Wert dann auszugeben. Nach dem Job wartet er 30 Sekunden, bevor der Job gestartet wird. Insgesamt werden 6 Pods erstellt, wobei jeweils 2 parallel laufen.

Am Anfang sieht man links "1 Job Running" (Unser CronJob "pi") und rechts "Running 2" (Momentan).

![Durchlauf 1](/Bilder/A_CronJobs_JobPods1.png)

Beim dritten Durchlauf der Pods sieht es dann so aus:

![Durchlauf 1](/Bilder/A_CronJobs_JobPodsDurchlauf2.png)

Am Ende wurde der Job erfolgreich ausgeführt, und alle Pods sind auf "Succeeded" gesetzt.

![Durchlauf 1](/Bilder/A_CronJobs_JobPodsDurchlauf3_Succeeded.png)