Projet-IT_Reseau_PME

Projet personnel de conception et paramétrage d'un réseau destiné à un usage de PME avec capacité de redondance

Le fichier Packet Tracer est mis à disposition dans le repository
Contexte

L'entreprise La Cabane Joviale, une PME d'un vingtaine de salariés, souhaite conçevoir le réseau de son siége social, cette entreprise dispose d'un bâtiment de 3 étages et se compose :

Au rez-de-chaussée :

    D'un service d'acceuil avec une standardiste
    D'un service contentieux de 4 personnes
    D'un service communication de 4 personnes

Au premier étage :

    D'un service comptable de 4 personnes
    D'une salle de réunion dans laquelle les invités pourront se connecter en Wi-Fi
    D'un service informatique interne de 2 personnes

Au second étage :

    D'un service Ressources humaines de 2 personnes
    Du directeur de l'entreprise ainsi que sa secretaire

Chaque membre du personnel de l'agence se doit d'avoir son propre téléphone fixe à son bureau pour travailler, un photocopieur sera mis a disposition à chaque étage pour chacun des services de cet étage (et seulement pour cet étage).

Il à été souligné que la continuité de services était une grande préoccupation de l'entreprise, ainsi pour rendre l'infrastructure réseau plus résiliente aux pannes, de la redondance sera effectuée au niveau de la couche distribution et des serveurs fournissant les services DHCP/DNS et AD.


Topologie

Voici la représentation globale finalisée de l'infrastructure

<img width="836" height="693" alt="image" src="https://github.com/user-attachments/assets/f95eea96-7f7d-4b0b-8780-3def6fbc426d" />

Comme on y voit pas grand chose, voici l'infrastrucure, découpée par section :

Switchs et routeurs :

<img width="1021" height="649" alt="image" src="https://github.com/user-attachments/assets/08c6201a-d6ef-4069-aa7c-4a66b67e1417" />

Rez-de-chaussée :

<img width="978" height="647" alt="image" src="https://github.com/user-attachments/assets/99bd46cd-4bf6-455e-b922-b8c945b50080" />

1er Etage :

<img width="787" height="683" alt="image" src="https://github.com/user-attachments/assets/2ddf9421-ee50-46f4-9ea5-e30fb63753ce" />

2e Etage :

<img width="903" height="386" alt="image" src="https://github.com/user-attachments/assets/b84cee55-346e-44ea-83da-5be85a6a720a" />



## Mots de passe

- Mode privilégié : Admin123!
- SSH : Ssh123!
- VTP : Vtp123!

- WIFI :
    - SSID : Cabane_Reunion
    - MDP : P4ssw0rd



## Plan d'adressage

- @ WAN : 203.0.115.1

- VLAN 10 : ACCEUIL 192.168.10.0 /24
- VLAN 20 : COMMUNICATION 192.168.20.0 /24
- VLAN 30 : CONTENTIEUX 192.168.30.0 /24
- VLAN 40 : COMPTABILITE 192.168.40.0 /24
- VLAN 50 : RESSOURCES HUMAINES 192.168.50.0 /24
- VLAN 60 : DIRECTION 192.168.60.0 /24
- VLAN 70 : REUNION 192.168.70.0 /24
- VLAN 80 : VOICE 192.168.80.0 /24
- VLAN 90 : COPIEURS 192.168.90.0 /24
- VLAN 99 : ADMINS 192.168.99.0 /24
- VLAN 100 : SERVEURS 192.168.100.0 /21
- VLAN 999 : BLACKHOLE
- PTP DST-SW1/EDGE-RTR : 10.0.0.0 /30
- PTP DST-SW2/EDGE-RTR : 10.0.0.4 /30



## Sécurité

- Blackhole VLAN 999 pour les interfaces inutilisées
- VLAN par défaut 999 sur DST-SW1 et ACC-SW1
- Seuls les VLANs 10,20,30,40,50,60,70,80,90,99 et 100 sont autorisés sur les liens trunks
- ACL étendue ACCES_COPIEURS sur DST-SW1 et  DST-SW2 :
    - Restreint le traffic accédant au SVI VLAN 90 aux seuls VLANS de l'etage seulement 
- ACL standard SSH sur les équipements réseaux :
    - Autorise l'accéss aux lignes vty 0 15 seulement au VLAN 99 (admins)
- Configuration du service DHCP Snooping sur les switchs de la couche Access
- Portfast et BPDUGuard activé sur les ports access de la couche Access
- SSID du Wi-Fi de la salle de réunion non diffusé, clé WPA2-PSK utilisée pour se connecter



## Serveurs

Le VLAN 100 est composé d'un serveur délivrant les services DHCP et Controleur de Domaine, il fournit les adresses IP (excepté pour le vlan 90 COPIEURS) et gére le domaine de l'infrastrucure.

La continuité de ces services est assurée par un 2e serveur de backup, qui est la pour servir de cluster de basculement en cas de panne du premier serveur.



## Protocoles utilisés

OSPF :
- Protocole de routage dynamique utilié pour peupler la table de routage de EDGE-RTR et de la couche de distribution

VTP :
- Protocole de diffusion des VLANS entre les différents switchs
- VTP mode serveur sur la couche distribution, mode client sur la couche access
- Domaine VTP : cabanejoviale.local
- Password VTP : Vtp123!

HSRP :
- Protocole de redondace de la passerelle par defaut entre DST-SW1 et DSW-SW2
- DST-SW1 :
    - Active router
    - Mode preempt
    - Priority 110
    - Track G0/1 (decrement 10)
- DST-SW2 :
    - Standby router
    - Priority 105 
    - Track G0/1 (decrement 10)

Spanning Tree :
- Protocole de prévention de boucle de couche 2
- Mode RPVST+
- DST-SW1 :
    - Root Bridge (Bridge priority 4096)
- DST-SW2 :
    - Root Secondary (Bridge priority 8192)

