Voici un récapitulatif des principales commandes sur Kubernetes pour la gestion des ressources courantes :

### 1. **Pods**
- **Lister les pods :**
  ```bash
  kubectl get pods
  ```
- **Créer un pod à partir d’un fichier YAML :**
  ```bash
  kubectl apply -f pod.yaml
  ```
- **Obtenir des détails sur un pod spécifique :**
  ```bash
  kubectl describe pod <pod-name>
  ```
- **Supprimer un pod :**
  ```bash
  kubectl delete pod <pod-name>
  ```

### 2. **Deployments**
- **Lister les déploiements :**
  ```bash
  kubectl get deployments
  ```
- **Créer un déploiement à partir d’un fichier YAML :**
  ```bash
  kubectl apply -f deployment.yaml
  ```
- **Mettre à jour un déploiement (rolling update) :**
  ```bash
  kubectl set image deployment/<deployment-name> <container-name>=<new-image>
  ```
- **Supprimer un déploiement :**
  ```bash
  kubectl delete deployment <deployment-name>
  ```

### 3. **Services**
- **Lister les services :**
  ```bash
  kubectl get services
  ```
- **Créer un service à partir d’un fichier YAML :**
  ```bash
  kubectl apply -f service.yaml
  ```
- **Obtenir des détails sur un service spécifique :**
  ```bash
  kubectl describe service <service-name>
  ```
- **Supprimer un service :**
  ```bash
  kubectl delete service <service-name>
  ```

### 4. **ConfigMaps**
- **Lister les ConfigMaps :**
  ```bash
  kubectl get configmaps
  ```
- **Créer un ConfigMap à partir d’un fichier YAML :**
  ```bash
  kubectl apply -f configmap.yaml
  ```
- **Obtenir des détails sur un ConfigMap spécifique :**
  ```bash
  kubectl describe configmap <configmap-name>
  ```
- **Supprimer un ConfigMap :**
  ```bash
  kubectl delete configmap <configmap-name>
  ```

### 5. **Secrets**
- **Lister les secrets :**
  ```bash
  kubectl get secrets
  ```
- **Créer un secret à partir d’un fichier YAML :**
  ```bash
  kubectl apply -f secret.yaml
  ```
- **Obtenir des détails sur un secret spécifique :**
  ```bash
  kubectl describe secret <secret-name>
  ```
- **Supprimer un secret :**
  ```bash
  kubectl delete secret <secret-name>
  ```

### 6. **Namespaces**
- **Lister les namespaces :**
  ```bash
  kubectl get namespaces
  ```
- **Créer un namespace à partir d’un fichier YAML :**
  ```bash
  kubectl apply -f namespace.yaml
  ```
- **Supprimer un namespace :**
  ```bash
  kubectl delete namespace <namespace-name>
  ```

### 7. **StatefulSets**
- **Lister les StatefulSets :**
  ```bash
  kubectl get statefulsets
  ```
- **Créer un StatefulSet à partir d’un fichier YAML :**
  ```bash
  kubectl apply -f statefulset.yaml
  ```
- **Supprimer un StatefulSet :**
  ```bash
  kubectl delete statefulset <statefulset-name>
  ```

### 8. **DaemonSets**
- **Lister les DaemonSets :**
  ```bash
  kubectl get daemonsets
  ```
- **Créer un DaemonSet à partir d’un fichier YAML :**
  ```bash
  kubectl apply -f daemonset.yaml
  ```
- **Supprimer un DaemonSet :**
  ```bash
  kubectl delete daemonset <daemonset-name>
  ```

### 9. **Ingress**
- **Lister les Ingress :**
  ```bash
  kubectl get ingress
  ```
- **Créer un Ingress à partir d’un fichier YAML :**
  ```bash
  kubectl apply -f ingress.yaml
  ```
- **Supprimer un Ingress :**
  ```bash
  kubectl delete ingress <ingress-name>
  ```

### 10. **Jobs/CronJobs**
- **Lister les jobs :**
  ```bash
  kubectl get jobs
  ```
- **Lister les CronJobs :**
  ```bash
  kubectl get cronjobs
  ```
- **Créer un Job/CronJob à partir d’un fichier YAML :**
  ```bash
  kubectl apply -f job.yaml
  kubectl apply -f cronjob.yaml
  ```
- **Supprimer un Job/CronJob :**
  ```bash
  kubectl delete job <job-name>
  kubectl delete cronjob <cronjob-name>
  ```

### 11. **PersistentVolume (PV) et PersistentVolumeClaim (PVC)**
- **Lister les PV :**
  ```bash
  kubectl get pv
  ```
- **Lister les PVC :**
  ```bash
  kubectl get pvc
  ```
- **Créer un PV/PVC à partir d’un fichier YAML :**
  ```bash
  kubectl apply -f pv.yaml
  kubectl apply -f pvc.yaml
  ```
- **Supprimer un PV/PVC :**
  ```bash
  kubectl delete pv <pv-name>
  kubectl delete pvc <pvc-name>
  ```

Ces commandes couvrent les principales ressources dans Kubernetes. Vous pouvez également obtenir des informations supplémentaires et approfondir l'usage des commandes avec l’option `-o` pour le format de sortie (json, yaml, wide), ou en ajoutant `--help` à chaque commande pour afficher plus de détails.