minikube start

minikube addons enable ingress

minikube addons enable ingress-dns

Wait until you see the ingress-nginx-controller-XXXX is up and running using Kubectl get pods -n ingress-nginx

Create an ingress using the K8s example yaml file

Update the service section to point to the NodePort Service that you already created

Append 127.0.0.1 hello-world.info to your /etc/hosts file on MacOS (NOTE: Do NOT use the Minikube IP)

Run minikube tunnel ( Keep the window open. After you entered the password there will be no more messages, and the cursor just blinks)

Hit the hello-world.info ( or whatever host you configured in the yaml file) in a browser and it should work