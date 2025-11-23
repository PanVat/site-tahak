###### **--- LINK AGGREGATION ---**



**Na obou Switch1 a Switch2:**



1. Switch(config)# interface port-channel 1
2. Switch(config-if)# switchport mode trunk
3. Switch(config-if)# switchport trunk allowed vlan <čísla\_vlan, ...>

!

1. Switch(config)# interface range <název\_int>
2. Switch(config-if-range)# channel-protocol lacp
3. Switch(config-if-range)# channel-group 1 mode active

 

 	- nastavit ještě 'trunk' na Switchi u portu, který vede k Routeru







###### **--- SUBINTERFACE ---**



**Na prvním portu Routeru, který vede do Switche**



1. Router(config)#int g0/2.200
2. Router(config-subif)# encapsulation dot1q <číslo>



 	- za <číslo> dáme číslo VLAN, z které přidělujeme gateway, tady je to '200'

 	- 'enkapsulace' musí být u každé sítě, i když je zde jen 1 VLAN





###### **--- STATICKÝ ROUTING ---**



1. Router(config)# ip route <cílová\_vlan> <maska> <next\_hop>
2. Router(config)#ip routing

!

1. Router(config)# ip route 0.0.0.0 0.0.0.0 <next\_hop>



 	- nastavíme u každé sítě, kterou router nezná (není k ní "přímo připojený"), <cílová\_vlan> je její adresa

 	- 'ip routing' je nutný, jinak nám nebude routování fungovat

 	- <next\_hop> je adresa rozhraní, které je o adresu vedle Routeru (je u jiného Routeru) - je vždy stejný u všech sítí, které nastavujeme

 	- 'default route' se nastavuje na interní Router, který je uvnitř sítě - své packety pak posílá routeru, který je mezi internetem a privátní sítí

 	- její <next\_hop> je port, který patří hraničnímu routeru





###### **--- DHCP ---**



1. Router(config)# ip dhcp pool <název\_poolu>
2. Router(dhcp-config)# network <adresa\_sítě> <maska\_sítě>
3. Router(dhcp-config)# default-router <ip\_adresa\_dg>
4. Router(dhcp-config)# dns-server 8.8.8.8

!

1. Router(config)# ip dhcp excluded-address <od\_ip> <do\_ip>  # Z tohoto rozsahu se nebudou přidělovat IP adresy



!!! Nastavíme helper-adress tam, kde se nastavovaly DGs (na subinterface) !!!

 

 	- zadáme síť, ze které se budou tvořit IP adresy

 	- <ip\_adresa\_dg> je adresa, kterou jsme nastavovali jako DG

 	- <od\_ip> a <do\_ip> je rozsah, ze kterého se nebudou přidělovat IPs (jako <od\_ip> dáme DG adresu, protože ta je vyhrazená)

 	- jako 'helper-address' se nastavuje adresa Routeru (tj. interface routeru, který vede směrem k VLANám)





###### **--- SSH ---**



Router(config)# hostname R1

R1(config)# ip domain-name cisco.com

R1(config)# username cisco privilege 15 secret cisco

R1(config)# line vty 0 4

R1(config-line)# transport input ssh

R1(config-line)# login local

!

R1(config)# crypto key generate rsa

R1(config)# enable secret cisco

R1(config)# ip ssh version 2



&nbsp;	- 'do show ip ssh' - zobrazí se nám SSH s verzí

&nbsp;	- 'ssh -l cisco <adresa\_routeru>' - testujeme funkčnost





**Zákaz SSH pro VLANy jiné než uvedenou**



R1(config)# ip access-list extended SSH\_ACL

R1(config-ext-nacl)# permit tcp 192.168.0.0 0.0.1.255 any eq 22

R1(config-ext-nacl)# deny tcp any any eq 22

!

R1(config)# line vty 0 4

R1(config-line)# access-class SSH\_ACL in



&nbsp;	- nastavíme na portu, který vede směrem do internetu

&nbsp;	- 150 pro 'extended ACL'

&nbsp;	- 'do show access-lists 150'





###### **--- ZÁLOHA KONFIGURACE ---**





**Nastavujeme na Switchi, kde má být konfigurace**



Switch(config)#int vlan 100

Switch(config-if)#ip address 192.168.0.254 255.255.254.0

Switch(config)#ip default-gateway 192.168.0.1

!

Switch(config)#ip ftp username cisco

Switch(config)#ip ftp password cisco





###### **--- LOGGING ---**





**Nastavujeme na obou Routerech, kde to má být**



R1(config)#logging 192.168.3.3

R1(config)#logging trap

R1(config)#service timestamps log datetime msec





###### **--- NAT ---**



R1(config)# access-list 1 permit <vlan> <wildcard>

R1(config)# ip nat inside source list 1 interface g0/0 overload

!

R1(config-if)# ip nat inside

R1(config-if)# ip nat outside



&nbsp;	- g0/0 je zde interface, který vede do internetu





###### **--- PORT FORWARDING ---**



R1(config)#ip nat inside source static tcp <adresa\_serveru> 443 <dg\_internetu> 443

R1(config)#ip nat inside source static tcp <adresa\_serveru> 21 <dg\_internetu> 21

R1(config)#ip nat inside source static tcp <adresa\_serveru> 22 <dg\_internetu> 22

R1(config)#ip nat inside source static tcp <adresa\_serveru> 23 <dg\_internetu>23



&nbsp;	- <adresa\_serveru> je server, na kterém to má být nastavené

&nbsp;	- <dg\_internetu> je default gateway internetu (kterou server nastavenou NEMÁ)

