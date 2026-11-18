# Rendu — Atelier Kubernetes

> Complétez ce fichier au fil de l'atelier, puis faites un commit et un push (voir la section « Rendu » du README).

**Nom / Prénom : BRAZ DA COSTA-RAMOS Emma**
**Groupe : RS2 ING3**

---

## Exercice 1 — Cluster

1. Combien de nœuds affiche `kubectl get nodes` ? Quel est le rôle de chacun ?
   > La commande affiche 3 noeuds en ready. Il y a le k3d-lab-server-0 qui a comme role le control-plane qui va décider ou placer les pods et maintenir l'état du cluster, les deux autres sont en <none> mais ce sont des workers qui exécutent les pods (k3d-lab-agent-0 et k3d-lab-agent-1).
2. Que montre `docker ps` ? Qu'est-ce qu'un « nœud » dans k3d, concrètement ?
   > docker ps montre 4 conteneurs : k3d-lab-server-0, k3d-lab-agent-0, k3d-lab-agent-1  et k3d-lab-serverlb. Dans k3d, un nœud est simplement un conteneur Docker qui fait tourner k3s : 1 conteneur = 1 nœud.

## Exercice 2 — Premier déploiement

3. Quels objets Kubernetes ont été créés par `kubectl create deployment` ? (indice : `kubectl get all`)
   >kubectl create deployment a créé 3 objets. Un Deployment web , un ReplicaSet web-68d995574f  pour maintenir le bon nombre de pods et un Pod web-68d995574f-bsl4q qui fait tourner le conteneur nginx. 
4. Vous avez supprimé le pod `web-...` : que s'est-il passé, et pourquoi ?
   >Quand j’ai supprimé le pod web-68d995574f-bsl4q, il est passé en Terminating puis a disparu. Ensuite Kubernetes en a créé un nouveau, web-68d995574f-gznzn (Pending, ContainerCreating, puis Running en quelques secondes). Le nom du pod a donc changé. C'est grace au ReplicaSet qui doit  maintenir le nombre de pods demandé par le Deployment. Quand il constate qu’il n’y en a plus, il en recrée un.

## Exercice 3 — Modifier un conteneur en direct

5. Après suppression du pod, votre message « Bienvenue... » a-t-il survécu ? Pourquoi ?
   > Non, le message Bienvenue sur mon site Web n’a pas survécu et la page est revenue à  Welcome to nginx! On a modifié index.html directement dans le système de fichiers du conteneur, qui est éphémère. Quand le pod a été supprimé la modif a disparu avec lui et le nouveau pod est reparti de l’image nginx d’origine.
6. Pourquoi le `port-forward` a-t-il cessé de fonctionner après la suppression du pod ?
   >Le port-forward est branché sur un pod précis et pas sur le Service de façon permanente. Quand ce pod a été supprimé le tunnel a perdu sa cible et le processus s’est arrêté -> le navigateur affiche une erreur HTTP 502. Le nouveau pod a une autre identité donc il faut relancer le tunnel.

## Exercice 4 — Déclaratif

7. Quelle différence entre `kubectl create` et `kubectl apply` ?
   >
8. À quoi sert le `selector` d'un Service ? Que se passe-t-il s'il ne correspond à aucun pod ?
   >

## Exercice 5 — Scaling & mises à jour

9. Sur quels nœuds se trouvent vos 3 réplicas ? Collez la sortie de `kubectl get pods -l app=site -o wide`.
   ```

   ```
10. Avec l'image `nginx:9.99`, quel statut avait le pod ? Quelle commande vous a donné la cause ?
    >
11. Pendant la panne, le site restait-il accessible ? Pourquoi ?
    >

## Exercice 6 — Portainer & Mario

12. URL publique de votre Mario :
    >
13. Retrouvez le Deployment `mario` en ligne de commande. Portainer et `kubectl` parlent-ils au même composant ? Lequel ?
    >

## Bonus (facultatif)

14. Bonus réalisés et remarques :
    >

---

**Score obtenu avec `./scripts/check.sh` :** ... / ...
