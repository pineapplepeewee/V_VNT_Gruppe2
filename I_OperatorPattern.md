## Links
- [Link zu den Ressourcen im GitLab](https://gitlab.com/ch-tbz-hf/Stud/v-cnt/-/tree/main/2_Unterrichtsressourcen/I)
- [Link zur Kubernetes-Oberfläche](https://10.5.38.10:8443/#/create?namespace=default)

# Aufgabe I - Operator Pattern

Grundkonzept des Operator Patterns

Ein Operator in Kubernetes ist eine Methode, um benutzerdefinierte Logik zur Verwaltung von Anwendungen zu implementieren. Ein Beispiel hierfür könnte der "MySampleDB Operator" sein:

1.  **Benutzerdefinierte Ressource "MySampleDB"**: Eine CRD (Custom Resource Definition), um Instanzen Ihrer Datenbank im Cluster zu definieren.
2.  **Operator-Deployment**: Ein Deployment, das einen Pod betreibt, welcher den Operator-Controller enthält.
3.  **Operator-Controller-Container-Image**: Ein spezifisches Container-Image, das den Operator-Code enthält.
4.  **Controller-Logik**: Code, der die Steuerebene abfragt, um den Status der "MySampleDB"-Instanzen zu überwachen und zu verwalten.
5.  **Reaktionsmechanismen**: Der Operator reagiert auf Ereignisse wie das Hinzufügen oder Löschen einer "MySampleDB"-Instanz, indem er Ressourcen wie PersistentVolumeClaims, StatefulSets und Jobs entsprechend verwaltet

Zusätzliche Funktionen und Automatisierung

*   **Backup-Management**: Für jede "MySampleDB"-Instanz automatisiert der Operator das Erstellen von Backups.
*   **Upgrade-Verwaltung**: Der Operator erkennt veraltete Datenbankversionen und initiiert automatisch Upgrades.

**Vorbereitung**

**1\. Docker installation:**

```plaintext
sudo snap install docker
```

 **2. Erstellung des Docker Containers „my-shell“**

```plaintext
docker build -t my-shell .
```

**Erstellung des Operators**

**1\. Dockerfile**

```plaintext
FROM flant/shell-operator:latest
ADD my-pods-hook.sh /hooks
```

**2\. Pods-Hook-Script**

```plaintext
#!/usr/bin/env bash

# Hook-Konfiguration
if [[ $1 == "--config" ]] ; then
    cat <<EOF
    configVersion: v1
    kubernetes:
    - name: onPodAdd
      apiVersion: v1
      kind: Pod
      executeHookOnEvent: ["Added"]
EOF
else
    # Verarbeitungslogik für hinzugefügte Pods
    for binding in $(jq -c '.[]' $BINDING_CONTEXT_PATH); do
        podName=$(jq -r '.object.metadata.name' <<< "$binding")
        echo "Pod '$podName' wurde hinzugefügt"
    done
fi
```

**3\. Operator-Pod-Konfiguration (my-shell-operator-pod.yaml)**

```plaintext
apiVersion: v1
kind: Pod
metadata:
  name: my-shell-operator
  namespace: my-example-monitor-pods
spec:
  containers:
  - name: shell-operator
    image: registry.gitlab.com/ihre-organisation/my-shell-operator
    imagePullPolicy: IfNotPresent
    volumeMounts:
    - name: hooks
      mountPath: /hooks
  volumes:
  - name: hooks
    hostPath:
      path: /pfad/zu/ihren/hooks
  serviceAccountName: monitor-pods-acc
```

**Operator Deployment im Cluster**

**1\. Erstellen eines Namensraums und Service Accounts:**

```plaintext
kubectl create namespace my-example-monitor-pods
```

**2\. Deployen des Operator Pods:**

```plaintext
kubectl -n my-example-monitor-pods apply -f my-shell-operator-pod.yaml
```
