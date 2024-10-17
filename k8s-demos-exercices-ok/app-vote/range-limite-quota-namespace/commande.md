kubectl create namespace vote

kubectl create secret generic postgres-secret \
  --from-literal=POSTGRES_USER=postgres \
  --from-literal=POSTGRES_PASSWORD=postgres -n vote

kubectl apply -f range-limite-quota-namespace

kubectl get pods -n vote

kubectl describe limitrange vote-limits -n vote
kubectl describe resourcequota vote-quota -n vote

Exemple : 

kubectl scale deployment db --replicas=8 -n vote

kubectl get deployment db -n vote -o yaml : -> error : on en aura que 6.

lastTransitionTime: "2024-09-21T20:31:38Z"
    lastUpdateTime: "2024-09-21T20:31:38Z"
    message: 'pods "db-59d877b8d4-twstc" is forbidden: exceeded quota: vote-quota,
      requested: limits.cpu=200m,limits.memory=256Mi,requests.cpu=100m,requests.memory=128Mi,
      used: limits.cpu=1400m,limits.memory=1792Mi,requests.cpu=700m,requests.memory=896Mi,
      limited: limits.cpu=1400m,limits.memory=1792Mi,requests.cpu=700m,requests.memory=896Mi'
    reason: FailedCreate






