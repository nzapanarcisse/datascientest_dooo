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

***déploiement**
```bash
cd postgres
kubectl apply -f .
kubectl get all -n ic-webapp
```
![image](https://github.com/user-attachments/assets/2acea960-6fba-4045-ba80-3f56a90158d1)

```bash
cd ../odoo
kubectl apply -f .
kubectl get all -n ic-webapp
```
![image](https://github.com/user-attachments/assets/5321cd2d-e867-4fba-882f-1abc36dfc481)

# Lab 4 : Backup du Cluster

Nous venons d'installer nos microservices sur le cluster. Supposons qu'un incident survienne et que nous souhaitions revenir en arrière. Bien que nos manifests soient sauvegardés sur un dépôt distant, il peut être nécessaire de restaurer l'état précédent du cluster. Il est donc essentiel de réaliser des sauvegardes, notamment de l'etcd (la base de données du cluster). Pour ce faire, nous allons utiliser la ligne de commande `etcdctl`. 

## Étapes d'installation

1. **Récupération de la version de votre etcd :**

   Exécutez la commande suivante pour obtenir la version d'etcd installée sur votre cluster :

```bash
   kubectl -n kube-system get pod/etcd-vmi822295 -o=jsonpath='{$.spec.containers[:1].image}'
```
   Cette commande vous donnera la version d'etcd installée sur votre cluster. Dans notre cas, nous avons etcd:3.5.4, comme le montre le schéma ci-dessous : ![image](https://github.com/user-attachments/assets/c569c7d6-d6c8-4af2-bf34-012eb62f04dd)
.

2. **Installation d'etcdctl : Voici les commandes à exécuter**

```bash
# Début de l'installation d'etcd
ETCD_VER=v3.5.4

# Choisissez l'URL
GOOGLE_URL=https://storage.googleapis.com/etcd
GITHUB_URL=https://github.com/etcd-io/etcd/releases/download
DOWNLOAD_URL=${GOOGLE_URL}

rm -f /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz
rm -rf /tmp/etcd-download-test && mkdir -p /tmp/etcd-download-test

curl -L ${DOWNLOAD_URL}/${ETCD_VER}/etcd-${ETCD_VER}-linux-amd64.tar.gz -o /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz
tar xzvf /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz -C /tmp/etcd-download-test --strip-components=1
rm -f /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz

/tmp/etcd-download-test/etcd --version
/tmp/etcd-download-test/etcdctl version

ln -s /tmp/etcd-download-test/etcdctl /usr/bin/etcdctl
etcdctl version
# Fin de l'installation d'etcd
```
![image](https://github.com/user-attachments/assets/a9696941-0e19-4efa-8794-29df17699b8a)

# Backup du Cluster

Pour voir la configuration et comprendre comment `kube-apiserver` communique avec les autres composants, vous pouvez exécuter la commande suivante :

```bash
cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep etcd
```

```bash
rm /tmp/backup.db

ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key \
snapshot save /tmp/backup.db
```
**vous pouvez voir la taile de votre base de données kubernetes**

```bash
ETCDCTL_API=3 etcdctl --write-out=table snapshot status /tmp/backup.db
```
![image](https://github.com/user-attachments/assets/ff1ae35f-824e-4c26-b4ff-d69626b03505)


Nous allons maintenant simuler une incident sur le cluster et effectuer une restauration. Pour ce faire, rendez-vous dans le dossier d'Odoo, puis supprimez les pods avec la commande suivante :

```bash
kubectl delete -f .
```
![image](https://github.com/user-attachments/assets/72111042-5c96-43a0-a82a-4e0b4f8576b1)

Pour commencer la restauration, nous allons d'abord supprimer les pods statiques afin d'éviter d'écrire dans la base de données pendant cette phase de restauration. Pour ce faire, exécutez les commandes suivantes :

```bash
mv /etc/kubernetes/manifests/kube-apiserver.yaml ./
mv /etc/kubernetes/manifests/etcd.yaml ./
rm -rf /var/lib/etcd
```

Nous pouvons donc restaurer notre cluster en utilisant la commande suivante :

```bash
ETCDCTL_API=3 etcdctl --data-dir="/var/lib/etcd" snapshot restore /tmp/backup.db
```
une fois la restoration nous pouvons remetre les fichiers static
```bash
mv kube-apiserver.yaml /etc/kubernetes/manifests/
mv etcd.yaml /etc/kubernetes/manifests/
```
![image](https://github.com/user-attachments/assets/d669038b-1675-4e0e-bd39-62803a2b9073)

vérifions la disponibilité de notre application
![image](https://github.com/user-attachments/assets/12ff1bd3-d946-42e6-a8b6-a32b9f643257)


***NB: En production, vous auriez besoin d'un outil plus complet comme Velero pour effectuer les sauvegardes de vos applications, d'un microservice en particulier ou d'un volume persistant spécifiquement. Vous pouvez également sauvegarder toutes les applications d'un namespace.**
![image](https://github.com/user-attachments/assets/7a5459cb-716e-4c10-a57a-b09b4e2899cc)
