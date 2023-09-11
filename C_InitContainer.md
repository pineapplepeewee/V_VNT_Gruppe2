# V_VNT_Gruppe2

## Aufgabe C: Init Container

Zuerst wird der Hauptcontainer `myapp-pod` erstellt mit 2 Init Containers. Die 2 Init Containers warten auf die 2 Services `myservice` und `mydb`, die später erstellt werden. Erst wenn er sich mit den 2 Services verbinden konnte, wird der Hauptcontainer erstellt bzw. ausgeführt. Ausschlaggebend hier ist der Code-Stück `until nslookup myservice` bzw. `until nslookup mydb`. Somit wird ein Echo bzw. Ausgabe gemacht mit dem Text "waiting for myservice" oder "waiting for mydb", solange er die beiden Services noch nicht erreichen kann.

Den YAML-Code kann man hier eingeben, sodass der (Haupt)-Container mit den 2 Init-Containern erstellt wird: [Link zur Kubernetes-Oberfläche](https://10.5.38.10:8443/#/create?namespace=default)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox:1.28
    command: ['sh', '-c', 'until nslookup myservice; do echo waiting for myservice; sleep 2; done;']
  - name: init-mydb
    image: busybox:1.28
    command: ['sh', '-c', 'until nslookup mydb; do echo waiting for mydb; sleep 2; done;']
```

Im Workload sieht man nun, dass der Pod "ausstehend" ist.

Mit den folgenden beiden Befehlen kann man Informationen über unseren soeben erstellten Pod sehen:

```bash
kubectl get pods/myapp-pod
```

Hier sieht man zum Beispiel, dass keiner der Init-Containers ready ist (weil sie sich nicht mit den nötigen Services verbinden können).

```bash
kubectl describe pods/myapp-pod
```

Nun wird der erste Service `myservice` erstellt, sodass der erste Init-Container eine Verbindung aufbauen kann.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myservice
spec:
  ports:
  - protocol: TCP
    port: 80
    targetPort: 9376
```

Nun wurde der Service "myservice" erstellt und beim "myapp-pod" steht jetzt "Init 1/2", da er sich mit dem ersten Service verbinden kann.

Nun wird der letzte Service erstellt, sodass der Container "myapp-pod" dann endlich erstellt werden kann.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mydb
spec:
  ports:
  - protocol: TCP
    port: 80
    targetPort: 9377
```

Nun sehen wir auch, dass unser "myapp-pod" läuft.
