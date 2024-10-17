# Kubernetes

## Fondamentaux
- [x] Rappels docker
- [x] Images et conteneurs
- [x] Volumes et réseaux

## Introduction à Kubernetes
- [x] Bref historique
- [x] Principe d'orchestration de conteneur
- [x] Avantages de l'orchestration de conteneurs
- [x] Cloud Native Computing Foundation et Open Container Initiative

## Architecture et tour d'horizon des composants principaux de Kubernetes
- [x] Masters, nodes
- [x] Kubelet, API server, Kube-proxy, Kube DNS, Kube Scheduler, et Kube-Controller-manager

## Créer un cluster Kubernetes
- [x] Solutions possibles (clouds publiques, distributions tierces, minikube)
- [x] Créer son cluster avec minikube
- [x] Introduction à kubectl
- [x] Commandes et options de base
- [x] Obtenir les informations du cluster
- [ ] Contextes et namespaces

## Déployer sa première application
- [x] Introduction aux Pods
- [x] Que mettre dans un pod ?
- [x] Créer un pod avec kubectl
- [x] Introduction aux fichiers de manifestes
- [x] Metadata, labels, annotations, specs

## Suivi et diagnostic d'un pod en fonctionnement
- [x] Lister les pods et afficher les détails
- [x] Accès aux logs
- [x] Exécuter des commandes à l'intérieur du pod
- [ ] Échanger des fichiers avec le pod
- [ ] Sondes de disponibilité et de bonne santé (liveness et readiness probes)

## Accéder à ses applications
- [x] Comprendre le réseau virtuel de Kubernetes
- [x] Rendre son service accessible sur le réseau virtuel
- [x] Introduction aux Services et leurs différents types
- [x] Déployer un service NodePort et comprendre son fonctionnement
- [ ] Principe de CNI, exemple de Flannel et Calico

## Accéder à ses applications depuis l'extérieur
- [ ] Utiliser les ExternalIps
- [x] Principe des Ingress et de l'ingress Controller
- [x] Exemples d'ingress controllers : Nginx, Traefik
- [x] Services de type Load Balancer
- [ ] Accès aux services avec Kube DNS

## Réplication des pods
- [x] Objectifs de la réplication (redondance, passage à l'échelle, sharding)
- [x] Principe de boucle de réconciliation
- [x] Créer et manipuler des ReplicaSet
- [x] Principe de passage à l'échelle horizontal (vs vertical)
- [x] Mettre en place l'auto-scaling horizontal
- [x] Introduction aux DaemonSet

## Gérer le cycle de vie d'une application
- [x] Principe de Rolling update
- [x] Créer et gérer des Deployments
- [x] Stratégies de déploiement

## Lancer des tâches à exécution unique
- [ ] Principe et intérêt des Job
- [ ] Lancer des Job
- [ ] Parallélisme et work-pools

## Déploiement et partage des éléments de configuration
- [ ] Principe de généricité et d'indépendance de l'application et de la plateforme
- [ ] Créer des ConfigMaps
- [ ] Différents moyens de consommation des ConfigMaps
- [ ] Créer et consommer des Secrets

## Gérer les données persistantes et les applications stateful
- [ ] Problématique du stockage persistant dans le cloud
- [x] Introduction aux volumes, tour d'horizon des différents types de volumes
- [x] Créer une application en exploitant un volume emptyDir
- [ ] Monter un ConfigMap en tant que volume
- [ ] Utiliser du stockage distant
- [x] Tour d'horizon de quelques solutions de stockage distant
- [x] Introduction aux StatefulSets
- [ ] Déployer Kafka avec un StatefulSet

## Sécurité dans Kubernetes
- [ ] Introduction et manipulation de l'api RBAC
- [ ] Introduction aux NetworkPolicies
- [ ] Cloisonner une application avec une NetworkPolicy
- [ ] Introduction aux PodSecurityPolicies

## Kubernetes et son écosystème
- [ ] Déployer des solutions avec Helm
- [ ] Etcd, gestion de configuration distribuée

## Cas d'exploitation courant, outils de diagnostic
- [x] Port forwarding avec kubectl
- [ ] Mettre un pod défaillant en quarantaine
- [ ] Diagnostiquer un problème d'accès réseau
- [x] Redémarrer rapidement une application répliquée
- [ ] Isoler une application du point de vue réseau
