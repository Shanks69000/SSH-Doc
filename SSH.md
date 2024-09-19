# Protocole SSH

## 1. Introduction à SSH

Le **Secure Shell (SSH)** est un protocole de communication réseau qui permet de se connecter de manière sécurisée à une machine distante. SSH utilise un système de clés cryptographiques pour authentifier les utilisateurs, garantir la confidentialité des échanges, et permettre des fonctionnalités comme le transfert de fichiers ou la gestion à distance de serveurs.

## 2. Fonctionnalités principales de SSH

- **Connexion sécurisée** : SSH chiffre les communications, protégeant ainsi les données en transit.
- **Transfert de fichiers** : Utilisation de SCP ou SFTP pour transférer des fichiers entre machines.
- **Authentification par clés** : Remplacement de l'authentification par mot de passe avec des clés publiques et privées.

---

## 3. Mise en place de l'environnement

### 3.1. Création des deux machines virtuelles sous Debian 12

On va créer deux machines virtuelles avec VMware Workstation en utilisant Debian 12 comme OS. Les adresses IP sont configurées comme suit :

- **VM1** : `172.16.10.1/24`
- **VM2** : `172.16.10.2/24`

---

## 4. Installation et configuration de SSH

### 4.1. Installation du serveur et du client SSH

Sur les deux machines, il est nécessaire d'installer le paquet SSH. Voici comment faire :


``sudo apt update``
``sudo apt install openssh-server -y``

Vérification que le service SSH fonctionne :

``sudo systemctl status ssh``

Pour démarrer SSH s'il n'est pas actif :

``sudo systemctl start ssh``
``sudo systemctl enable ssh``

### 4.2. Configuration du serveur SSH

Les fichiers de configuration se trouvent dans **/etc/ssh/sshd_config**. Si on souhaite modifier des paramètres (comme le port ou les options d'authentification), voici comment faire :

``sudo nano /etc/ssh/sshd_config``

Modifie des paramètres comme :

- Port : par défaut 22
- PermitRootLogin : désactive l'accès root (no)
- PasswordAuthentication : désactive l'authentification par mot de passe pour renforcer la sécurité (no si tu utilises des clés)

Une fois les modifications effectuées, on redémarre SSH pour appliquer les changements :

``sudo systemctl restart ssh``

---

## 5. Création et gestion des clés SSH

### 5.1. Génération d'une paire de clés SSH

Sur VM1, on génère une paire de clés SSH pour s'authentifier sur VM2 :

``ssh-keygen -t rsa -b 4096``

On peut choisir un nom pour la clé ou laisser la valeur par défaut. On laisse la passphrase vide pour un usage simplifié pendant les tests.

La clé publique est stockée dans **~/.ssh/id_rsa.pub** et la clé privée dans **~/.ssh/id_rsa**.

### 5.2. Copie de la clé publique sur la machine distante

Pour se connecter à VM2 sans mot de passe, il faut copier notre clé publique sur cette machine :

``ssh-copy-id user@172.16.10.2``

si le port par defaut est changé :

``ssh-copy-id -p (port modifié) user@172.16.10.2``

On sera invité à entrer le mot de passe de la machine distante une dernière fois. SSH se chargera de copier notre clé publique dans le fichier **~/.ssh/authorized_keys** sur VM2.

---

## 6. Connexion SSH sans mot de passe

Une fois la clé publique configurée, on peut se connecter de VM1 à VM2 sans avoir à entrer de mot de passe :

``ssh user@172.16.10.2``

si le port par defaut est changé :

``ssh -p  (port modifié) user@172.16.10.2``

Si tout est configuré correctement, on devrait être directement connecté à la machine distante.

---

## 7. Transfert de fichiers avec SCP

Pour transférer des fichiers entre les machines via SSH, on utilise SCP. Par exemple, pour envoyer un fichier de VM1 à VM2 :

``scp /chemin/vers/fichier.txt user@172.16.10.2:/chemin/destination/``

si le port par defaut est changé :

``scp -P (pot modifié) /chemin/vers/fichier.txt user@172.16.10.2:/chemin/destination/``

Et pour récupérer un fichier de VM2 vers VM1 :

``scp user@172.16.10.2:/chemin/vers/fichier.txt /chemin/local/``

si le port par defaut est changé :

``scp -P (pot modifié) user@172.16.10.2:/chemin/vers/fichier.txt /chemin/local/``

---

## 8. Bonnes pratiques de sécurité avec SSH

- Utiliser des clés RSA de 4096 bits ou plus pour une meilleure sécurité.
- Désactiver l'authentification par mot de passe une fois que les clés SSH sont configurées.
- Changer le port par défaut de SSH pour éviter les attaques par force brute sur le port 22.
- àLimiter les utilisateurs autorisés à se connecter via SSH avec la directive AllowUsers.
