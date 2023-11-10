# Aufgabe D - Horizontal Pod Autoscaler

## Links
- [Link zu den Ressourcen im GitLab](https://gitlab.com/ch-tbz-hf/Stud/v-cnt/-/tree/main/2_Unterrichtsressourcen/D)
- [Link zur Kubernetes-Oberfläche](https://10.5.38.10:8443/#/create?namespace=default)

## Horizontal Pod Autoscaler

Um einen Horizontal Pod Autoscaler zu erstellen, musste von uns zuerst einige Theorie aufgenommen werden, um das Ganze zu verstehen. Ein HorizontalPodAutoscaler ist ein Mechanismus in Kubernetes, der automatisch eine Arbeitslastsressource aktualisiert. 
Das Hauptziel besteht darin, die Arbeitslast automatisch zu skalieren, um der Nachfrage gerecht zu werden. Horizontales Skalieren bedeutet, dass die Reaktion auf eine erhöhte Last darin besteht, mehr Pods bereitzustellen. 
Dies bedeutet, zusätzliche Pods für die Arbeitslast zu starten. Im Gegensatz dazu bezieht sich vertikales Skalieren darauf, bereits laufenden Pods mehr Ressourcen zuzuweisen. Wir konzentrieren uns in dieser Aufgabe C jedoch auf das horizontale Skalieren.
Der HorizontalPodAutoscaler überwacht die aktuelle Auslastung der Pods. Wenn die Last zunimmt und die Anzahl der Pods unterhalb des konfigurierten Maximums liegt, skaliert der HPA automatisch hoch, indem er mehr Pods für die Arbeitslast startet. 
Wenn die Last abnimmt und die Anzahl der Pods über dem konfigurierten Minimum liegt, skaliert der HPA die Arbeitslast automatisch herunter, um Ressourcen zu sparen.
Wie wir dies verstanden haben, bietet der HorizontalPodAutoscaler zusammengefasst, eine Möglichkeit, die Ressourcennutzung in Kubernetes automatisch an die dynamischen Anforderungen einer Anwendung anzupassen.

Damit wir ein Beispiel des HPA in dieser Aufgabe auzeigen können, brauchen wir zuerst ein Deployment, das einen Container mit dem Image "hpa-example" ausführt und es als Service freigibt. Dafür verwenden wir diesen Manifest in unserem Kubernetes Dashboard:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: php-apache
spec:
  selector:
    matchLabels:
      run: php-apache
  template:
    metadata:
      labels:
        run: php-apache
    spec:
      containers:
      - name: php-apache
        image: registry.k8s.io/hpa-example
        ports:
        - containerPort: 80
        resources:
          limits:
            cpu: 500m
          requests:
            cpu: 200m
---
apiVersion: v1
kind: Service
metadata:
  name: php-apache
  labels:
    run: php-apache
spec:
  ports:
  - port: 80
  selector:
    run: php-apache
```

Wir sehen sofort, wie es das Deployment erstellt hat:
![image](https://github.com/Andreeyy/Aufgabe-B---Liveness-Readiness/assets/64062748/806764c2-7ab3-44d3-95e7-f09b511760f3)

Den Pod:
![image](https://github.com/Andreeyy/Aufgabe-B---Liveness-Readiness/assets/64062748/51be115a-817b-4d57-9c03-1ded435d3c81)

Und auch den Service:
![image](https://github.com/Andreeyy/Aufgabe-B---Liveness-Readiness/assets/64062748/1cc5c1f5-63aa-4437-8c4a-2e790e2f470c)

### Bereitstellung der Anwendung
Um die Anwendung nun in Kubernetes bereitzustellen, laden wir die Ressourcendefinitionen aus der YAML-Datei aus. Dies machen wir mit folgendem Befehl im Terminal:
```yaml kubectl apply -f https://k8s.io/examples/application/php-apache.yaml ```

Das ganze sieht dann so aus:
![image](https://github.com/Andreeyy/Aufgabe-B---Liveness-Readiness/assets/64062748/b638b4ac-53ce-423a-8c8b-bd9a9891770e)

Zusammenfassend wurden nun die in der YAML-Datei unter der angegebenen URL definierten Ressourcen im Kubernetes-Cluster erstellt. Die Warnungen weisen nur darauf hin, dass bestimmte Annotations fehlen, aber das System wird versuchen, sie automatisch zu ergänzen.
Somit können wir die Warnungen vorerst Mal ignorieren.

### Erstellung des HPA
Wir haben oben nun den Server zum Laufen gebracht, somit können wir nun den HPA erstellen. Wir wollen den HPA so konfigurieren, um die Anzahl der Pods zwischen 1 und 10 zu skalieren, je nach dem wie die CPU-Auslastung ist. Dies machen wir mit folgendem Befehl:
```yaml kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10 ```

Um den Status anzeigen zu lassen, können wir im Terminal den Befehl ```yaml kubectl get hpa ``` eingeben:
![image](https://github.com/Andreeyy/Aufgabe-B---Liveness-Readiness/assets/64062748/2a776fe8-4c92-4ddc-9d87-5ce6ab1b6cb7)

Das "unknown" im Kontext der CPU-Auslastung für die Deployments könnte darauf hinweisen, dass der HorizontalPodAutoscaler (HPA) Schwierigkeiten hat, die tatsächliche CPU-Auslastung der Pods zu bestimmen. Dies kann mehrere Gründe haben.
In den Events sehen wir die Meldung "FailedGetRersourceMetric". Die Meldung "FailedGetResourceMetric" weist darauf hin, dass der HorizontalPodAutoscaler (HPA) Schwierigkeiten hat, Metriken für die Ressource "cpu" zu erhalten.
![image](https://github.com/Andreeyy/Aufgabe-B---Liveness-Readiness/assets/64062748/6a9d220f-cf28-46c1-b7c3-c7ec208920b9)

Daher habe ich versucht, diesen zu installieren, jedoch erfolglos, da er vermutlich bereits drauf ist:
![image](https://github.com/Andreeyy/Aufgabe-B---Liveness-Readiness/assets/64062748/a26e0403-58bc-4208-a62e-7725e1ca7cc7)

Nichtsdestotrotz, nehmen wir an, dass anstatt unknown, dort 0% steht. Es ist nun möglich, die Last eines anderen Pods mit folgendem Befehl zu erhöhen:
```yaml kubectl run -i --tty load-generator --rm --image=busybox:1.28 --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://php-apache; done" ```

Wenn dies gemacht wird, kann man mit dem Befehl: ```yaml kubectl get hpa php-apache --watch ``` sehen, dass nun anstatt 0% dort etwa 305% stehen würde. 









