# Aufgabe E - Zugriffssteuerung

## Links
- [Link zu den Ressourcen im GitLab](https://gitlab.com/ch-tbz-hf/Stud/v-cnt/-/tree/main/2_Unterrichtsressourcen/E)
- [Link zur Kubernetes-Oberfläche](https://10.5.38.10:8443/#/create?namespace=default)

## Zugriffssteuerung
MicroK8s ist von Natur aus mehrbenutzerfähig ist, was bedeutet, dass jeder Benutzer, der der microk8s-Gruppe hinzugefügt wird, Befehle gegen den MicroK8s-Cluster ausführen kann.
Allerdings kann es in bestimmten Situationen wünschenswert sein, eine gewisse Benutzerisolierung zu haben, besonders wenn mehrere Benutzer auf einen MicroK8s-Cluster zugreifen. MicroK8s ist eine vollständige Implementierung von Kubernetes, und daher können alle bestehenden Strategien zur Handhabung mehrerer Benutzer angewendet werden.

Als aller erstes, aktivieren wir RBAC mit folgendem Befehl im Terminal: ```yaml microk8s enable rbac ```
![image](https://github.com/Andreeyy/Aufgabe-B---Liveness-Readiness/assets/64062748/2c12f075-accf-4d70-8da5-c6979df86a67)

Nun erstellen wir einen Namensraum (Namespace) für einen Benutzer. Wir nehmen in diesem Fall mich und geben den Namen Andre ein. Dies machen wir mit json:
```json
{
  "apiVersion": "v1",
  "kind": "Namespace",
  "metadata": {
    "name": "andre",
    "labels": {
      "name": "andre"
    }
  }
}
```

Leider erhalten wir folgende Meldung:
![image](https://github.com/Andreeyy/Aufgabe-B---Liveness-Readiness/assets/64062748/a7661b5a-e1b8-4f1c-bcf1-6a5560a14287)

Hätte es jedoch funktioniert, hätten wir den Befehl ```yaml microk8s kubectl apply -f namespace.json ``` anwenden können, um den json Befehl bzw. das json File zu aktivieren.
In Kubernetes und insbesondere im Kontext von Role-Based Access Control (RBAC) werden RoleBindings verwendet, um festzulegen, welche Benutzer oder Gruppen auf welche Ressourcen innerhalb eines Namespaces zugreifen dürfen.
RBAC nutzt Rollen, um zu steuern, welche Aspekte eines Namensraums (Namespace) von einem Benutzer oder einer Gruppe angesehen und/oder modifiziert werden können. Eine Rolle definiert, welche Berechtigungen für eine bestimmte Ressource oder Gruppe von Ressourcen gewährt werden. Ein RoleBinding ordnet dann diese Rolle einem Benutzer oder einer Gruppe zu, um die definierten Berechtigungen zuzuweisen.

Dafür hätten wir den folgenden Befehl benutzt:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: andre
  name: andre-pods
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```

Um die Rolle an den User zu binden, würden wir diesen Befehl nutzen: ```yaml kubectl create rolebinding rolebindingname --role andre-pods --user andre ```

Im nächsten Schritt würden wir den User authentifizieren, dafür gibt es verschiedene Wege. Am besten generieren wir den Private Key und das User-Zertifikat am Terminal:

![image](https://github.com/Andreeyy/Aufgabe-B---Liveness-Readiness/assets/64062748/0e56984d-7430-424b-a6f6-afb3b81b9ae5)

Mit dem Befehl ```yaml cat andre.csr ``` können wir das Zertifikat dann anschauen:

![image](https://github.com/Andreeyy/Aufgabe-B---Liveness-Readiness/assets/64062748/14ef867e-8e9d-472b-ac0a-fb081bce57ca)

Zu letzt können wir dann einen lokalen kubectl config mit folgendem Befehl erstellen: ```yaml microk8s config ```






