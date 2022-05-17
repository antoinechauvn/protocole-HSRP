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
