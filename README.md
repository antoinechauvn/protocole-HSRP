# protocole-HSRP
Découverte du protocole HSRP


Hot Standby Router Protocol (HSRP) est un protocole propriétaire de Cisco implémenté sur les routeurs et les commutateurs de niveau 3 permettant une continuité de service. HSRP est principalement utilisé pour assurer la disponibilité de la passerelle par défaut dans un sous-réseau en dépit d'une panne d'un routeur.

# Principe de Fonctionnement
![image](https://user-images.githubusercontent.com/83721477/168839422-d194d263-ac80-45b6-ac99-1b6e8ac34a3b.png)
* L'adresse IP de la passerelle est configurée sur deux routeurs différents. Une seule de ces deux interfaces est active. Si l'interface active n'est plus accessible, l'interface passive devient active.

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

* La commande `standby preempt` donne la priorité au routeur de protocole Cisco HSRP (Hot
Standby Router Protocol) pour que ce dernier devienne actif immédiatement. La priorité est
d'abord déterminée par la valeur de priorité configurée puis par l'adresse IP. Dans chaque cas,
une valeur plus élevée a une plus priorité plus grande. Quand un routeur d'une certaine priorité
acquiert un routeur de moins grande priorité, le routeur envoie un message « Coup ». Quand un
routeur actif de basse priorité reçoit un message « Coup » ou un message « Hello » depuis un
routeur actif de plus grande priorité, le statut du routeur change en « Speak » et un message
« Resign » est envoyé.

* La commande `standby track` vous permet de spécifier une autre interface sur le routeur pour
surveiller le processus HSRP, en vue de modifier la priorité HSRP pour un groupe donné. Si le
protocole de ligne de l'interface spécifiée échoue, la priorité HSRP est réduite. Ainsi, un autre
routeur HSRP de priorité plus élevée peut devenir le routeur actif si standby preempt est activé.
```
