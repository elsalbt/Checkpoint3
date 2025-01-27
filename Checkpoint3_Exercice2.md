## Exercice 2 : Manipulations pratiques sur VM Linux 


### Partie 1 : `Gestion des utilisateurs`

#### Q.2.1.1 Sur le serveur, créer un compte pour ton usage personnel.

![image](https://github.com/user-attachments/assets/7cf862a4-b8d7-4eb9-a6ba-eb711c493c54)


Q.2.1.2 Quelles préconisations proposes-tu concernant ce compte ?

* ####  Pour mon compte à usage personnel, je lui donnerais le moindre privilège pour qu'il ait accès juste a ce dont il a besoin. Je mettrais également un mot de passe robuste.


### Partie 2 : `Configuration de SSH`

### Un serveur SSH est lancé sur le port par défaut.
### Il est possible de s'y connecter avec n'importe quel compte, y compris le compte root.

#### Q.2.2.1 Désactiver complètement l'accès à distance de l'utilisateur root.
* #### editer le fichier  /etc/ssh/sshd_config.d/local.conf, ainsi on garde la configuration de base vierge et le fichier local.conf sera lut en premier
        nano  /etc/ssh/sshd_config.d/local.conf

![image](https://github.com/user-attachments/assets/560a1712-2c38-47d6-9b83-9ed930dc0df1)


#### Q.2.2.2 Autoriser l'accès à distance à ton compte personnel uniquement.

* #### Toujours dans /etc/ssh/sshd_config.d/local.conf
![image](https://github.com/user-attachments/assets/d0e1281b-071e-4737-b7eb-b68a9690cf13)

#### Q.2.2.3 Mettre en place une authentification par clé valide et désactiver l'authentification par mot de passe
 
 ### 1) Connection avec Putty
 
* #### Sur putty créer un fichier pour le partage
![image](https://github.com/user-attachments/assets/433a1cd5-305a-4b16-997c-805058ec102f)

* #### accepter la connection
![image](https://github.com/user-attachments/assets/794d7f96-f895-45b5-98d9-e8024db69d7e)

* #### renseigner le login et le mot de passe
![image](https://github.com/user-attachments/assets/f7c83761-9b92-467f-a08d-f09f30b51843)



### 2) Création de la clé avec Puttygen

* #### Renseigner RSA 4096 et Generate

* #### La clé :
![image](https://github.com/user-attachments/assets/6d3ff8ab-7330-40c5-ad22-b728f23ff120)

* #### Sauvegarder les clés :
  ![image](https://github.com/user-attachments/assets/90adaa2d-6944-40eb-a9a6-f0d63d5d1125)

* #### Création du dossier .ssh et du fichier /authorized_keys
![image](https://github.com/user-attachments/assets/95398e2b-35f6-4009-a4a3-c408805cab26)

* #### copier la clé générée dan Puttygen dans le fichier /authorized_keys
* #### Dans Putty Connection -> SSH -> Auth -> Credentials et mettre l'emplacement de la clé privée
![image](https://github.com/user-attachments/assets/6e710b58-1024-442b-b0b4-b735c6e54cc1)

* #### Connection -> Data et mettre le login à utiliser sur le serveur => sednal
![image](https://github.com/user-attachments/assets/41d4737c-e3e5-4859-b44a-06ee349ff245)


* #### Sauvegarderet lancer la connection sans reiseigner login ou clé






https://github.com/user-attachments/assets/7cddfdd3-1165-4dcf-a5b7-34662f262e44





#### 2) Interdiction de connection par MDP

![image](https://github.com/user-attachments/assets/c811cb09-1b3d-4651-81fb-ef276ad35d9c)



### Partie 3 : Analyse du stockage

#### Q.2.3.1 Quels sont les systèmes de fichiers actuellement montés ?
                lsblk
#### les fichier sont ext2 et ext4

#### Q.2.3.2 Quel type de système de stockage ils utilisent ?

#### 1) une partion sda1
#### 2) md0 en raid 1 md0p1/md0p2/md0p5             
#### 3) deux partions LVM cp3 root et cp3 swap_1   

![image](https://github.com/user-attachments/assets/37cec455-072b-4462-ad59-8958fae6bd8f)

#### Q.2.3.3 Ajouter un nouveau disque de 8,00 Gio au serveur et réparer le volume RAID

* #### Ajouter le disque
 ![image](https://github.com/user-attachments/assets/e6ec1b17-14a0-4e2a-a44c-8a7a2fe0b36e)

* #### Création d'une partition sur le disque de 8 Go
           fdisk /dev/sdb             
* #### Réparer le RAID
          mdadm --add /dev/md0 /dev/sdb1

![image](https://github.com/user-attachments/assets/a3c275cd-eba9-40e5-91d1-0b7d17330dbb)

#### Q.2.3.4 Ajouter un nouveau volume logique LVM de 2 Gio qui servira à héberger des sauvegardes. Ce volume doit être monté automatiquement à chaque démarrage dans l'emplacement par défaut : /var/lib/bareos/storage.

* #### Intéroger les groupes de volumes
![image](https://github.com/user-attachments/assets/26c9634b-7e69-4e6b-81fd-f59a1af13a65)
          
* #### Création du volume logique sauvegardes a partir de cp3-vg
![image](https://github.com/user-attachments/assets/53f4b548-2e3e-4d32-99d5-a67985591a7b)


* ### Montage au démarage
* #### formater le volume logique
![image](https://github.com/user-attachments/assets/08cdb93e-8e40-441c-80e2-253bc5185915)

* #### Editer /etc/fstab 
![image](https://github.com/user-attachments/assets/80de7401-9196-4927-91ee-62a8c31400af)

* #### Sauvegarder/redémarer et vérifier
![image](https://github.com/user-attachments/assets/5817689c-dbc6-4a84-b1fe-955e87990832)



#### Q.2.3.5 Combien d'espace disponible reste-t-il dans le groupe de volume ?
* #### <1.79 Gib
![image](https://github.com/user-attachments/assets/1a9b06c8-4f2d-4cd2-bc38-94728e29acef)


#### Partie 4 : Sauvegardes
Le logiciel bareos est installé sur le serveur.
Les composants bareos-dir, bareos-sd et bareos-fd sont installés avec une configuration par défaut.

#### Q.2.4.1 Expliquer succinctement les rôles respectifs des 3 composants bareos installés sur la VM.

* #### Bareos-dir (Directeur) gère la configuration et le planning des sauvegardes.
* #### Bareos-sd (Stockage) gère le stockage et la récupération des données sauvegardées.
* #### Bareos-fd (Client) exécute les sauvegardes et restaurations sur les systèmes clients.


#### Partie 5 : Filtrage et analyse réseau
#### Q.2.5.1 Quelles sont actuellement les règles appliquées sur Netfilter ?
                nft list rulset

#### Q.2.5.2 Quels types de communications sont autorisées ?
* #### ct state established, related accept : les connexions déjà établies

#### Q.2.5.3 Quels types sont interdit ?
* #### ct state invalid drop : paquets ne pouvant pas être identifiés et tout le reste

#### Q.2.5.4 Sur nftables, ajouter les règles nécessaires pour autoriser bareos à communiquer avec les clients bareos potentiellement présents sur l'ensemble des machines du réseau local sur lequel se trouve le serveur.

#### Rappel : Bareos utilise les ports TCP 9101 à 9103 pour la communication entre ses différents composants.
![image](https://github.com/user-attachments/assets/b518ce5f-ed9b-45ce-b845-9f99ef24d348)

#### Partie 6 : Analyse de logs
#### Q.2.6.1 Lister les 10 derniers échecs de connexion ayant eu lieu sur le serveur en indiquant pour chacun :

#### La date et l'heure de la tentative
#### L'adresse IP de la machine ayant fait la tentative
 #### ici il y en a deux mais avec l'option -n10 j'aurais pu filtrer les 10 dernières

 ![image](https://github.com/user-attachments/assets/615c1dde-e734-4b1e-9384-73285eb8c92c)
