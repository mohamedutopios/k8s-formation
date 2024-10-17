Dans cet exercice, vous allez ajouter une contrainte afin de déployer un Pod sur un node en particulier.

## 1. Définition d'un Pod Mysql

La spécification suivante définie un Pod contenant un container *mysql*.

```
apiVersion: v1
kind: Pod
metadata:
  name: db
spec:
  containers:
  - image: mysql:5.7
    name: mysql
    env:
    - name: MYSQL_ROOT_PASSWORD
      value: mysqlpwd
    volumeMounts:
    - name: data
      mountPath: /var/lib/mysql
  volumes:
  - name: data
    emptyDir: {}
```

## 2. Ajout d'une contrainte de déploiement

Pour des raisons de performances, nous souhaitons déployer ce Pod sur un node contenant un disk SSD. Nous allons alors ajouter cette contrainte de déploiement via la clé *affinity* dans la spécification du Pod.

## 3. Déploiement du Pod

Lancez la commande suivante afin de créer le Pod.

puis vérifiez si le Pod a été correctement créé:

Vous devriez observer que le Pod reste dans l'état *Pending*. On peut en connaitre la raison en listant les évènements qui se sont déroulés suite à la demande de création:

Etant donné qu'aucun node ne porte le label *disktype: ssd*, la contrainte de déploiement ne peut pas être respecté, le Pod ne peut pas être déployé.

## 4. Ajout d'un label sur l'un des nodes du cluster

Utilisez la commande afin d'ajouter un label sur l'un des nodes du cluster.

Après quelques secondes, lancez la commande afin de vérifier le status du Pod

Lancez un nouvelle fois le describe du Pod. Vous pourrez cette fois observer que le Pod a pu être schédulé sur le node sur lequel le label a été posé.

