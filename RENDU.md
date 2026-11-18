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
   >kubectl create crée une ressource et échoue si elle existe déjà c'est impératif. kubectl apply applique un fichier YAML : il crée la ressource si elle n’existe pas, ou la met à jour si elle existe c'est déclaratif. On peut donc le relancer sans erreur.
8. À quoi sert le `selector` d'un Service ? Que se passe-t-il s'il ne correspond à aucun pod ?
   >Le selector permet de désigner les pods qui reçoivent le trafic, grâce à leurs labels comme ici app: site. S’il ne correspond à aucun pod, le Service n’a aucune cible : kubectl get endpoints site est vide et le site ne répond pas.

## Exercice 5 — Scaling & mises à jour

9. Sur quels nœuds se trouvent vos 3 réplicas ? Collez la sortie de `kubectl get pods -l app=site -o wide`.
   > Les réplicas se trouvent sur 3 nœuds différents k3d-lab-agent-0, k3d-lab-agent-1 et k3d-lab-server-0.
   ```@emmabraz ➜ /workspaces/atelier-kubernetes (main) $ kubectl get pods -l app=site -o wide
NAME                    READY   STATUS    RESTARTS   AGE     IP          NODE               NOMINATED NODE   READINESS GATES
site-7cd5578b5d-j97br   1/1     Running   0          8m22s   10.42.1.8   k3d-lab-agent-1    <none>           <none>
site-7cd5578b5d-kjwc4   1/1     Running   0          109s    10.42.2.5   k3d-lab-server-0   <none>           <none>
site-7cd5578b5d-vkcht   1/1     Running   0          109s    10.42.0.7   k3d-lab-agent-0    <none>           <none>
@emmabraz ➜ /workspaces/atelier-kubernetes (main) $ ```
10. Avec l'image `nginx:9.99`, quel statut avait le pod ? Quelle commande vous a donné la cause ?
    >Le nouveau pod était en ErrImagePull puis ImagePullBackOff (0/1). C’est la commande kubectl describe pod -l app=site qui m’a donné la cause : l’image nginx:9.99 reste en état Waiting, car ce tag n’existe pas et Kubernetes n’arrive pas à la télécharger.
11. Pendant la panne, le site restait-il accessible ? Pourquoi ?
    >Le site est resté accessible. Pendant le rolling update, Kubernetes ne supprime pas les anciens pods tant que le nouveau n’est pas prêt. Le pod nginx 9.99 n’est jamais devenu prêt donc le service a continué d’envoyer le trafic aux 3 anciens pods en nginx:1.28

## Exercice 6 — Portainer & Mario

12. URL publique de votre Mario :
    >https://refactored-spork-7vrgpwxq6prr3pgvq-8585.app.github.dev/
13. Retrouvez le Deployment `mario` en ligne de commande. Portainer et `kubectl` parlent-ils au même composant ? Lequel ?
    >Oui kubectl get deploy mario retrouve le Deployment créé par Portainer. Portainer et kubectl parlent au même composant qui est l’API Server de Kubernetes. Portainer n’est qu’une interface graphique qui envoie des requêtes à l’API Server, comme kubectl.

## Bonus (facultatif)

14. Bonus réalisés et remarques :
    > pour le A le site reste accessible après la suppression des pods. L’Ingress ne cible pas un pod précis et il envoie le trafic vers le Service site qui redirige vers les pods disponibles. Avec le port-forward, le tunnel mourait avec son pod.
    > pour le B, j’ai ajouté une readinessProbe HTTP sur / (port 80) ainsi que des requests et limits (CPU 50m/100m, mémoire 32Mi/64Mi) au conteneur nginx. Les 3 pods sont repartis en 1/1 Running. Avec le path /nope, le nouveau pod reste en 0/1 : nginx répond 404 (Readiness probe failed ... statuscode: 404), donc le pod n’est jamais prêt et ne reçoit aucun trafic. Le site reste accessible (200 OK) grâce aux anciens pods.
    > pour le C, après le drain de k3d-lab-agent-0, le pod site qui s’y trouvait a été évacué et recréé sur k3d-lab-agent-1 pareil pour les pods Portainer et système. Après le uncordon par contre aucun pod n’est revenu sur agent-0. Kubernetes ne déplace pas un pod déjà en cours d’exécution et uncordon rend seulement le nœud disponible pour les prochains pods.
    > pour le D
---

**Score obtenu avec `./scripts/check.sh` :**
>Score : 18 / 18  (l'exercice 3 n'est pas vérifiable automatiquement)
