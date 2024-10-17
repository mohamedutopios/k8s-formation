**Mise en place de l'Horizontal Pod Autoscaler (HPA) pour votre déploiement**

Pour démontrer l'augmentation du nombre de pods en fonction de la consommation CPU, vous devez configurer un HPA pour votre déploiement `result-ui`. Voici les étapes détaillées pour y parvenir.

---

### **1. Vérifier l'installation de Metrics Server**

L'HPA nécessite Metrics Server pour récupérer les métriques d'utilisation des ressources.

- **Vérifiez si Metrics Server est installé :**

  ```bash
  kubectl get deployment metrics-server -n kube-system
  ```

- **Si Metrics Server n'est pas installé, installez-le :**

  ```bash
  kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
  ```

---

### **2. Mettre à jour le déploiement avec les ressources CPU**

L'HPA utilise les valeurs de `requests` et `limits` pour calculer l'utilisation du CPU en pourcentage.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: result-ui
  name: result-ui
  namespace: vote
spec:
  replicas: 1
  selector:
    matchLabels:
      app: result-ui
  template:
    metadata:
      labels:
        app: result-ui
    spec:
      containers:
        - image: mohamed1780/result-ui
          name: result-ui
          imagePullPolicy: Always
          ports:
            - containerPort: 80
              name: result-ui
          resources:
            requests:
              cpu: 100m
            limits:
              cpu: 200m
```

**Explications :**

- **resources.requests.cpu** : La quantité de CPU garantie pour le conteneur.
- **resources.limits.cpu** : La quantité maximale de CPU que le conteneur peut utiliser.

---

### **3. Créer le HPA pour le déploiement**

Voici le fichier YAML pour le HPA :

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: result-ui
  namespace: vote
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: result-ui
  minReplicas: 1
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 10
```

**Explications :**

- **minReplicas** : Nombre minimum de pods.
- **maxReplicas** : Nombre maximum de pods.
- **averageUtilization** : Pourcentage cible d'utilisation du CPU (ici, 50%).

---

### **4. Appliquer les configurations au cluster**



- **Créez le HPA :**

  ```bash
  kubectl apply -f hpa.yaml
  ```

   kubectl top nodes

---

### **5. Générer une charge CPU pour la démonstration**


kubectl run ab -ti --rm --restart='Never' --image=lucj/ab -- -n 200000 -c 100 http://result-ui.vote.svc.cluster.local:5000/


---

### **6. Observer l'HPA en action**

- **Surveiller l'HPA :**

  ```bash
  kubectl get hpa result-ui -n vote --watch
  ```

- **Surveiller l'utilisation des pods :**

  ```bash
  
  kubectl top pods -n vote

  kubectl get pods -n vote -l app=result-ui

  ```

**Ce que vous devriez observer :**

- L'utilisation du CPU augmente au-delà du seuil de 10%.
- L'HPA augmente le nombre de réplicas pour répartir la charge.

---

### **7. Nettoyage après la démonstration**

- **Retirez le conteneur de stress du déploiement pour revenir à la normale.**
- **Réappliquez le déploiement sans le conteneur cpu-stress :**

  ```bash
  kubectl apply -f deployment.yaml
  ```

---

### **8. Points supplémentaires**

- **Assurez-vous que le Metrics Server fonctionne correctement.**
- **Vérifiez que les valeurs de `requests` et `limits` sont cohérentes.**
- **Après la suppression du stress, observez l'HPA réduire le nombre de pods.**

---

### **Résumé de la procédure**

1. **Configurer le Metrics Server** pour permettre à l'HPA de récupérer les métriques.
2. **Ajouter les ressources CPU** dans le déploiement pour chaque conteneur.
3. **Créer et appliquer le HPA** avec les seuils d'utilisation CPU souhaités.
4. **Générer une charge CPU** pour déclencher l'autoscaling.
5. **Observer le comportement de l'HPA** et le nombre de pods qui augmente.
6. **Nettoyer les modifications** après la démonstration.

---

**En maîtrisant ces étapes, vous serez en mesure de démontrer efficacement comment l'HPA ajuste le nombre de pods en fonction de la consommation CPU dans Kubernetes.**