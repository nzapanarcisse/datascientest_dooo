## LAB O (Architecture et Fonctionnement d'un Cluster Kubernetes)
![image](https://github.com/user-attachments/assets/d6cabbf4-5fe5-4852-b66d-3e0e98252866)

L'image ci-dessus illustre l'architecture d'un cluster Kubernetes. Dans cet environnement, il existe deux types d'interaction avec le cluster : soit vous êtes un développeur (qui déploie une application ou administre le cluster), soit vous êtes un utilisateur final (qui consomme une application déployée sur le cluster).

Il y a deux types de nœuds : les nœuds master et les nœuds worker. Les nœuds master abritent les composants nécessaires au fonctionnement de Kubernetes, tandis que les nœuds worker possèdent leurs propres composants et sont principalement chargés d’exécuter les instructions données par le nœud master.

Concentrons-nous sur le nœud master. Ce dernier, également appelé nœud gestionnaire, se compose généralement de quatre composants principaux :

1. **API Server (Kube API)**
   - **Description** : Point d'entrée principal pour toutes les requêtes au cluster Kubernetes. Il expose l'API Kubernetes et gère les opérations CRUD (Create, Read, Update, Delete) sur les ressources.
   - **Rôle** : Valide et transforme les demandes en objets de l'API, puis les stocke dans **etcd**, la base de données clé-valeur du cluster.

2. **Kube Controller Manager**
   - **Description** : Exécute les contrôleurs qui surveillent l'état des clusters et prennent des mesures pour corriger les déséquilibres.
   - **Rôle** : Par exemple, le contrôleur de réplication veille à ce qu'un nombre souhaité de pods soit en cours d'exécution.

3. **Kube Scheduler**
   - **Description** : Prend des décisions sur le nœud sur lequel exécuter les pods non planifiés.
   - **Rôle** : Évalue les ressources disponibles sur les nœuds, les affinités, les tolérances et d'autres contraintes pour effectuer un placement optimal.

4. **Kubelet**
   - **Description** : Agent qui s'exécute sur chaque nœud du cluster, assurant la gestion des pods.
   - **Rôle** : Vérifie l'état des conteneurs, les démarre, les arrête et communique l'état au Kube API Server.

### Exemple d'Interaction entre les Composants

**Lancement d’un Pod** :
- Lorsqu'une requête de création de pod est initiée, elle est envoyée au Kube API Server, qui valide et enregistre la demande dans etcd.

**Planification du Pod** :
- Le Kube Scheduler surveille les nouvelles demandes de pods, détecte le nouveau pod et évalue les nœuds disponibles pour choisir un nœud approprié en fonction des ressources et des contraintes.

**Création du Pod sur le Nœud** :
- Le Kubelet du nœud sélectionné reçoit l'information concernant le pod à créer via le Kube API Server.

**Gestion des Conteneurs** :
- Le Kubelet crée le conteneur sur le nœud, le démarre et surveille son état.

**Contrôle de l'État** :
- Pendant que le pod fonctionne, le Kube Controller Manager surveille l'état des ressources. Si un pod échoue, il prend des mesures pour le redémarrer ou le remplacer.


# Lab 1 (Exploration d'un cluster kubernetes)

Sans plus tarder, explorons les composants du cluster en commençant par le nœud principal.

## Commandes à exécuter

- `kubectl get nodes`
- `kubectl get pods -n kube-system`
- `kubectl get pods -A`
- `kubectl get rs -n kube-system`

```bash
systemctl status kubelet
```
```bash
kubectl delete pod etcd-vmi822295 -n kube-system
```
***Supprimons par exemple etcd (base de données du cluster). Nous allons nous rendre compte qu'un nouveau pod se crée. Cela est dû au fait que les quatre composants du nœud master que nous venons de voir sont des pods statiques.***

## Exploration des Pods statiques
Pour examiner les pods statiques, accédez au répertoire suivant :
```bash
cd /etc/kubernetes/manifests
ls
```

**Ensuite, consultez les fichiers de configuration des composants :**

```bash
cat kube-apiserver.yaml   # Vous pourrez voir les IP de vos services
cat kube-controller-manager.yaml
cat etcd.yaml
```
***Revenons à kubelet pour vérifier son état :***

```bash
systemctl status kubelet
```

***Pour voir la configuration de kubelet, exécutez :***

```bash
cat /etc/kubernetes/kubelet.conf
```
# Lab 2 (Problème de scalabilité ou de remonté en charge sur une application)
Ce fichier permet à kubelet de communiquer avec Kubernetes, notamment avec l'API server, et contient l'URL de connexion au cluster.
Pour voir la configuration de kubelet, consultez :
```bash
cat /var/lib/kubelet/config.yaml
```
![image](https://github.com/user-attachments/assets/0741f086-96ff-4510-8749-9f046188a5ff)

# Lab 2 (Problème de scalabilité ou de montée en charge sur une application ou un microservice)

Supposons que nous avons une application ou un microservice en production qui rencontre des problèmes de montée en charge (le microservice est très sollicité, notamment lors de jours de forte affluence, par exemple). Cela peut aussi être dû au fait que le nombre d'utilisateurs sollicitant le microservice a doublé, voire triplé, et que l'application ne répond plus ou met trop de temps à le faire.

Pour résoudre ce genre de souci avec Kubernetes, vous pouvez simplement augmenter le nombre de réplicas de votre application, et Kubernetes va équilibrer le trafic sur les différentes instances de votre application.

Appliquez les manifests du dossier `tp-1` et connectez-vous à l'application.
```bash
cd tp-1
kubectl apply -f .
```
![image](https://github.com/user-attachments/assets/f3d154d6-e2ee-40a7-b31e-d0ac674091a9)
![image](https://github.com/user-attachments/assets/d975fb2b-14bc-4225-8a62-be5680ddb13f)
![image](https://github.com/user-attachments/assets/898e94ec-c7d1-46be-a66e-6af22fd719c0)

Tout se passe comme si vous aviez mis un load balancer devant votre microservice, qui répartit la charge sur les différentes instances de celui-ci.
# Lab 3 (déploiement d'une application avec base de donnée sur kubernetes)
![image](https://github.com/user-attachments/assets/0a6bb3d2-b94d-4d23-a204-9bf4eebe2247)


Dans ce lab, nous allons déployer l'application Odoo dans un cluster Kubernetes. Odoo est un progiciel de gestion intégrée de type application à deux tiers. 

Pour la base de données, nous utiliserons un service de type ClusterIP, car la base de données est consommée par l'application frontale, et nous n'avons pas besoin d'y accéder depuis l'extérieur. Pour l'application Odoo elle-même (partie frontale), nous l'exposerons via un service de type NodePort, car nous souhaitons pouvoir consommer l'application depuis l'extérieur du cluster.

Pour des raisons spécifiques liées aux ressources CPU et mémoire, nous souhaitons que le microservice de la base de données ne soit déployé que sur le nœud `node01`, car ce nœud dispose des ressources nécessaires, ce que `node02` n'a pas. Pour résoudre ce genre de contrainte sur Kubernetes, vous pouvez utiliser la notion de taint ou le NodeSelector.
***taint***
![image](https://github.com/user-attachments/assets/0442570e-0863-451f-b771-4f6e7462fc71)
Vous pouvez donc taint vos nœuds et ajouter des tolerations à vos pods, afin qu'ils ne se déploient que sur des nœuds respectant cette taint. Dans le schéma ci-dessus, le pod (microservice) avec une taint rouge ne pourra se déployer que sur des nœuds avec une taint rouge. Il en va de même pour les pods avec une taint rose et une taint orange.

***affinity***
![image](https://github.com/user-attachments/assets/53fa40e7-a16c-45f2-af1c-a711884be878)

Vous pouvez ajouter un `nodeSelector` dans le manifest de votre microservice, indiquant sur quel nœud il doit se déployer. Cela doit être fait après avoir étiqueté vos nœuds Kubernetes. 

C'est ce que nous allons faire :

nous allons ajouter un label `app=db` au nœud `node01` et un autre label `front=front-end` au nœud `node02`.
```bash
kubectl label nodes vmi822295 app=db
kubectl label nodes vmi822295 front=front-end
```
pour voir les label sur un noeud
```bash
kubectl get node vmi822295 --show-labels
```
pour supprimer le label **app** sur le node **vmi822295**
```bash
kubectl label nodes vmi822295 app-
```

Ainsi, en utilisant ce code dans notre microservice, il ne pourra se déployer que sur le nœud étiqueté `app=db`.
**voir le les manifest(dossier odoo et postgres)**
![image](https://github.com/user-attachments/assets/f6f87240-54c2-410b-a3c7-5a1c74a55d95)

![image](https://github.com/user-attachments/assets/45c13ef5-39c6-4bda-93c9-33335e1280cd)


