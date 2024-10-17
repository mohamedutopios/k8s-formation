Voici un guide détaillé pour mettre en place **Prometheus** et **Grafana** dans votre projet **vote** afin de collecter et visualiser les métriques pour vos services **result-ui**, **db**, et autres composants de votre application.

### **1. Installer Helm (si ce n'est pas déjà fait)**

**Helm** est un outil de gestion des packages pour Kubernetes, qui facilite l'installation de Prometheus et Grafana.

- **Installer Helm** :
  
  Sur Linux/MacOS :
  ```bash
  curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
  ```

### **2. Créer un Namespace pour Monitoring**

Il est recommandé de créer un namespace spécifique pour isoler Prometheus et Grafana.

```bash
kubectl create namespace monitoring
```

### **3. Ajouter les Repositories Helm pour Prometheus et Grafana**

Ajoutez les dépôts Helm pour installer Prometheus et Grafana :

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
```

### **4. Installer Prometheus**

Nous allons installer la **kube-prometheus-stack**, qui est une stack complète incluant Prometheus, Alertmanager et les exporters nécessaires pour Kubernetes.

```bash
helm install prometheus prometheus-community/kube-prometheus-stack -n monitoring
```

Cela va installer :
- **Prometheus** : pour collecter les métriques.
- **kube-state-metrics** : pour les métriques Kubernetes (pods, nœuds, etc.).
- **Node Exporter** : pour monitorer les nœuds.
- **Prometheus Operator** : pour gérer l'installation de Prometheus.

### **5. Installer Grafana**

Une fois que Prometheus est installé, installez Grafana dans le même namespace.

```bash
helm install grafana grafana/grafana -n monitoring
```

### **6. Configurer les Accès à Grafana**

Après l'installation, récupérez le mot de passe administrateur pour Grafana :

```bash
kubectl get secret --namespace monitoring grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```

Accédez à Grafana via un port-forwarding sur votre machine locale :

```bash
kubectl port-forward -n monitoring svc/grafana 3000:80
```

Ensuite, ouvrez un navigateur et accédez à `http://localhost:3000`. Utilisez le nom d'utilisateur `admin` et le mot de passe récupéré pour vous connecter.

### **7. Ajouter Prometheus comme Source de Données dans Grafana**

1. Allez dans **Configuration** → **Data Sources**.
2. Cliquez sur **Add data source**.
3. Sélectionnez **Prometheus**.
4. Dans l'URL, entrez :
   ```
   http://prometheus-kube-prometheus-prometheus.monitoring.svc.cluster.local:9090
   ```
5. Cliquez sur **Save & Test**.

### **8. Exposer des Métriques pour `result-ui` et `db`**

#### a) **Exposer des métriques pour `result-ui`**

Si votre service `result-ui` n'expose pas encore de métriques Prometheus, vous pouvez ajouter un endpoint `/metrics` comme vu précédemment. Par exemple, pour une application Node.js, vous pouvez utiliser la bibliothèque **prom-client** pour exposer les métriques.

Une fois l'endpoint `/metrics` configuré, ajoutez les annotations suivantes à votre déploiement `result-ui` pour que Prometheus puisse scraper les métriques :

```yaml
metadata:
  annotations:
    prometheus.io/scrape: 'true'
    prometheus.io/port: '80'
    prometheus.io/path: '/metrics'
```

#### b) **Exposer des métriques pour `db` (PostgreSQL)**

Pour surveiller la base de données **PostgreSQL**, vous pouvez utiliser un **PostgreSQL Exporter**. Vous devez ajouter un second conteneur dans le même pod que PostgreSQL pour exporter les métriques.

Voici comment modifier le déploiement `db` pour ajouter PostgreSQL Exporter :

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: db
  name: db
  namespace: vote
spec:
  replicas: 1
  selector:
    matchLabels:
      app: db
  template:
    metadata:
      labels:
        app: db
      annotations:
        prometheus.io/scrape: 'true'
        prometheus.io/port: '9187'
        prometheus.io/path: '/metrics'
    spec:
      containers:
        - image: postgres:13.2-alpine
          name: postgres
          env:
            - name: POSTGRES_USER
              valueFrom:
                secretKeyRef:
                  name: postgres-secret
                  key: POSTGRES_USER
            - name: POSTGRES_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: postgres-secret
                  key: POSTGRES_PASSWORD
          ports:
            - containerPort: 5432
        - image: wrouesnel/postgres_exporter
          name: postgres-exporter
          env:
            - name: DATA_SOURCE_NAME
              value: "postgresql://$(POSTGRES_USER):$(POSTGRES_PASSWORD)@localhost:5432/postgres"
```

Cela permet à Prometheus de scraper les métriques sur le port `9187`.

### **9. Vérifier les Métriques dans Prometheus**

Pour vérifier que Prometheus scrape correctement les métriques, vous pouvez accéder à l'interface Prometheus avec un port-forwarding :

```bash
kubectl port-forward -n monitoring svc/prometheus-kube-prometheus-prometheus 9090:9090
```

Ensuite, accédez à `http://localhost:9090` et cherchez les métriques que vous voulez visualiser, par exemple :

- Pour `result-ui` :
  ```bash
  http_requests_total
  ```
- Pour la base de données `db` :
  ```bash
  pg_stat_activity_count
  ```

### **10. Créer des Dashboards dans Grafana**

Une fois que vous avez configuré Prometheus comme source de données dans Grafana, vous pouvez soit :

- **Importer des tableaux de bord prédéfinis** pour Kubernetes et PostgreSQL.
  - Exemple de tableau de bord pour PostgreSQL : Dashboard ID **9628** sur le site de Grafana.
  - Exemple de tableau de bord pour Kubernetes : Dashboard ID **315** (Kubernetes cluster monitoring).

- **Créer vos propres tableaux de bord** :
  - Allez dans **Create** → **Dashboard** → **Add New Panel**.
  - Utilisez les requêtes Prometheus pour les métriques de `result-ui` et `db`.

### **11. Nettoyage des Tests (optionnel)**

Une fois que tout est configuré et fonctionnel, vous pouvez vérifier et nettoyer les ressources temporaires si nécessaire. Prometheus et Grafana seront configurés pour surveiller vos services et fournir des métriques en temps réel.

### **Conclusion**

Vous avez maintenant un système de monitoring complet avec **Prometheus** et **Grafana** intégré dans votre projet **vote**. Vous pouvez collecter et visualiser des métriques pour **result-ui**, **db**, et d'autres services. Si vous rencontrez des problèmes ou avez des questions supplémentaires, n'hésitez pas à demander.

Pour obtenir des **métriques du cluster Kubernetes** avec **Prometheus** et les visualiser dans **Grafana**, vous pouvez utiliser les données collectées via **kube-state-metrics** et **cadvisor**. Ces outils, intégrés dans la stack **kube-prometheus-stack**, permettent de collecter des informations sur les ressources utilisées dans votre cluster (pods, nœuds, conteneurs, etc.).

Voici comment configurer Prometheus et Grafana pour collecter et visualiser ces **métriques du cluster** Kubernetes.

### **1. Installation de Prometheus pour le Monitoring du Cluster**

Si vous avez déjà installé **Prometheus** avec la **kube-prometheus-stack**, cela inclut par défaut **kube-state-metrics** et **Node Exporter**, qui collectent des métriques sur les objets du cluster et l'état des nœuds.

Sinon, voici un rappel pour installer Prometheus via Helm :

```bash
kubectl create namespace monitoring
helm install prometheus prometheus-community/kube-prometheus-stack -n monitoring
```

Cela va automatiquement configurer Prometheus pour collecter les métriques suivantes :
- **Métriques des nœuds** : CPU, mémoire, utilisation du disque, etc.
- **Métriques des pods** : Consommation CPU, mémoire, réseau, etc.
- **État des objets Kubernetes** : Pods, services, déploiements, etc.

### **2. Visualiser les Métriques du Cluster dans Grafana**

1. **Accéder à Grafana** (si ce n'est pas déjà fait) :
   - Si vous avez installé Grafana via Helm, accédez à l'interface Grafana via un port-forwarding :
   
     ```bash
     kubectl port-forward -n monitoring svc/grafana 3000:80
     ```

     Puis ouvrez `http://localhost:3000` dans votre navigateur. Connectez-vous avec le nom d'utilisateur `admin` et le mot de passe récupéré (voir les étapes précédentes).

2. **Ajouter Prometheus comme source de données** :
   - Allez dans **Configuration** → **Data Sources** → **Add data source**.
   - Sélectionnez **Prometheus** et entrez l'URL suivante :
     ```
     http://prometheus-kube-prometheus-prometheus.monitoring.svc.cluster.local:9090
     ```
   - Sauvegardez la configuration.

3. **Importer des tableaux de bord prédéfinis pour Kubernetes** :
   Grafana propose plusieurs tableaux de bord prédéfinis pour le monitoring des clusters Kubernetes. Vous pouvez les importer directement dans Grafana.

   Voici quelques exemples de tableaux de bord populaires :
   
   - **Kubernetes Cluster Monitoring (via Prometheus)** : Dashboard ID **315**.
   - **Kubernetes Node Exporter Full** : Dashboard ID **1860**.
   - **Kubernetes Pods Monitoring** : Dashboard ID **6417**.
   
   **Étapes pour importer un tableau de bord** :
   - Dans Grafana, allez dans **Dashboards** → **Manage** → **Import**.
   - Entrez l'ID du tableau de bord (ex. : `315` pour le cluster monitoring).
   - Cliquez sur **Load** et sélectionnez Prometheus comme source de données.
   
   Ces tableaux de bord vous permettront de visualiser les métriques du cluster telles que :
   - **Utilisation du CPU et de la mémoire par nœud et par pod**.
   - **Statut des nœuds (Ready, Not Ready)**.
   - **Utilisation du réseau**.
   - **Nombre de pods et leur statut** (Running, Pending, Failed, etc.).

### **3. Exemples de Métriques du Cluster Disponibles dans Prometheus**

Vous pouvez également créer vos propres tableaux de bord et visualiser des métriques spécifiques du cluster. Voici quelques exemples de requêtes Prometheus que vous pouvez utiliser :

#### **a) Utilisation du CPU par nœud**
Cette métrique montre l'utilisation du CPU (en cores) pour chaque nœud du cluster.

```bash
sum(rate(container_cpu_usage_seconds_total{job="kubelet", image!=""}[5m])) by (instance)
```

#### **b) Utilisation de la mémoire par nœud**
Affiche la quantité de mémoire utilisée par chaque nœud.

```bash
sum(container_memory_usage_bytes{job="kubelet", image!=""}) by (instance)
```

#### **c) Nombre total de pods dans le cluster**
Cela montre le nombre total de pods par namespace ou dans l'ensemble du cluster.

```bash
count(kube_pod_info)
```

#### **d) Pods par statut (Running, Pending, etc.)**
Obtenez le nombre de pods dans différents états (Running, Pending, etc.).

```bash
count(kube_pod_status_phase) by (phase)
```

#### **e) État des nœuds Kubernetes**
Visualisez si vos nœuds sont en état "Ready" ou non.

```bash
kube_node_status_condition{condition="Ready",status="true"}
```

#### **f) Utilisation du réseau par pod**
Surveillez le trafic réseau entrant et sortant pour chaque pod.

```bash
sum(rate(container_network_receive_bytes_total[5m])) by (pod)
sum(rate(container_network_transmit_bytes_total[5m])) by (pod)
```

### **4. Créer des Graphiques Personnalisés dans Grafana**

1. **Créer un nouveau tableau de bord** :
   - Dans Grafana, allez dans **Create** → **Dashboard** → **Add new panel**.
   
2. **Ajouter une requête Prometheus** :
   - Sélectionnez **Prometheus** comme source de données.
   - Entrez une requête Prometheus, par exemple :
     - **Utilisation CPU par nœud** :
       ```bash
       sum(rate(container_cpu_usage_seconds_total{job="kubelet"}[5m])) by (instance)
       ```
     - **Nombre de pods en état Running** :
       ```bash
       count(kube_pod_status_phase{phase="Running"})
       ```
   
3. **Configurer l'affichage** :
   - Personnalisez l'apparence du graphique : type (ligne, barre, jauge), échelle, unité (cores, MB, etc.).
   - Sauvegardez le tableau de bord pour une surveillance continue.

### **5. Alertes avec Prometheus et Grafana**

Une autre fonctionnalité importante est la configuration des alertes dans Grafana ou directement dans Prometheus. Par exemple, vous pouvez créer des alertes pour :
- Avertir si un nœud utilise plus de 80 % de son CPU.
- Alerter si un pod est en échec ou s'il dépasse un certain seuil de consommation de mémoire.

Pour configurer les alertes dans Prometheus :
1. Créez une règle d'alerte Prometheus (dans le fichier de configuration de Prometheus) :
   ```yaml
   groups:
   - name: example-alert
     rules:
     - alert: HighCPUUsage
       expr: sum(rate(container_cpu_usage_seconds_total{job="kubelet"}[5m])) by (instance) > 0.8
       for: 2m
       labels:
         severity: critical
       annotations:
         summary: "High CPU usage detected on {{ $labels.instance }}"
         description: "The node {{ $labels.instance }} is using more than 80% of its CPU capacity."
   ```

2. Configurez **Alertmanager** pour envoyer des notifications (Slack, email, etc.).

### **Conclusion**

En suivant ces étapes, vous avez maintenant Prometheus et Grafana installés dans votre cluster **vote**, et vous pouvez surveiller une variété de **métriques du cluster** : utilisation du CPU, de la mémoire, réseau, état des nœuds, nombre de pods, etc. Grafana vous permet d'importer des tableaux de bord prédéfinis ou de créer vos propres visualisations personnalisées pour une surveillance en temps réel.

Si vous avez des questions supplémentaires ou besoin d'aide pour configurer certaines requêtes ou alertes, n'hésitez pas à demander !