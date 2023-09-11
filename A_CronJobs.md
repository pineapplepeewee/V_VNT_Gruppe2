# CronJobs
## Link Unterrichtsressourcen

https://gitlab.com/ch-tbz-hf/Stud/v-cnt/-/tree/main/2_Unterrichtsressourcen/A

## CronJob erstellen

https://10.5.38.10:8443/#/create?namespace=default


CronJob erstellen (zB dieser wartet jeweils 30 sekunden)
 







Code:
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
 

Unser Script führt einen Perl-Befehl aus, um die Zahl Pi berechnet und den Wert dann ausgibt.
Nach dem Job wartet es 20 sekunden
Dann wird der Job gestartet. (Wir haben insgesamt 6 Pods, bei dem 2 immer Parallel laufen
Daher sieht man links: 1 Job Running (Unser CroJob „pi“)
Rechts: Running 2 (Momentan)

 

Beim 3. Durchlauf der Pods sieht es dann so aus:

 
Am Schluss wurde der Job erfolgreich ausgeführt und dementsprechend alle auf Succeeded
 
