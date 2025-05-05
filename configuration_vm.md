# Configuration des VM

## Description de l'installation

Objectif :
- VM 1 : au moins 2 CPU, 2 GB de RAM, 15 GM disque
- VM 2 : 1 CPU et 1 GB de RAM suffisent, 15 GB disque
- OS installé : Debian 12
- Réseau : accès par pont (bridge)

Pré-requis :
- Installer WSL2 avec une distribution Debian 12 : https://tqdev.com/2024-running-debian-on-windows-10-with-wsl
- Installer ensuite Ansible sur WSL2

## Installation avec Windows Hyper-V

Nous allons ici mettre en place un sous-réseau NAT, ce qui permettra de :
- accéder à internet depuis les VM
- garantir une indépendance par rapport à la connexion générale de l'hôte

Problème : il faut le mode réseau "mirrored" pour accéder aux VM depuis WSL,
ou alors une configuration réseau assez complexe

### Pré-requis

- Activer Hyper-V : https://gist.github.com/HimDek/6edde284203a620745fad3f762be603b
- Disposer du mode "mirrored" sous WSL2 (possible sur Windows 11) :
https://learn.microsoft.com/fr-fr/windows/wsl/networking#mirrored-mode-networking

### Création du sous-réseau NAT

Avec Powershell en mode administrateur :
```
# Création d'un switch interne, récupérer son InterfaceIndex
New-VMSwitch -SwitchName "NAT-VM" -SwitchType Internal
$switchInterfaceId = ...

# Création de la passerelle de sous-réseau
New-NetIPAddress -IPAddress 10.0.1.1 -PrefixLength 24 -InterfaceIndex $switchInterfaceId

# Création du sous-réseau NAT
New-NetNat -Name NAT-VM-Network -InternalIPInterfaceAddressPrefix 10.0.1.0/24
```

### Configuration des VM sous Hyper-V

- Associer l'image iso Debian 12
- Sélectionner le commutateur virtual "NAT-VM" dans la carte réseau
- Désactiver le démarrage sécurisé Windows
- Sélectionner les paramètres CPU/RAM
- Installer Debian 12 avec: 
  - user adminvm
  - réseau :
    - IP : 10.0.1.10 (VM1) / 10.0.1.10 (VM2)
    - Netmask : 255.255.255.0
    - Gateway : 10.0.1.1
    - DNS : 8.8.8.8 1.1.1.1
  - seulement serveur ssh et paquets de base

### Editer le fichier /etc/hosts local

Afin d'attribuer des noms de hosts aux VM, éditer le fichier `C:\Windows\System32\drivers\etc\hosts` (mode administrateur) :
```
# VM Debian 1
10.0.1.10 vm-debian-1 vm-debian-1.perso.com

# VM Debian 2
10.0.1.11 vm-debian-2 vm-debian-2.perso.com
```

## Installation avec VirtualBox

Ici, l'accès par pont (bridge) sera configuré pour Wifi et Ethernet afin de couvrir les deux modes de connexion

Problème : dépendance avec la connexion de l'hôte

### Configuration de la VM

- Lors de l'installation : choisir un nom d'utilisateur et l'enregistrer pour config Ansible
- Réseau : sélectionner le réseau avec lequel la VM est connectée dans Settings (enp0s3 pour WiFi, enp0s8 pour Ethernet),
puis passer la méthode de connexion IPv4 en `Manual` avec les paramètres suivants, si l'IP du routeur est 192.168.8.1 :
  - Addresses :
    - Address: 192.168.8.250 (VM1) / 192.168.8.251 (VM2)
    - Netmask : 255.255.255.0
    - Gateway : 192.168.8.1 (IP routeur)
    - DNS : 192.168.8.1,8.8.8.8,1.1.1.1

- Activer le serveur ssh
```
# Passer en root
adminvm@vbox:~$ su -
root@vbox:~# apt-get update
root@vbox:~# apt-get install openssh-server
root@vbox:~# systemctl status ssh
```

### Editer le fichier /etc/hosts local

Afin d'attribuer des noms de hosts aux VM, éditer le fichier `C:\Windows\System32\drivers\etc\hosts` (mode administrateur) :
```
# VM Debian 1
192.168.8.250 vm-debian-1 vm-debian-1.perso.com

# VM Debian 2
192.168.8.251 vm-debian-2 vm-debian-2.perso.com
```

## Configuration des users sur les VM

Se connecter en ssh aux VM avec l'user adminvm

```
adminvm@vm-debian-1:~$ su -
root@vm-debian-1:~# apt-get update
root@vm-debian-1:~# apt-get install sudo
root@vm-debian-1:~# USERNAME="adminvm"
root@vm-debian-1:~# echo "$USERNAME ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/$USERNAME
root@vm-debian-1:~# exit

adminvm@vm-debian-1:~$ 
```

- Créer l'utilisateur ansible
```
# Créer l'utilisateur ansible avec des droits sudo sur toutes les commandes
adminvm@vm-debian-1:~$ sudo useradd -m ansible -s /bin/bash
adminvm@vm-debian-1:~$ sudo usermod -aG sudo ansible
adminvm@vm-debian-1:~$ echo "ansible ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/ansible
 
# Supprimer l'authentification par mot de passe
adminvm@vm-debian-1:~$ sudo passwd -l ansible
```
A partir d'une clef publique SSH générée précédemment :
```
adminvm@vm-debian-1:~$ sudo su - ansible
ansible@vm-debian-1:~$ mkdir -p ~/.ssh
ansible@vm-debian-1:~$ echo "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx" | tee -a ~/.ssh/authorized_keys
```

## Configuration avec Ansible

- Configurer `inventories/int/hosts` avec les deux VM :
```
[application]
vm_debian_1 ansible_host=vm-debian-1 ansible_user=ansible
vm_debian_2 ansible_host=vm-debian-2 ansible_user=ansible
```

- Playbook de configuration: à partir de son poste, installer Docker et déployer le certificat racine
```
ansible-playbook -i inventories/int playbook-setup.yml
```