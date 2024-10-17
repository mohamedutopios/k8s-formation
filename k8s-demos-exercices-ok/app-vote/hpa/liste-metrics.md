

- https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/


En Kubernetes, l'**Horizontal Pod Autoscaler (HPA)** peut être configuré pour ajuster le nombre de pods en fonction de différentes métriques. Ces métriques peuvent provenir de plusieurs sources : **métriques de ressources** (comme le CPU et la mémoire), **métriques personnalisées** provenant d'applications spécifiques, et **métriques externes** fournies par des services externes à Kubernetes.

Voici une liste complète des types d'indicateurs que vous pouvez utiliser avec HPA :

### **1. Métriques de Ressources**
Ces métriques sont fournies par le **Metrics Server** et se basent sur l'utilisation des ressources des conteneurs :

#### a) **CPU**
- **`cpu`** : Autoscale en fonction de l'utilisation du CPU, souvent mesuré en milli-cores (m). Par exemple, si un conteneur demande 100m (0,1 core), Kubernetes ajuste en fonction de l'utilisation de ces milli-cores.

#### b) **Mémoire**
- **`memory`** : Autoscale en fonction de la consommation de mémoire (RAM), mesurée en MiB ou GiB. C'est utile pour les applications sensibles à la mémoire.

**Exemple de configuration pour le CPU** :
```yaml
metrics:
- type: Resource
  resource:
    name: cpu
    target:
      type: Utilization
      averageUtilization: 50
```

---

### **2. Métriques Personnalisées**
Les **métriques personnalisées** permettent d’utiliser des métriques spécifiques à vos applications, collectées par des outils comme **Prometheus**. Vous pouvez utiliser les métriques d’application ou d’infrastructure que vous collectez via un adaptateur comme **Prometheus Adapter** pour Kubernetes.

Exemples de métriques personnalisées que vous pourriez utiliser :
- **Transactions par seconde** : Le nombre de transactions ou requêtes traitées par pod.
- **Nombre d'éléments dans une file d'attente** : Pour des systèmes de traitement par file d'attente, le nombre d'éléments en attente peut être une bonne mesure de la charge.

**Exemple de configuration pour une métrique personnalisée (Prometheus) :**
```yaml
metrics:
- type: Pods
  pods:
    metric:
      name: requests-per-second
    target:
      type: AverageValue
      averageValue: 100
```

---

### **3. Métriques Externes**
Les **métriques externes** sont des indicateurs qui proviennent de sources extérieures au cluster Kubernetes. Cela peut inclure des métriques issues de services cloud, de services managés ou de systèmes de monitoring externes. Par exemple, vous pouvez autoscaler en fonction d'une métrique de performance issue d'AWS CloudWatch, Azure Monitor, ou même d'une API externe personnalisée.

Exemples de métriques externes :
- **Nombre de messages dans une file SQS (AWS)**.
- **Taille d'une base de données hébergée dans un service cloud**.
- **Taux de requêtes d'un système de surveillance externe**.

**Exemple de configuration pour une métrique externe (file d'attente) :**
```yaml
metrics:
- type: External
  external:
    metric:
      name: queue-length
      selector:
        matchLabels:
          queue-name: task-queue
    target:
      type: Value
      value: 100
```

---

### **4. Métriques sur les Objets Kubernetes**
Les **object metrics** permettent d’autoscaler un déploiement en fonction d’une métrique d’un objet Kubernetes spécifique, comme un **Ingress**, un **Service**, ou un autre type d'objet. Cela vous permet de monitorer les objets Kubernetes directement, par exemple le nombre de requêtes gérées par un Ingress.

Exemples d'object metrics :
- **Nombre de requêtes HTTP par seconde sur un Ingress**.
- **Taille d'un cache Redis exposé via un service**.
- **État de la saturation d'un Load Balancer**.

**Exemple de configuration pour une métrique sur un Ingress :**
```yaml
metrics:
- type: Object
  object:
    metric:
      name: requests-per-second
    describedObject:
      apiVersion: networking.k8s.io/v1
      kind: Ingress
      name: my-ingress
    target:
      type: Value
      value: 1000
```

---

### **Résumé des Types d'Indicateurs dans HPA :**

1. **Métriques de ressources** (fournies par le Metrics Server) :
   - **CPU** : Utilisation du CPU (cores ou milli-cores).
   - **Mémoire** : Utilisation de la mémoire (MiB, GiB).

2. **Métriques personnalisées** (nécessite un adaptateur, comme Prometheus Adapter) :
   - Tout type de métriques collectées par votre application ou service (ex : transactions, requêtes par seconde, latence, etc.).

3. **Métriques externes** (fournies par des services externes au cluster Kubernetes) :
   - Des métriques provenant de services cloud (AWS, Azure, etc.) ou d'API externes (ex : longueur de file d'attente, taux de requêtes).

4. **Métriques sur les objets Kubernetes** :
   - Indicateurs basés sur les objets Kubernetes, tels que le nombre de requêtes d'un Ingress ou l'état d'un Service.

---

### Outils pour Collecter et Fournir les Métriques

Pour les **métriques personnalisées** et **externes**, vous aurez besoin de configurer des adaptateurs dans Kubernetes pour exposer ces métriques au HPA :

- **Prometheus Adapter** : Permet de récupérer les métriques de Prometheus et de les exposer au HPA.
- **Custom Metrics Adapter** : Un framework pour exposer des métriques personnalisées à Kubernetes.
- **External Metrics Adapter** : Utilisé pour récupérer des métriques de systèmes externes (comme AWS CloudWatch ou une API tierce).

---

### Conclusion

L'HPA de Kubernetes est extrêmement flexible, permettant de baser les décisions de scaling sur plusieurs types de métriques. Que vous souhaitiez autoscaler en fonction du CPU, de la mémoire, ou de métriques spécifiques à vos applications ou services, vous pouvez configurer l'HPA pour répondre à vos besoins.