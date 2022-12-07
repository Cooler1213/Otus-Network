### **Настройка DHCPv4**

#### **Схема Paket Tracer:**

![Scheme](https://github.com/Cooler1213/Otus-Network/blob/37bb58b96d1e507d3eb2a657462a07a2de71d186/Lab/DHCPv4/Scheme%20v4.png)

#### Задача:

Часть 1. Построение сети и настройка основных параметров устройств.  
Часть 2. Настройка и проверка двух серверов DHCPv4 на R1.  
Часть 3. Настройка и проверка ретрансляции DHCP на R2.   

Таблица адресации:

![IP](https://github.com/Cooler1213/Otus-Network/blob/f4ca1c7a366edbb844ca06151599a39213227bc0/Lab/DHCPv4/IP.png)

Схема адресации должна соответствовать:
1. "Подсеть A" 58 хостов
2. "Подсеть B" 28 хостов
3. "Подсеть C" 12 хостов

![Subnet](https://github.com/Cooler1213/Otus-Network/blob/f4ca1c7a366edbb844ca06151599a39213227bc0/Lab/DHCPv4/Subnet.png)

Vlans:

![Vlan](https://github.com/Cooler1213/Otus-Network/blob/f4ca1c7a366edbb844ca06151599a39213227bc0/Lab/DHCPv4/Vlan.png)

#### Часть 1. Построение сети и настройка основных параметров устройств.

Задаем начальную конфигурацию всех устройств.

> hostname "имя"  
> no ip domain lookup  
> enable secret class  
> line console 0   
> password cisco  
> login  
> line vty 0 4  
> password cisco  
> login  
> service password-encryption  
> banner motd $  
> This is a secure system. Authorized Access Only! $

На R1 и R2 настраиваем интерфейсы и адресацию соответственно таблицам адресации и включаем их.

Прописываем маршруты по умолчанию на каждом из маршрутизаторов.

> R1(config)# ip route 0.0.0.0 0.0.0.0 10.0.0.2  
> R2(config)# ip route 0.0.0.0 0.0.0.0 10.0.0.1

Проверяем статическую маршрутизацию командой ping на адрес G0/0/1 R2 от R1

> R1#ping 192.168.1.97 
>
> Type escape sequence to abort.
>
> Sending 5, 100-byte ICMP Echos to 192.168.1.97, timeout is 2 seconds:
>
> !!!!!
>
> Success rate is 100 percent (5/5), round-trip min/avg/max = 0/0/0 ms
>



Настраиваем S1 и S2 соответственно таблицам адресации.  
Интерфейсы управления:  
S1 VLAN200 , ip 192.168.1.2  
S2 VLAN1 , ip 192.168.1.66  
Выключаем интерфейсы, которые не используются.  
Указываем для вланов управления шлюз по умолчанию.

> S1(config)# ip default-gateway 192.168.1.1  
> S2(config)# ip default-gateway 192.168.1.65

На данный момент если на PC включить dhcp они получат адреса из подсети  169.254.x.x "Automatic Private IP Address (APIPA)" 



#### Часть 2. Настройка и проверка двух серверов DHCPv4 на R1. 

Первый пул R1_Client_LAN

> R1(config)# ip dhcp excluded-address 192.168.1.1 192.168.1.5  
> R1(config)# ip dhcp pool R1_Client_LAN   
> R1(dhcp–config)# network 192.168.1.0 255.255.255.192  
> R1(dhcp–config)# domain-name ccna-lab.com  
> R1(dhcp–config)# default-router 192.168.1.1  
> R1(dhcp–config)# lease 2 12 30

Второй пул пул R2_Client_LAN

> R1(config)# ip dhcp excluded-address 192.168.1.97 192.168.1.101  
> R1(config)# ip dhcp pool R2_Client_LAN  
> R1(dhcp–config)# network 192.168.1.96 255.255.255.240  
> R1(dhcp–config)# default-router 192.168.1.97  
> R1(dhcp–config)# domain-name ccna-lab.com  
> R1(dhcp–config)# lease 2 12 30

Проверку настроек, выданных адресов, статистику можно посмотреть с помощью следующих команд.

> show ip dhcp pool   
> show ip dhcp bindings  
> show ip dhcp server statistics

Проверяем на PC-A что настройки от сервера получены.
В командной строке вводим поочередно: 

>
>  
> ipconfig /renew  
> ipconfig 

![ipconfig PC-A](https://github.com/Cooler1213/Otus-Network/blob/d7039c92b4f3739e5bfafbcd0d5d3aaf6c209c8c/Lab/DHCPv4/ipconfig%20PC-A.png)

Проверяем доступность  G0/0/1  R1 с PC-A

![Ping PC-A to R1](https://github.com/Cooler1213/Otus-Network/blob/682b6cb15ca98a656a395efcf787d96d899a8f19/Lab/DHCPv4/Ping%20PC-A%20to%20R1.png)

#### Часть 3. Настройка и проверка ретрансляции DHCP на R2.

Настраиваем ip helper-address на интерфейсе G0/0/1 R2, указав IP-адрес R1 G0/0/0.

> ip helper-address 10.0.0.1

Открываем командную строку PC-B и командами 

> ipconfig /renew
>
> ipconfig

Проверяем, что PC-B получил IP адрес.

Командой ping проверяем доступность интерфейса G0/0/1 на R1.

Например  G0/0/1.100 

