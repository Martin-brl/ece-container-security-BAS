# ECE 

**Major: Cybersecurity International**  

---

**Course**  
*Sécurité des Containers*

**Lab Report**  
Projet Final : Minecraft Secure Cluster

**Instructor**  
Etienne LOUTSCH

**Group Members**  
- ARDILLON Bastian
- SAVEAUX Octave
- BRULEY Martin


**Date**  
Mars 2026

---
<div style="page-break-after: always;"></div>

<br><br><br><br><br><br><br><br><br><br>

## Sommaire

1. [Étapes du projet](#étapes-du-projet)
2. [Capture d'écran du serveur Minecraft accessible](#capture-décran-du-serveur-minecraft-accessible)
3. [Documentation des choix de sécurité](#documentation-des-choix-de-sécurité)
4. [Exemple de rejet de pipeline pour CVE critique](#exemple-de-rejet-de-pipeline-pour-cve-critique)
5. [Bonus](#bonus)

---
<div style="page-break-after: always;"></div>

## Étapes du projet

Nous avons utilisé une machine Ubuntu 22.04 LTS avec Docker, kubectl, Kind et Helm installés. Le cluster Kind a été configuré avec un fichier `cluster.yml` incluant un port mapping pour exposer le serveur Minecraft :

Le port mapping redirige le port 25565 (port par défaut de Minecraft) de la machine hôte vers le port 30001 du cluster

Après la création du cluster, nous avons vérifié que le nœud est bien en état Ready :

```bash
kubectl get nodes
```

<div align="center">
  <img src="screenshots/octave/Q1 affiche le noeud en ready .png" alt="Noeud du cluster en Ready" width="900">
  <p><em>Figure 1 : nœud du cluster Kind en état Ready</em></p>
</div>


---

Pour pouvoir déployer depuis GitLab, nous avons installé un GitLab Runner directement dans notre cluster. Nous avons directement suivi les instructions données dans le sujet du projet.

Le Runner apparaît bien comme connecté dans l'interface GitLab :

<div align="center">
  <img src="screenshots/octave/Q3 page gitlab des runners qui montre qu'il est bien co .png" alt="GitLab Runner connecté" width="900">
  <p><em>Figure 2 : GitLab Runner connecté dans l'interface GitLab</em></p>
</div>

Puis nous avons vérifié que le pod du Runner est en état Running :

<div align="center">
  <img src="screenshots/octave/Q2 pods en running.png" alt="Pods en running" width="900">
  <p><em>Figure 3 : pods du GitLab Runner en état Running</em></p>
</div>

---

Nous avons par la suite créé un namespace `minecraft` , puis configuré un ServiceAccount, un Role et un RoleBinding pour que la CI puisse déployer dans ce namespace :

Nous avons ensuite généré un token pour le ServiceAccount et l'avons stocké comme variable CI/CD dans GitLab :


<div align="center">
  <img src="screenshots/octave/Q4 variable ajoutée a gitlab CI_DEPLOY_TOKEN_K8S.png" alt="Variable CI/CD GitLab" width="900">
  <p><em>Figure 4 : variable CI_DEPLOY_TOKEN_K8S ajoutée dans GitLab (Protected & Masked)</em></p>
</div>

---

Le Dockerfile est basé sur l'image officielle `itzg/minecraft-server` :

Le fichier `minecraft-deploy.yml` contient un Deployment et un Service. Le Deployment configure le serveur avec les variables d'environnement nécessaires. Le Service de type NodePort expose le port 25565 sur le nodePort 30001 :

Nous avons vérifié que le pod Minecraft démarre correctement :

<div align="center">
  <img src="screenshots/octave/Q5 montrer le pod en running .png" alt="Pod Minecraft en Running" width="700">
  <p><em>Figure 5 : pod Minecraft en état Running</em></p>
</div>

Les logs du serveur confirment le démarrage avec le message "Done" :

<div align="center">
  <img src="screenshots/octave/Q6 message Done(XXs) pour dire que ca a bien marche.png" alt="Serveur Minecraft démarré" width="900">
  <p><em>Figure 6 : message "Done" confirmant le démarrage du serveur Minecraft</em></p>
</div>

---

Le pipeline `.gitlab-ci.yml` est composé de 5 étapes :
- `lint` : analyse du code 
- `build` : construit l'image Docker
- `scan` : détecte les CVE
- `verify` : analyse les CVE HIGH/MEDIUM
- `deploy` : déploie le serveur dans Kubernetes

La pipeline s'exécute avec succès, sauf le stage `scan` qui est en warning car des CVE critiques ont été détectées :

<div align="center">
  <img src="screenshots/octave/Q7 page gitlab de la pipeline qui est en vert sauf scan an warning.png" alt="Pipeline GitLab CI/CD" width="900">
  <p><em>Figure 7 : pipeline CI/CD — tous les stages passent sauf le scan (warning attendu)</em></p>
</div>

Après le déploiement par la CI/CD, le pod Minecraft est toujours en état Running :

<div align="center">
  <img src="screenshots/octave/Q9 pods toujours en running apres le deployed ci cd.png" alt="Pod Running après CI/CD" width="600">
  <p><em>Figure 8 : pod Minecraft en Running après déploiement via la CI/CD</em></p>
</div>

---
<div style="page-break-after: always;"></div>





## Capture d'écran du serveur Minecraft accessible

Le serveur Minecraft est accessible en se connectant à l'adresse `127.0.0.1:25565`.

Ajout du serveur dans Minecraft :

<div align="center">
  <img src="screenshots/octave/Q10 jeu mc edit de l'adresse ip.png" alt="Ajout du serveur Minecraft" width="300">
  <p><em>Figure 9 : ajout du serveur avec l'adresse 127.0.0.1:25565</em></p>
</div>

Le serveur apparaît dans la liste multijoueur avecle message "ECE 2026 - Server!" :

<div align="center">
  <img src="screenshots/octave/Q11 jeu mc liste des servers multi avec le notre.png" alt="Liste des serveurs Minecraft" width="400">
  <p><em>Figure 10 : serveur visible dans la liste multijoueur</em></p>
</div>


Connexion réussie au serveur :

<div align="center">
  <img src="screenshots/octave/Q12 jeu mc dans la game ca marche.png" alt="Connecté au serveur Minecraft" width="700">
  <p><em>Figure 11 : connexion réussie au serveur Minecraft — le joueur est dans le monde</em></p>
</div>

---
<div style="page-break-after: always;"></div>









## Documentation des choix de sécurité


### Choix de sécurité implémentés

**1. RBAC (Role-Based Access Control)**
Le ServiceAccount `ci-deployer` dispose uniquement des permissions nécessaires au déploiement dans le namespace `minecraft`. Il ne peut pas accéder aux autres namespaces ni effectuer d'opérations en dehors de son périmètre. Ce principe du **moindre privilège** limite l'impact en cas de compromission du token CI.

**2. Kaniko pour le build d'images**
Nous avons choisi Kaniko pour construire les images. Kaniko ne nécessite pas de privilèges root, il peut donc s'exécuter entièrement en espace utilisateur dans le cluster Kubernetes.

**3. Trivy pour le scan de vulnérabilités**
Trivy analyse automatiquement l'image construite à chaque pipeline. Le stage `scan` est configuré avec `--exit-code 1 --severity CRITICAL` pour bloquer le déploiement si des CVE critiques sont détectées. Le stage `verify` analyse les vulnérabilités HIGH/MEDIUM à titre informatif.

**4. Variables protégées et masquées**
Le token `CI_DEPLOY_TOKEN_K8S` est stocké en tant que variable CI/CD **Protected** (utilisable uniquement sur les branches protégées) et **Masked** (non visible dans les logs de la pipeline).

**5. Network Policies (Bonus)**
Une NetworkPolicy restreint le trafic entrant vers le pod Minecraft au seul port 25565. Tout autre trafic réseau entrant est bloqué, réduisant la surface d'attaque.

**6. Falco Monitoring (Bonus)**
Falco surveille en temps réel les activités suspectes sur le cluster. Il détecte notamment l'ouverture de shells dans les containers et la lecture de fichiers sensibles, ce qui permet de détecter des tentatives de triche ou d'intrusion sur le serveur Minecraft.

---
<div style="page-break-after: always;"></div>

## Exemple de rejet de pipeline pour CVE critique

Lorsque des CVE critiques sont détectées dans l'image, le job échoue. Grâce au paramètre `allow_failure: true`, la pipeline continue mais le stage apparaît en **warning**.

Trivy a détecté les CVE critiques suivantes dans l'image `itzg/minecraft-server` :

| Bibliothèque | CVE | Sévérité | Version installée | Version corrigée |
|--------------|-----|----------|-------------------|------------------|
| scala-library | CVE-2022-36944 | CRITICAL | 2.13.1 | 2.13.9 |
| easy-add (Go stdlib) | CVE-2025-68121 | CRITICAL | v1.24.4 | 1.24.13, 1.25.7 |
| gosu (Go stdlib) | CVE-2025-68121 | CRITICAL | v1.24.6 | 1.24.13, 1.25.7 |

- **CVE-2022-36944** : vulnérabilité dans la bibliothèque Scala, permettant l'exécution de code.
- **CVE-2025-68121** : problème de reprise de session inattendue dans `crypto/tls` de Go, pouvant compromettre la sécurité TLS.

<div align="center">
  <img src="screenshots/octave/Q8 tableaux du scan qui montre les CVE.png" alt="CVE critiques détectées par Trivy" width="900">
  <p><em>Figure 12 : CVE critiques détectées par Trivy lors du scan de l'image</em></p>
</div>

Ces vulnérabilités proviennent de l'image de base `itzg/minecraft-server` et de ses dépendances. Il faut donc mettre a jour les dépendances concernées aux versions recommandées.

---
<div style="page-break-after: always;"></div>











## Bonus

### 5.1 Network Policies

Nous avons créé une NetworkPolicy pour restreindre l'accès réseau au pod Minecraft. Seul le trafic entrant sur le port 25565 (TCP) est autorisé :

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: minecraft-network-policy
  namespace: minecraft
spec:
  podSelector:
    matchLabels:
      app: minecraft
  policyTypes:
    - Ingress
  ingress:
    - ports:
        - protocol: TCP
          port: 25565
```

<div align="center">
  <img src="screenshots/octave/Q13 le fichier network-policy.png" alt="Fichier network-policy.yml" width="500">
  <p><em>Figure 13 : contenu du fichier network-policy.yml</em></p>
</div>

La policy a été appliquée avec succès :

<div align="center">
  <img src="screenshots/octave/Q14 montrer que le minecraft-network-policy a bien lancé.png" alt="NetworkPolicy appliquée" width="700">
  <p><em>Figure 14 : NetworkPolicy minecraft-network-policy créée</em></p>
</div>

Nous avons vérifié que le serveur Minecraft reste accessible depuis le jeu après l'application de la policy :

<div align="center">
  <img src="screenshots/octave/Q15 montrer jeu mc apres le apply des network policy que ca marche bien .png" alt="Minecraft accessible après NetworkPolicy" width="700">
  <p><em>Figure 15 : serveur Minecraft toujours accessible sur le port 25565</em></p>
</div>

Pour prouver que la policy fonctionne, nous avons lancé un pod de test dans le namespace `minecraft` et tenté d'accéder au service sur un port non autorisé. La connexion est bien bloquée:

<div align="center">
  <img src="screenshots/octave/Q16 montrer que venant d'un autre port ca marche pas.png" alt="Port 80 bloqué par la NetworkPolicy" width="500">
  <p><em>Figure 16 : port 25565 ouvert vs port 80 bloqué</em></p>
</div>

---

### 5.2 Falco Monitoring

Nous avons installé Falco dans le cluster pour surveiller les activités suspectes sur les containers, notamment le pod Minecraft

Nous avons vérifié que les pods Falco sont en état Running :

<div align="center">
  <img src="screenshots/octave/Q17 verif que flaco tourne.png" alt="Pods Falco en Running" width="600">
  <p><em>Figure 17 : pods Falco en état Running</em></p>
</div>



Pour simuler une tentative de triche nous avons exécuté un shell dans le pod et effectué les commandes suivante :

```bash
whoami
cat /etc/shadow
exit
```

<div align="center">
  <img src="screenshots/octave/Q18 execution de commande dans le container minecraft.png" alt="Commandes suspectes dans le container" width="500">
  <p><em>Figure 18 : exécution de commandes suspectes dans le container Minecraft</em></p>
</div>

Falco a immédiatement détecté ces activités et généré des alertes :

<div align="center">
  <img src="screenshots/octave/Q19 log falco qui montre la detection de commande bash dans container.png" alt="Alertes Falco" width="900">
  <p><em>Figure 19 : alertes Falco</em></p>
</div>

<div align="center">
  <img src="screenshots/octave/Q20 log falco qui montre la meme chose mais en plus lisible.png" alt="Alertes Falco filtrées" width="900">
  <p><em>Figure 20 : alertes Falco filtrées"</em></p>
</div>

Falco a détecté deux types d'alertes :
- **Terminal shell in container** : détection de l'ouverture d'un shell bash dans le container Minecraft, correspondant à une potentielle tentative d'accès non autorisé.
- **Read sensitive file untrusted** : détection de la lecture du fichier `/etc/shadow`, indiquant une tentative d'accès à des informations sensibles du système.


