# Ansible

Ce repository contient le code Ansible nécessaire pour déployer et administrer un cluster de 3 nœuds Docker Swarm avec ELK et Portainer.  

Pour les modifications et ajouts de code ansible il est intéressant de noter que les fonctionnalités ansible lint de vérification de logique et de syntaxe du code ne fonctionneront que depuis un environnement Linux ou WSL. Il est possible de modifier et contribuer au code depuis un environnement windows sans les fonctionnalités ansible lint. 
## Liste des playbooks et leur description

### Playbooks de gestion des nœuds

- **shutdown.yml** : Envoie le signal d’extinction aux nœuds.
- **reboot.yml** : Envoie le signal de redémarrage aux nœuds.

### Configuration système

- **ntp-setup-sync.yml** : Configure les serveurs NTP de l’aéroport ainsi que le fuseau horaire. Rafraîchit ensuite l’heure.

- **adduser-ssh.yml**  
  Ce playbook créé et ajoute un utilisateur a un groupe, ici le groupe admin avec moindre privilège
  Le nouvel utilisateur pourra se connecter uniquement en ssh
  Il est nécessaire de fournir un nouveau login, un mot de passe et une clef publique ssh  
- **adduser-admin-ssh**  
  Ce playbook ajoute l'utilisateur admin ansible (admindsi) au groupe sudo si ce n'est pas déja fait, puis configure le ssh pour pouvoir utiliser ansible    
  L'utilisateur pourra se connecter uniquement en ssh  
  Il est nécessaire de fournir une clef publique ssh  

- **groups-hardening.yml**  
  Ce playbook créée un groupe admin avec droits limités comparé au groupe sudo
  Les droits du groupe admin sont ceux qui vont être affecté aux administrateurs de la machine
  Certains privilèges puissant comme la gestion des droits et des utilisateurs y sont restreints
  De plus, le changement d'utilisateur est bloqué

- **elk-stack-setup.yml**    
  Ce playbook installe le repo github docker-elk et exécute le script de mise en place du cluster ELK
  /!\ Le .env et les secrets doivent être renseignés et/ou modifiés avant execution
  Comme prérequis les playbooks  `Docker-engine-setup.yml` et `Docker-swarm-init.yml` doivent avoir été exécutés
  
  Sont inclus:
  - 3 conteneurs elasticsearch
  - 1 conteneur kibana
  - 1 conteneur logstash (facultatif)
  - 1 conteneur fleet-server
  - 3 conteneurs metricbeat
  - 3 conteneurs filebeat

- **elk-stack-rm.yml**
  Ce playbook installe le repo github docker-elk et exécute le script de deletion du cluster ELK
  

### Configuration SSH
- **ssh-hardening.yml** :  
  Applique une couche de hardening SSH au fichier sshd_config. 
  
### Docker

- **docker-engine-setup.yml** :  
  Prépare les nœuds à être conteneurisés en installant Docker de manière officielle et en ajoutant l’utilisateur Ansible au groupe Docker.

- **docker-swarm-init.yml** :  
  Initialise et configure le mode Docker Swarm (cluster).

- **docker-stack-portainer.yml** :  
  Déploie un stack Docker pour Portainer en haute disponibilité.

### Stockage

- **microceph-deploy.yml** :  
  Installe et configure le stockage distribué MicroCeph en mode stockage objet.
  S'assurer d'avoir un disque libre par noeud

## Playbooks informatifs (test/debug)

- **docker-info.yml** :  
  Affiche les conteneurs en cours ainsi que les ressources utilisées par Docker.

- **docker-container-test.yml** :  
  Teste et valide le fonctionnement de Docker.

## Playbook complet

- **portainer-ha-full.yml** :  
  Ce playbook inclut les rôles suivants :
  - `Ssh-setup.yml`
  - `Ntp-setup-sync.yml`
  - `Docker-engine-setup.yml`
  - `Microceph-deploy.yml`
  - `Docker-swarm-init.yml`
  - `Docker-stack-portainer.yml`

## Comment installer et utiliser Ansible

### Installation sur environnement GNU/Linux Debian
*Téléchargement et Installation d'Ansible et ses dépendances:*  
$ sudo apt update && sudo apt upgrade -y  
$ sudo apt install git python3 ansible ansible-lint sshpass  
*Cloner le repo:*  
$ git clone ssh://git@ssh.github.com:443/NTE-Airport-DSI/ansible.git  
$ cd ansible/  
*Adapter vos IP et hosntames dans l'inventaire:*  
$ nano inventory.ini  
*Tester la connectivité*  
$ ansible all -m ping  
*Exécuter des playbooks!*  
*Avec mot de passe*  
$ ansible-playbook playbooks/playbook-a-executer -K  
*Avec ssh configuré*  
$ ansible-playbook playbooks/playbook-a-executer  

