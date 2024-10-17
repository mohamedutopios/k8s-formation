# Afficher les paramètres fusionnés de la configuration kubeconfig.
`kubectl config view`

# Utilisez plusieurs fichiers kubeconfig en même temps et affichez la configuration fusionnée.
`KUBECONFIG=~/.kube/config:~/.kube/kubconfig2`
`kubectl config view`

# Obtenez le mot de passe pour l'utilisateur e2e.
`kubectl config view -o jsonpath='{.users[?(@.name == "e2e")].user.password}'`

# Affichez le nom du premier utilisateur.
`kubectl config view -o jsonpath='{.users[].name}'`

# Obtenez une liste des noms d'utilisateurs.
`kubectl config view -o jsonpath='{.users[*].name}'`

# Affichez la liste des contextes.
`kubectl config get-contexts`

# Affichez le contexte actuel.
`kubectl config current-context`

# Définissez le contexte par défaut sur my-cluster-name.
`kubectl config use-context my-cluster-name`

# Définissez une entrée de cluster dans la kubeconfig.
`kubectl config set-cluster my-cluster-name`


# Enregistrez définitivement l'espace de noms pour toutes les commandes kubectl ultérieures dans ce contexte.
`kubectl config set-context --current --namespace=ggckad-s2`


1. Pour supprimer un context, utilisez la commande suivante :
   ```
   kubectl config delete-context <nom-du-contexte>
   ```
   Remplacez `<nom-du-contexte>` par le nom du contexte que vous souhaitez supprimer. Par exemple, si votre contexte s'appelle "my-context", vous pouvez le supprimer avec :
   ```
   kubectl config delete-context my-context
   ```

2. Pour supprimer un cluster, utilisez la commande suivante :
   ```
   kubectl config delete-cluster <nom-du-cluster>
   ```
   Remplacez `<nom-du-cluster>` par le nom du cluster que vous souhaitez supprimer. Par exemple, si votre cluster s'appelle "my-cluster", vous pouvez le supprimer avec :
   ```
   kubectl config delete-cluster my-cluster
   ```
