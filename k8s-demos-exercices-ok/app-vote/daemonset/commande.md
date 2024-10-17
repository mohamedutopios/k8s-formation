
kubectl label nodes minikube-m02 node-role=minikube-m04

apiVersion: apps/v1
kind: Deployment
metadata:
  name: vote
spec:
  replicas: 1
  selector:
    matchLabels:
      app: vote
  template:
    metadata:
      labels:
        app: vote
    spec:
      containers:
        - name: vote
          image: mohamed1780/vote
          ports:
            - containerPort: 80
      nodeSelector:
        node-role: minikube-m04


kubectl apply -f simple-info-daemonset-fixed.yaml

kubectl logs simple-info-daemonset-<pod-name>

Node: minikube
Pod Name: simple-info-daemonset-abc123
Namespace: default
Node Name: minikube
Sleeping for 30 seconds...

