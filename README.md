## Architecture et Fonctionnement d'un Cluster Kubernetes

Dans un cluster Kubernetes, il existe deux types d'interactions :

1. **Développeurs et Administrateurs** : Ils sont responsables du déploiement des applications et de l'administration du cluster.
2. **Utilisateurs** : Ils consomment les applications déployées sur le cluster.

### Types de Nœuds

Un cluster Kubernetes se compose de deux types de nœuds :

- **Nœuds Master** : Ils hébergent les composants essentiels au fonctionnement de Kubernetes.
- **Nœuds Worker** : Ils contiennent leurs propres composants et exécutent les instructions du nœud master.

### Composants du Nœud Master

Le nœud master, également appelé nœud manager, est constitué de quatre composants principaux :

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
