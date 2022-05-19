# protocole-HSRP
Découverte du protocole HSRP


Hot Standby Router Protocol (HSRP) est un protocole propriétaire de Cisco implémenté sur les routeurs et les commutateurs de niveau 3 permettant une continuité de service. HSRP est principalement utilisé pour assurer la disponibilité de la passerelle par défaut dans un sous-réseau en dépit d'une panne d'un routeur.
HSRP, Host Standby Router Protocol est un protocole de redondance du premier saut (FHRP, First Hop redundancy Protocols)

![image](https://user-images.githubusercontent.com/83721477/168839422-d194d263-ac80-45b6-ac99-1b6e8ac34a3b.png)

La technologie HSRP permettra aux routeurs situés dans un même groupe (que l’on nomme `standby group`) de former un routeur virtuel qui sera l’unique passerelle des hôtes du réseau local.

# Principe de Fonctionnement

### Modes HSRP
* Mode Active (Actif)<br>Ce routeur portera l’adresse IP et l’adresse MAC virtuelle
* Mode Passive (Passif)<br>Les autres routeurs attendent que le routeur en mode active soit indisponible pour prendre sa place.

### Commandes HSRP
* La commande « standby priority xxx » définit une priorité au routeur. Celui qui possédera la plus grande valeur sera élus actif. Si la configuration du routeur ne stipule pas la priorité, alors la valeurs par défaut de 100 sera appliquée.
* La commande « standby track xxxxxx » permet de superviser une interface et de baisser de 10 la valeur de la priorité HSRP si elle devenait Down.
* La commande « standby ip xxx.xxx.xxx.xxx » indique l’adresse IP virtuelle partagée entre les deux routeurs.
* La commande « standby authentication« , permet de remplacer le mot de passe par défaut « Cisco    » (63 69 73 63 6F 00 00 00).

## Versions HSRP

### HSRP VERSION 1
* IPv4
* Adresse MAC utilisée : 0000.0C07.ACxx
* Utilise l’adresse multicast 224.0.0.2
* Groupe 0 au groupe 255
### HSRP VERSION 2
* IPv4 / IPv6
* Adresse MAC utilisée : 0000.0C9F.Fxxx
* Utilise l’adresse multicast 224.0.0.102
* Groupe 0 au groupe 4095

## Election des routeurs HSRP
1. Le routeur qui aura la plus haute priorité (ou « priority ») sera le routeur primaire ou principal du groupe HSRP.
    * Si égalité (100 par défaut), c’est le routeur qui aura l’IP la plus haute qui sera désigné comme routeur primaire.

2. Le ou les routeurs en mode Passive se tiendront informés de l’état de santé du routeur Active via des paquets “Hello” envoyés en mulitcast.

3. Le routeur en mode `Active` envoi des paquets `Hello` aux routeurs en mode Passive.<br>Cet intervalle de temps s’appelle :
`Hello Timer` (par défaut: toutes les 3 secondes)<br>
Si nos routeurs en mode Passive ne reçoivent plus de paquets “Hello”, ils considèrent que le routeur en mode Active est hors service, il va donc y avoir une nouvelle élection !

<br>Cet intervalle de temps s’appelle :<br>

`Hold-time Timer` (par défaut: 10 secondes soit 3x le Hello Timer)<br>Si aucun hello durant le Hold-time Timer alors le routeur actif est down
<br>Les valeurs “Hello Timer” et “Hold-time Timer” peuvent être changées administrativement.

* Si notre routeur en mode Active n’est plus en état de fonctionner, une nouvelle élection à lieu.<br>
Un routeur en mode Passive va donc passer en mode Active.

* La commande `preempt` va permettre à un routeur possédant une priorité supérieure aux autres de remplacer le routeur actuellement en mode Active (sans attendre la prochaine élection)

## Configurations

### Configuration de R1
```
R1(config)#int fa0/0
R1(config-if)#ip address 192.168.0.3 255.255.255.0
R1(config-if)#no shut
R1(config-if)#
R1(config-if)#standby 100 ip 192.168.0.1
R1(config-if)#standby 100 preempt
R1(config-if)#end
R1#
R1#sh run int fa0/0
Building configuration...

Current configuration : 145 bytes
!
interface FastEthernet0/0
ip address 192.168.0.3 255.255.255.0
duplex auto
speed auto
standby 100 ip 192.168.0.1
standby 100 preempt
end

R1#
```

### Configuration de R2
```
R2(config)#int fa0/0
R2(config-if)#ip address 192.168.0.2 255.255.255.0
R2(config-if)#standby 100 ip 192.168.0.1
R2(config-if)#standby 100 priority 110
R2(config-if)#standby 100 preempt
R2(config-if)#end
R2#
R2#sh run int fa0/0
Building configuration...

Current configuration : 171 bytes
!
interface FastEthernet0/0
ip address 192.168.0.2 255.255.255.0
duplex auto
speed auto
standby 100 ip 192.168.0.1
standby 100 priority 110
standby 100 preempt
endR2(config)#int fa0/0
R2(config-if)#ip address 192.168.0.2 255.255.255.0
R2(config-if)#standby 100 ip 192.168.0.1
R2(config-if)#standby 100 priority 110
R2(config-if)#standby 100 preempt
R2(config-if)#end
R2#
R2#sh run int fa0/0
Building configuration...

Current configuration : 171 bytes
!
interface FastEthernet0/0
ip address 192.168.0.2 255.255.255.0
duplex auto
speed auto
standby 100 ip 192.168.0.1
standby 100 priority 110
standby 100 preempt
end
```

* « Standby » est la commande qui permet la configuration du HSRP.
* « 10 » est le numéro du groupe HSRP dont fait partie l’interface.
*Note: Un routeur peut faire partie de plusieurs groupes HSRP*

## Vérification
Dans la sortie de commande suivante, le routeur actif est R2 (conformément à la priorité donnée dans la configuration des interfaces).
```
R1#show standby fastEthernet 0/0
FastEthernet0/0 - Group 100
State is Standby
4 state changes, last state change 00:00:51
Virtual IP address is 192.168.0.1
Active virtual MAC address is 0000.0c07.ac64
Local virtual MAC address is 0000.0c07.ac64 (default)
Hello time 3 sec, hold time 10 sec
Next hello sent in 2.712 secs
Preemption enabled
Active router is 192.168.0.2, priority 110 (expires in 9.696 sec)
Standby router is local
Priority 100 (default 100)
IP redundancy name is "hsrp-Fa0/0-100" (default)
R1#
```
