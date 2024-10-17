Pour mettre en place une **Network Policy avec Calico** dans Kubernetes, vous devez d'abord vous assurer que **Calico** est installé en tant que plugin CNI (Container Network Interface) dans votre cluster Kubernetes. Calico permet de définir des **Network Policies** pour contrôler le trafic réseau entre les pods.

### **Étape 1 : Installer Calico**
Si Calico n'est pas encore installé, suivez les étapes ci-dessous pour l'installer.

#### **1.1. Installation de Calico**
Vous pouvez installer Calico en appliquant un manifeste Kubernetes depuis la documentation officielle de Calico. Exécutez la commande suivante pour installer Calico :

```bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

Cela installera Calico et les composants nécessaires (comme les CRDs) pour prendre en charge les **Network Policies**.

#### **1.2. Vérification de l'installation**
Vérifiez que Calico est installé et en cours d'exécution :

```bash
kubectl get pods -n kube-system -l k8s-app=calico-node
```

Les pods `calico-node` doivent être en statut **Running**.

Vous pouvez également vérifier que les **Custom Resource Definitions (CRDs)** nécessaires sont présentes :

```bash
kubectl get crds | grep 'calico'
```

Vous devriez voir des CRDs comme `globalnetworkpolicies.crd.projectcalico.org`, `networkpolicies.crd.projectcalico.org`, etc.

---

### **Étape 2 : Créer une Network Policy avec Calico**

Une fois Calico installé, vous pouvez créer des **Network Policies** pour restreindre ou autoriser le trafic entre les pods.

Voici un exemple de Network Policy qui **bloque tout le trafic entrant** vers un pod sauf s'il est explicitement autorisé.

#### **Exemple : Bloquer tout le trafic entrant par défaut**
Ce fichier YAML définit une Network Policy qui bloque tout le trafic entrant à moins que vous ne définissiez des règles d'autorisation.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
  namespace: vote  # Remplacez par votre namespace
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress: []
```

### **Explications** :
- **`podSelector: {}`** : Cela sélectionne **tous les pods** dans le namespace spécifié (`vote`).
- **`policyTypes: Ingress`** : Cette politique s'applique au trafic entrant (Ingress).
- **`ingress: []`** : Aucune règle d'autorisation pour le trafic entrant, donc tout le trafic est bloqué.

---

### **Étape 3 : Autoriser le Trafic Sélectivement avec Calico**

Ensuite, vous pouvez créer des politiques qui **autorisent** certaines communications entre des pods spécifiques.

#### **Exemple : Autoriser le trafic de `vote-ui` vers `vote`**
Voici une Network Policy qui autorise uniquement le trafic du pod `vote-ui` vers le pod `vote` sur le port 5000.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-vote-ui-to-vote
  namespace: vote  # Remplacez par votre namespace
spec:
  podSelector:
    matchLabels:
      app: vote  # Cible les pods avec le label `app=vote`
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: vote-ui  # Autorise uniquement le trafic venant des pods `app=vote-ui`
    ports:
    - protocol: TCP
      port: 5000  # Autorise le trafic sur le port 5000
```

### **Explications** :
- **`podSelector: app: vote`** : La politique s'applique aux pods avec le label `app=vote`.
- **`ingress`** : Autorise uniquement le trafic entrant venant des pods `vote-ui`.
- **`ports: 5000`** : Le trafic est autorisé seulement sur le port 5000 (celui utilisé par `vote`).

---

### **Étape 4 : Appliquer les Network Policies**

Une fois que vous avez défini vos Network Policies dans un fichier YAML, appliquez-les à votre cluster avec la commande suivante :

```bash
kubectl apply -f <nom_du_fichier>.yaml
```

Par exemple :

```bash
kubectl apply -f default-deny-ingress.yaml
kubectl apply -f allow-vote-ui-to-vote.yaml
```

---

### **Étape 5 : Vérifier les Network Policies**

Vous pouvez vérifier les Network Policies appliquées dans votre namespace avec la commande suivante :

```bash
kubectl get networkpolicy -n vote
```

Pour obtenir plus de détails sur une policy spécifique, utilisez :

```bash
kubectl describe networkpolicy <nom_de_la_policy> -n vote
```

---

### **Étape 6 : Tester les Network Policies**

1. **Lancer un pod de test** pour vérifier les communications entre les services :

   ```bash
   kubectl run test-pod --rm -it --image=alpine -- sh
   ```

2. **Tester la communication avec `curl`** pour vérifier si le trafic est autorisé ou bloqué entre les pods.

   - Tester la communication avec le service `vote-ui` :

     ```bash
     curl http://vote-ui.vote.svc.cluster.local:5000
     ```

   - Tester la communication avec le service `vote` :

     ```bash
     curl http://vote.vote.svc.cluster.local:5000
     ```

   Si la Network Policy est correctement appliquée, le trafic devrait être autorisé ou bloqué selon les règles définies.

---


