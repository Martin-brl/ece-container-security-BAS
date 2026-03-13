# ECE 

**Major: Cybersecurity International**  

---

**Course**  
*Sécurité des Containers*

**Lab Report**  
Session 3

**Instructor**  
Etienne LOUTSCH

**Group Members**  
- ARDILLON Bastian
- SAVEAUX Octave
- BRULEY Martin


**Date**  
February 25, 2026

---
<div style="page-break-after: always;"></div>


## Activités Pratiques


### Déployer un Cluster Kubernetes avec Kind 


Nous avons installé Kind et kubectl, puis créé un cluster Kubernetes avec 2 nœuds control-plane et 2 nœuds worker en utilisant un fichier de configuration YAML

```bash
kind create cluster --config cluster-config.yaml
```

La création du cluster a été effectuée avec succès : 

<div align="center">
  <img src="screenshots/octave/Q1-1 creation du cluster.png" alt="Création du cluster Kind" width="900">
  <p><em>Figure 1: création du cluster Kubernetes avec Kind</em></p>
</div>


Pour vérifier que le cluster est correctement déployé, nous avons exécuté :

```bash
kubectl get nodes
```

Le résultat affiche bien les quatre nœuds :

<div align="center">
  <img src="screenshots/octave/Q1-2 verif des 4 cluster avec 2 worker 2 master.png" alt="Vérification des 4 nœuds du cluster" width="900">
  <p><em>Figure 2: vérification des 4 nœuds du cluster (2 master, 2 workers)</em></p>
</div>


Pour afficher la liste des namespaces du cluster nous avons executé la commande suivante :

```bash
kubectl get namespaces
```

<div align="center">
  <img src="screenshots/octave/Q1-3 afficher la liste des namespace.png" alt="Liste des namespaces" width="600">
  <p><em>Figure 3: liste des namespaces du cluster</em></p>
</div>


Pour connaître la version de Kubernetes déployée, voici la commande utilisée:

```bash
kubectl version 
```



<div align="center">
  <img src="screenshots/octave/Q1-4 version de kubernetes.png" alt="Version de Kubernetes" width="900">
  <p><em>Figure 4: version de Kubernetes déployée</em></p>
</div>

La version de Kubernetes déployée est la **v1.27.3** (comme on peut le voir sur les nœuds et dans la sortie de `kubectl version`).


---
<div style="page-break-after: always;"></div>

### Expérimentation des RBAC

Nous avons créé un namespace dédié à l’expérimentation RBAC se nommant **test-rbac**:

```bash
kubectl create ns test-rbac
```

<div align="center">
  <img src="screenshots/octave/Q2-2 creation d'un namespace.png" alt="Création du namespace test-rbac" width="600">
  <p><em>Figure 5: création du namespace test-rbac</em></p>
</div>

On vérifie ensuite avec `kubectl get namespaces` que le namespace `test-rbac` apparaît bien dans la liste

<div align="center">
  <img src="screenshots/octave/Q2-2-2 verif que le ns soit bien creer .png" alt="Vérification du namespace test-rbac" width="600">
  <p><em>Figure 5b: vérification de la création du namespace test-rbac</em></p>
</div>

Nous avons déployé un pod nginx dans le namespace **test-rbac** avec un fichier YAML. Voici le fichier YAML utilisé :

<div align="center">
  <img src="screenshots/octave/Q2-1 fichier mon-pod yaml.png" alt="Fichier mon-pod.yaml" width="800">
  <p><em>Figure 6: contenu du fichier mon-pod.yaml</em></p>
</div>


Nous avons déployé ce pod avec la commande suivante :

```bash
kubectl apply -f mon-pod.yaml
```

<div align="center">
  <img src="screenshots/octave/Q2-3 deployd de ce pod dans le namespace.png" alt="Déploiement du pod" width="600">
  <p><em>Figure 7: déploiement du pod nginx dans le namespace test-rbac</em></p>
</div>



Nous avons par la suite afficher les logs de ce pod :


```bash
kubectl logs nginx -n test-rbac
```

<div align="center">
  <img src="screenshots/octave/Q2-4 afficher les logs de ce pod.png" alt="Logs du pod nginx" width="900">
  <p><em>Figure 8: logs du pod nginx</em></p>
</div>



Nous avons créé un Role `pod-reader` qui autorise uniquement les actions `get` et `list` sur les pods dans le namespace `test-rbac`, voici le fichier YAML utilisé :

<div align="center">
  <img src="screenshots/octave/Q2-6 role pod rader yaml.png" alt="Fichier role-pod-reader.yaml" width="1000">
  <p><em>Figure 9: contenu du fichier role-pod-reader.yaml</em></p>
</div>

Pour appliquer ce Role, nous avons executé la commande suivante :

```bash
kubectl apply -f role-pod-reader.yaml
```

<div align="center">
  <img src="screenshots/octave/Q2-5  Créer un role pour lire les pods dans le namespace test-rbac.png" alt="Création du Role" width="950">
  <p><em>Figure 10: création du Role pod-reader</em></p>
</div>



Pour afficher ce nouvelle role créé nous avons executé la commande suivante :

```bash
kubectl get role pod-reader -n test-rbac
```

<div align="center">
  <img src="screenshots/octave/Q2-7 Comment afficher ce role.png" alt="Affichage du Role" width="800">
  <p><em>Figure 11: affichage du Role pod-reader</em></p>
</div>

Nous pouvons bien voir que le Role a été créé avec succès.






Nous avons appliqué un RoleBinding donné dans l'énoncé pour associer le Role `pod-reader` à l’utilisateur `titi`. Voici le fichier YAML utilisé :

<div align="center">
  <img src="screenshots/octave/Q2-8 ier ce Role avec un utilisateur fictif titi, appliquer ce fichier.png" alt="Application du RoleBinding" width="900">
  <p><em>Figure 12: liaison du Role avec l'utilisateur titi</em></p>
</div>


Pour créer l’utilisateur fictif `titi`, nous avons récupéré la CA du cluster depuis le conteneur Kind :

```bash
docker cp kind-control-plane:/etc/kubernetes/pki/ca.crt .
docker cp kind-control-plane:/etc/kubernetes/pki/ca.key .
```

<div align="center">
  <img src="screenshots/octave/Q2-9 Récupérer l'autorité de certification (CA) du cluster.png" alt="Récupération de la CA" width="850">
  <p><em>Figure 13: récupération de l'autorité de certification du cluster</em></p>
</div>

Ensuite, nous avons généré les certificats pour l’utilisateur titi :

```bash
openssl genrsa -out titi.key 2048
openssl req -new -key titi.key -out titi.csr -subj "/CN=titi"
openssl x509 -req -in titi.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out titi.crt -days 365
```

<div align="center">
  <img src="screenshots/octave/Q2-10 Générer les certificats pour Titi.png" alt="Génération des certificats pour titi" width="1000">
  <p><em>Figure 14: génération des certificats pour l'utilisateur titi</em></p>
</div>

Puis nous avons ajouté l’utilisateur titi au contexte kubectl et créé un contexte dédié :

```bash
kubectl config set-credentials titi --client-certificate=titi.crt --client-key=titi.key
kubectl config set-context titi-context --cluster=kind-kind --namespace=test-rbac --user=titi
```

<div align="center">
  <img src="screenshots/octave/Q2-11 ajout de titi dans le kube context.png" alt="Ajout de titi au contexte" width="600">
  <p><em>Figure 15: ajout de l'utilisateur titi au contexte kubectl</em></p>
</div>

<div align="center">
  <img src="screenshots/octave/Q2-13 creation du contexte pour titi dans ns test-rbac.png" alt="Création du contexte titi-context" width="600">
  <p><em>Figure 16: création du contexte titi-context dans le namespace test-rbac</em></p>
</div>

Nous avons vérifié que l'utilisateur était bien pris en compte dans la liste des credentials disponibles.

<div align="center">
  <img src="screenshots/octave/Q2-12 verification presence titi creds.png" alt="Vérification presence credentials titi" width="600">
  <p><em>Figure 17: vérification de la présence titi</em></p>
</div>



Pour utiliser le contexte de l’utilisateur titi nous avons executé la commande suivante :

```bash
kubectl config use-context titi-context
```

<div align="center">
  <img src="screenshots/octave/Q2-14 basculer vers context.png" alt="Basculer vers le contexte titi" width="600">
  <p><em>Figure 18: basculement vers le contexte titi-context</em></p>
</div>


Avec le contexte `titi-context`, nous pouvons lister les pods dans le namespace `test-rbac` car le Role `pod-reader` autorise les verbes `get` et `list` sur les pods :

```bash
kubectl get pods -n test-rbac
```

<div align="center">
  <img src="screenshots/octave/Q2-15 on peut acceder a la liste des pods.png" alt="Liste des pods en tant que titi" width="400">
  <p><em>Figure 19: accès à la liste des pods par l'utilisateur titi</em></p>
</div>

Le résultat confirme que l’utilisateur titi peut accéder à la liste des pods (le pod nginx apparaît en statut Running).


En revanche, l’utilisateur titi ne dispose pas de la possibilité de créer un pod. Lorsque nous essayons de créer un pod :

```bash
kubectl run test --image=nginx -n test-rbac
```



<div align="center">
  <img src="screenshots/octave/Q2-16 access denied to run pods.png" alt="Erreur Forbidden lors de la création d'un pod" width="950">
  <p><em>Figure 20: accès refusé — titi ne peut pas créer de pods</em></p>
</div>

Nous obtenons une erreur `Forbidden` : *User "titi" cannot create resource "pods" in API group "" in the namespace "test-rbac"*. Cela démontre que le RBAC restreint correctement les permissions : titi peut lire les pods mais pas en créer.

---
<div style="page-break-after: always;"></div>

### Scanner un Cluster Kubernetes avec Kube-Bench



Nous avons déployé Kube-Bench en tant que Job Kubernetes pour scanner la sécurité du cluster. Le Job a été appliqué avec :

```bash
kubectl apply -f job.yml
```

Nous avons vérifié que le Job s’exécute correctement.

<div align="center">
  <img src="screenshots/octave/Q2-17 appliquer le job yaml et verif qui tourne bien .png" alt="Application du Job Kube-Bench" width="500">
  <p><em>Figure 21: application du Job Kube-Bench et vérification de l'exécution</em></p>
</div>


Le scan Kube-Bench analyse le cluster selon les recommandations CIS (Center for Internet Security) et produit un rapport listant les contrôles passés, avertis et échoués. Les résultats permettent d’identifier les configurations à durcir 


<div align="center">
  <img src="screenshots/octave/Q2-18 resultat .png" alt="Résultats du scan Kube-Bench" width="250">
  <p><em>Figure 22: résultats du scan Kube-Bench</em></p>
</div>

Kube-Bench a détecté 11 PASS, 6 FAIL et 42 WARN. Les FAIL sont à corriger, beaucoup de WARN indiquent des points à améliorer. Notre cluster est donc pas optimisé

---
<div style="page-break-after: always;"></div>

### Détection et alerte d'intrusions avec Falco

Nous avons ajouté le dépôt Helm Falco et installé la chart dans un namespace dédié :

<div align="center">
  <img src="screenshots/octave/Q3-1 ajout du repo helm creation ns et install chart falco .png" alt="Installation de Falco" width="900">
  <p><em>Figure 23: ajout du dépôt Helm et installation de Falco</em></p>
</div>


Après quelques minutes, nous avons vérifié que les pods du namespace `falco` sont en état Running :

```bash
kubectl get pods -n falco
```

<div align="center">
  <img src="screenshots/octave/Q3-2 verif que tout les pods sont en running.png" alt="Pods Falco en Running" width="600">
  <p><em>Figure 24: vérification des pods Falco en Running</em></p>
</div>

Tous les pods sont en état *Running*, l'installation s'est parfaitement déroulée et tout fonctionne comme attendu.



Pour afficher l’interface web de Falco, nous avons effectué un port-forward dans notre navigateur :

```bash
kubectl port-forward svc/falco-falcosidekick-ui 2802:2802 --namespace falco
```


<div align="center">
  <img src="screenshots/octave/Q3-3 commande afficher falco dans navigateur.png" alt="Port-forward vers l'UI Falco" width="600">
  <p><em>Figure 25: commande port-forward pour l'interface Falco</em></p>
</div>


Puis ouvert l’URL http://127.0.0.1:2802 dans le navigateur.

<div align="center">
  <img src="screenshots/octave/Q3-4 site dans navigateur.png" alt="Interface Falco dans le navigateur" width="900">
  <p><em>Figure 26: interface Falco dans le navigateur</em></p>
</div>







Nous avons déployé un pod basé sur Alpine avec un fichier YAML donné dans l'énoncé :

<div align="center">
  <img src="screenshots/octave/Q3-5 creation et apply du yml.png" alt="Création et déploiement du pod front" width="600">
  <p><em>Figure 26: création et déploiement du pod front</em></p>
</div>

Puis nous avons exécuté un shell dans le pod et lu le fichier `/etc/shadow`. Falco a détecté cette action et généré une alerte : 


<div align="center">
  <img src="screenshots/octave/Q3-6 warning dans falco .png" alt="Alerte Falco - Read sensitive file" width="900">
  <p><em>Figure 27: alerte Falco — lecture du fichier /etc/shadow (Priorité: Warning, Règle: Read sensitive file untrusted)</em></p>
</div>


Toujours dans le shell du pod `front`, nous avons effectué une requête vers l’API Kubernetes :

Falco peut également détecter ce type d’activité. L’alerte générée a pour priorité **Warning** et concerne la règle liée aux connexions sortantes vers l’API Kubernetes.

<div align="center">
  <img src="screenshots/octave/Q3-6 warning dans falco pour générer une requête sur l'API Kubernetes.png" alt="Alerte Falco - Requête API Kubernetes" width="900">
  <p><em>Figure 28: alerte Falco — requête vers l'API Kubernetes depuis le pod</em></p>
</div>
