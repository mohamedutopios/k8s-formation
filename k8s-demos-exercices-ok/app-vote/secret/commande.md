

kubectl create secret generic postgres-secret \
  --from-literal=POSTGRES_USER=postgres \
  --from-literal=POSTGRES_PASSWORD=postgres


apiVersion: v1
kind: Secret
metadata:
  name: postgres-secret
type: Opaque
data:
  POSTGRES_USER: bXl1c2Vy  # 'myuser' encodé en base64
  POSTGRES_PASSWORD: bXlwYXNzd29yZA==  # 'mypassword' encodé en base64


kubectl apply -f postgres-secret.yaml
