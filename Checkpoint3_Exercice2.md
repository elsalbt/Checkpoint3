# Partie 1 : Gestion des utilisateurs

### Q.2.1.1
Création d'un nouveau compte avec la commande `adduser`  
![1](https://github.com/user-attachments/assets/e6f68fdc-4ad9-44ad-b4a9-059d61d546c8)  


### Q.2.1.2
Le compte que je viens de créer est pour mon usage personnel, j'opterais pour le moindre privilège et ne lui donnerais donc pas de privilèges `root`.
Je m'assurerais aussi que le mot de passe de ce compte soit sécuritaire.
Je placerais ce compte dans le groupe `users` et je lui créerais son répertoire personnel dans `/home`


# Partie 2 : Gestion des utilisateurs

### Q.2.2.1
Pour désactiver le compte `root`, il faut modifier le fichier `sshd_config` en utilisant la commande suivante `nano /etc/ssh/sshd_config`  
Il faut décommenter et modifier la ligne `PermitRootLogin` comme suit :  
![2](https://github.com/user-attachments/assets/bf4d0d3e-3338-4301-ba13-17821ad2f047)  


### Q.2.2.2
Pour autoriser uniquement l'accès à distance à mon compte personnel, il faut ajouter la ligne `AllowUsers yohann` dans le fichier `sshd_config`  
![3](https://github.com/user-attachments/assets/7752890d-334e-474b-8d02-803fa011636c)  


### Q.2.2.3
Pour mettre en place une authentification par clé valide et désactiver l'authentification par mot de passe, il faut :
  - Décommenter la ligne `PubkeyAuthentication` et bien vérifier qu'il est écrit `yes`
  - Décommenter la ligne `PasswordAuthentication` et écrire `no`  
![4](https://github.com/user-attachments/assets/0f2d1ec4-4a0d-45a9-83f2-3ef227f64d53)  

  - Redémarrer le service ssh avec la commande `systemctl restart sshd`
  - Vérifier l'état du service ssh avec la commande `systemctl status sshd`
![5](https://github.com/user-attachments/assets/4d83cd88-0b0b-4ade-8498-01af31ff4666)  



# Partie 3 : Analyse du stockage

### Q.2.3.1
On utilise la commande `lsblk -f`  
![Capture d’écran 2025-01-21 165429](https://github.com/user-attachments/assets/53db371c-ce1a-43fe-a005-ab0835117782)  
Les fichiers montés sont ext2 et ext4.  


### Q.2.3.2
![6](https://github.com/user-attachments/assets/4bfbd014-c754-4089-9e56-534a6d0ed8fa)  
Je vois que du `RAID1` est utilisé sur `md0` et que du `LVM` est utilisé `cp3-vg`


### Q.2.3.3
Ajout d'un disque dur sur VirtualBox  
![7](https://github.com/user-attachments/assets/8e31067a-135b-4044-9106-6f0f1134907f)  
Vérification de la présence de ce nouveau disque dur avec la commande `lsblk`  
![8](https://github.com/user-attachments/assets/74cc6ca3-ccd3-45fe-9755-693d79af20f0)  
Configuration du disque avec la commande `fdisk`  
![9](https://github.com/user-attachments/assets/067a3098-4744-46eb-80ce-7f96e34cfbfc)  
Mise en place de la partition  
![10](https://github.com/user-attachments/assets/e1b7f4d1-ee9a-489f-94d7-e9d68acc6f34)  
On donne à la partition le type RAID  
![11](https://github.com/user-attachments/assets/d35a4b2a-d889-472c-a290-4589ef475c89)  
Vérification de la partition  
![12](https://github.com/user-attachments/assets/8eb7db07-6d46-4527-ba5b-9e2c52201936)  
Montage du disque `sdb` sur le RAID `/dev/md0` avec la commande `mdadm --manage /dev/md0 --ad /dev/sdb1`  
![13](https://github.com/user-attachments/assets/5fed9360-f287-4cc2-b76d-c052c1917075)  


### Q.2.3.4
Afficher les Volume Groups disponibles avec la commande `vgdisplay`  
![14](https://github.com/user-attachments/assets/890f7a50-6f5d-428c-a9fc-30ea09f54c61)  
Créer le nouveau volume logique de 2GO du nom de `lv_svg`  
![15](https://github.com/user-attachments/assets/c3ea91a5-9168-4f75-9d4a-b53165037109)  
Formater le volume en lui donnant le nom `save`  
![17](https://github.com/user-attachments/assets/13b71f78-7edc-4c05-9ad1-ae97d9cfb7fb)  
Monter le LV  
![18](https://github.com/user-attachments/assets/3a666bc7-cdb3-48b4-b140-8a1b17fcb3f9)  
Configurer le fichier `/etc/fstab`  
![19](https://github.com/user-attachments/assets/dcbda843-2943-4189-95f1-d308c4ea993a)  
Vérifier que le montage est bon avec la commande `df -h`  
![20](https://github.com/user-attachments/assets/75a9eaa8-b492-4cb4-829e-85d77b9c5743)  


### Q.2.3.5
Vérifier l'espace disponible dans le VG avec la commande `vgdisplay`  
![21](https://github.com/user-attachments/assets/c9c514e8-c0b3-450e-8c57-e25149696ca3)  



# Partie 4 : Sauvegardes

### Q.2.4.1
- **Bareos-dir** : le Director s'occupe de la configuration des tâches de Bareos comme gérer les sauvegardes (leur planification, contrôle et lancement). Il s'installe sur le serveur.
- **Bareos-sd** : le Storage Daemon s'occupe du stockage des sauvegardes.
- **Bareos-fd** : le File Daemon est l'agent installé sur les clients, il s'occupe d'envoyer les données à sauvegarder sur le Storage Daemon.



# Partie 5 : Filtrage et analyse réseau

### Q.2.5.1
Vérification des règles appliquées sur Netfilter avec la commande `nft list ruleset`
![22](https://github.com/user-attachments/assets/c5d049a0-e688-42b6-b537-cddf8ce2ea3c)


### Q.2.5.2
- `ct state established,related accept` autorise les connexions établies
- `iifname "lo" accept` autorise trafic de bouclage
- `tcp dport 22 accept` autorise les connexions TCP destinées au port 22 (port SSH)
- `ip protocol icmp accept` autorise les paquets ICMPV4
- `ip6 nexthdr icmpv6 accept` autorise les paquets ICMPV6


### Q.2.5.3
- `ct state invalid drop` interdit les paquets invalides
- Tous les paquest qui ne sont pas autorisés par une règle sont interdits


### Q.2.5.4
![23](https://github.com/user-attachments/assets/842148fc-798c-46cb-a4a2-70a9e7dbd473)



# Partie 6 : Analyse de logs

### Q.2.6.1
![24](https://github.com/user-attachments/assets/b4bcdfcd-aeae-4647-ab4e-19d61b8f54a0)
