1. Afficher la liste des nœuds dans le cluster :
   ```
   kubectl get nodes
   ```

2. Afficher des informations détaillées sur un nœud spécifique :
   ```
   kubectl describe node <nom-du-noeud>
   ```

3. Marquer un nœud comme étant en maintenance (cordon) pour empêcher de nouvelles charges de travail d'y être planifiées :
   ```
   kubectl cordon <nom-du-noeud>
   ```

4. Annuler le marquage de maintenance d'un nœud (reprendre la planification) :
   ```
   kubectl uncordon <nom-du-noeud>
   ```

5. Étiqueter un nœud avec des balises personnalisées pour l'identification ou le regroupement :
   ```
   kubectl label nodes <nom-du-noeud> <clé>=<valeur>
   ```

6. Supprimer une étiquette d'un nœud :
   ```
   kubectl label nodes <nom-du-noeud> <clé>-
   ```

7. Tuer un nœud de manière abrupte (non recommandé, car cela peut entraîner la perte de données) :
   ```
   kubectl delete node <nom-du-noeud>
   ```

8. Drainer un nœud avant de le mettre hors service pour la maintenance :
   ```
   kubectl drain <nom-du-noeud>
   ```

9. Mettre à jour la configuration d'un nœud, par exemple pour désactiver ou activer des fonctionnalités spécifiques :
   ```
   kubectl edit node <nom-du-noeud>
   ```

10. Éviter que de nouvelles charges de travail soient planifiées sur un nœud (pour des raisons de maintenance ou de dépannage) :
    ```
    kubectl cordon <nom-du-noeud>
    ```

11. Vérifier l'état de santé d'un nœud :
    ```
    kubectl get node <nom-du-noeud> -o jsonpath='{.status.conditions[?(@.type=="Ready")].status}'
    ```