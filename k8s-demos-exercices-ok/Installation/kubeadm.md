Dans cet exercice, vous allez mettre en place un cluster Kubernetes à l'aide de l'utilitaire *kubeadm*.

## Création des VMs

Afin de créer les VMs qui seront utilisées dans le cluster, vous allez utiliser [Multipass](https://multipass.run), un outil qui permet de créer des machines virtuelles très simplement.

Installez Multipass (celui-ci est disponible pour Windows, Linux et MacOS) puis lancez les commandes suivantes pour créer 2 machines virtuelles nommées *control-plane* et *worker*

```
multipass launch -n control-plane -m 2G -c 2 -d 10G
multipass launch -n worker -m 2G -c 2 -d 10G
```

Note: par défaut Multipass crée des VMs en limitant leurs ressources à 1 cpu, 1G RAM et 5G de disque. Les commandes ci-dessus donnent un peu plus de ressources que les valeurs par défaut.

## Installation de Kubectl

Assurez-vous d'avoir installé *kubectl* sur la machine depuis laquelle vous avez lancé les commandes Multipass. *kubectl*  permet de communiquer avec un cluster Kubernetes depuis la ligne de commande.

## Initialisation du cluster

Une fois les VMs créées, vous allez initialiser le cluster.

Lancez tout d'abord un shell sur *control-plane*:

```
multipass shell control-plane
```

Depuis ce shell lancez la commande suivante, celle-ci installe les dépendances nécessaires (container runtime et quelques packages)

```
curl -sSL https://luc.run/kubeadm/master.sh | bash
```

Toujours depuis le shell sur la VM *control-plane* lancez ensuite la commande qui initialize le cluster:

```
sudo kubeadm init --ignore-preflight-errors=NumCPU,Mem
```

Note: le flag *--ignore-preflight-errors* permet de forcer l'installation même si les requirements en terme de CPU et Memoire ne sont pas respectés (cette option ne sera évidememnt pas utilisée pour la mise en place d'un cluster de production)

Après quelques dizaines de secondes, vous obtiendrez alors une commande qui vous servira, par la suite, à ajouter un node worker au cluster qui vient d'être créé.

Exemple de commande retournée (les tokens que vous obtiendrez seront différents):

```
sudo kubeadm join 192.168.64.40:6443 --token xrtqvq.9zmmzjx16b4jc4q8 --discovery-token-ca-cert-hash sha256:fabe1bbc0264b36a624b0c7284fe58151dad0640c81bff9a9f0e33fecd377e1a
```

Récupérez le fichier kubeconfig pour l'utilisateur courant (*ubuntu*):

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## Ajout d'un node worker

Une fois le cluster initialisé, vous allez ajouter un node worker.

Lancez tout d'abord un shell sur *worker*:

```
multipass shell worker
```

Depuis ce shell lancez la commande suivante, celle-ci installe les dépendances nécessaires sur le worker:

```
curl -sSL https://luc.run/kubeadm/worker.sh | bash
```

Lancez ensuite la commande retournée lors de l'étape d'initialisation (*sudo kubeadm join ...*) afin d'ajouter le node *worker* au cluster.

```
sudo kubeadm join 192.168.64.40:6443 --token xrtqvq.9zmmzjx16b4jc4q8 --discovery-token-ca-cert-hash sha256:fabe1bbc0264b36a624b0c7284fe58151dad0640c81bff9a9f0e33fecd377e1a
```

Note: si vous avez perdu la commande d'ajout de node, vous pouvez la générer avec la commande suivante (à lancer depuis le node control-plane)

```
sudo kubeadm token create --print-join-command
```

Après quelques dizaines de secondes, vous obtiendrez rapidement une confirmation indiquant que la VM *worker* fait maintenant partie du cluster::


```
This node has joined the cluster
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.
...
```

## Etat du cluster

Listez à présent les nodes du cluster avec la commande suivante depuis un shell sur le node *control-plane*:

```
ubuntu@control-plane:~$ kubectl get nodes
NAME            STATUS     ROLES           AGE     VERSION
control-plane   NotReady   control-plane   3m48s   v1.25.2
worker          NotReady   <none>          52s     v1.25.2
```

Les nodes sont dans l'état *NotReady*, cela vient du fait qu'aucun plugin network n'a été installé pour le moment.

## Plugin network

Afin que le cluster soit opérationnel il est nécessaire d'installer un plugin network. Plusieurs plugins sont disponibles (WeaveNet, Calico, Flannel, Cilium, …), chacun implémente la spécification CNI (Container Network Interface) et permet notamment la communication entre les différents Pods du cluster.

Dans cet exercice, vous allez installer le plugin WeaveNet. Utilisez pour cela La commande suivante:

```
ubuntu@control-plane:~$ kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s-1.11.yaml
```

Plusieurs ressources sont alors créées pour mettre en place cette solution de networking:

```
serviceaccount/weave-net created
clusterrole.rbac.authorization.k8s.io/weave-net created
clusterrolebinding.rbac.authorization.k8s.io/weave-net created
role.rbac.authorization.k8s.io/weave-net created
rolebinding.rbac.authorization.k8s.io/weave-net created
daemonset.apps/weave-net created
```

Après quelques secondes, les nodes apparaitront dans l'état *Ready*

```
ubuntu@control-plane:~$ kubectl get nodes
NAME            STATUS   ROLES           AGE     VERSION
control-plane   Ready    control-plane   4m42s   v1.25.2
worker          Ready    <none>          106s    v1.25.2
```

Le cluster est prêt à être utilisé.

## Récupération du context

Afin de pouvoir dialoguer avec le cluster via le binaire *kubectl* que vous avez installé sur votre machine locale, il est nécessaire de récupérer le fichier de configuration généré lors de l'installation.

Pour cela, il faut récupérer le fichier */etc/kubernetes/admin.conf* présent sur le node control-plane et le copier sur votre machine locale.

Avec Multipass vous pouvez récupérer le fichier de configuration avec la commande suivante (il sera alors sauvegardé dans le fichier *kubeconfig* du répertoire courant):

```
multipass exec control-plane -- sudo cat /etc/kubernetes/admin.conf > kubeconfig
```

Une fois que le fichier est présent en local, il faut simplement indiquer à *kubectl* ou il se trouve en positionnant la variable d'environnement *KUBECONFIG*:

```
export KUBECONFIG=$PWD/kubeconfig
```

Listez une nouvelle fois les nodes du cluster.

```
$ kubectl get nodes
NAME            STATUS   ROLES           AGE     VERSION
control-plane   Ready    control-plane   5m28s   v1.25.2
worker          Ready    <none>          2m32s   v1.25.2
```

Vous pouvez à présent communiquer avec le cluster depuis votre machine locale et non depuis une connexion ssh sur le node control-plane.

## En résumé

Le cluster que vous avez mis en place dans cet exercice contient un node control-plane et 1 node worker. Il est également possible avec kubeadm de mettre en place un cluster HA avec plusieurs nodes control-plane en charge de la gestion de l'état du cluster. Kubeadm est l'une des référence pour la mise en place et la gestion de cluster en environnement de production.
