# HorizontalPodAutoscaler

Dans cet exercice, nous allons utiliser une ressource de type *HorizontalPodAutoscaler* afin d'augmenter, ou de diminuer, automatiquement le nombre de réplicas d'un Deployment en fonction de l'utilisation du CPU.

## Création d'un Deployment

Copiez le contenu suivant dans le fichier *deploy.yaml*.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: w3
spec:
  selector:
    matchLabels:
      app: w3
  replicas: 1
  template:
    metadata:
      labels:
        app: w3
    spec:
      containers:
        - image: nginx:1.20-alpine
          name: w3
          ports:
            - containerPort: 80
          resources:
            requests:
              cpu: 200m
```

