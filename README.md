# AltDemo2025
1. Сначала нужно развернуть все виртуальные машины, которые указаны в таблице.

| Машина | RAM,ГБ | CPU | HDD/SDD, ГБ | Шаблон |
| ------ | ------ | --- | ----------- | ------ |
| ISP | 4 | 1 | 10 | «vESR» |
| HQ-RTR | 4 | 1 | 10 | «EcoRouter» |
| BR-RTR | 4 | 1 | 10 | «vESR» |
| HQ-SRV | 2 | 1 | 10 | «ALT-server» |
| BR-SRV | 2 | 1 | 10 | «ALT-server» |
| HQ-CLI | 3 | 2 | 15 | «ALT-workstatison» |
### EcoRouter
Установка `EcoRouter`
Имеем образ EcoRouter в qcow2 формате.

Шаг 1. Конвертируем qcow2 в vmdk

Используем qumu-img скачать можно [здесь](https://cloudbase.it/downloads/qemu-img-win-x64-2_3_0.zip)

ipv6.qcow2 - имя файла образа EcoRouter в формате qcow2

ipv6.vmdk - имя файла результата конвертации образа EcoRouter (может быть любым) 

```
qemu-img.exe convert -f qcow2 ipv6.qcow2 -O vmdk ipv6.vmdk
```
### vESR
Установка `vESR`
Предвариетльно нужно получить `ISO` образ `vESR`. Это можно сделать на оффсайте `ELTEX` по запросу или просто загуглить.
У vESR есть FREE лицензия. Ограничения только пропускной способности в 1Мб/с
1. Нажимаем `Create/Register VM`.
2. Выбераем пункт `Create a new virtual machine`.
3. Нажимаем `Next` для перехода к следующему шагу.
4. Установите имя виртуальной машины.
5. Поле `Compatibillity` заполняется автоматически. Оставляем как есть.
6. Поле `Guest OS family` выберите операционную систему `Linux`.
7. Поле "Guest OS version" выберите любую 64-битную версию, например "Other Linux (64-bit)".
8. Нажимаем `Next` для перехода к следующему шагу.
9. Выбираем где будет храниться виртуальная машина.
10. Нажимаем `Next` для перехода к следующему шагу.
11. Установите `Memory` минимум 4 Гб.
12. Изменита настройка контроллера HDD на `IDE.
13. Удаляем `SCSI Controller`.
14. Удаляем `USB Controller`.
15. Изменяем `CD/DVD Drive` на `Datastore ISO file`
16. ВЫбираем предварительно загруженный ISO образ vESR
17. Нажимаем `SELECT`.
18. Выделяем созданную виртуальную машину.
19. Включаем ее.
После загрузки появиться меню выбора действий.
20. Вибираем `vESR installation`.
21. Нажимаем `OK`.
22. При помощи пробела выделите диск куда будет проходить установка.
23. Нажимаем `ОК`.
24. Соглашаемся с продолжением установки `Yes`.
25. После завершения установки перезагружаем виртуальную машину. Введите комманду `reboot`.
После переустановки нужно ввести дефолтные учетные данные. После этого нужно сразу сменить парооль администратора
26. Логин `admin`.
27. Пароль `password`.
28. Введите новый пароль. Пароль должен отличаться от дефолтного.
29. Зафиксируйте изменения введите `commit`.
30. Подтвердите изменения `confirm`.

Установка завершена.


### 2. Создаем виртуальные сети.
1. Переходим в раздел `Networking` затем во вкладку `Virtual switchs`
2. Добавляем новый виртуальный коммутатор `add standart virtual switch`
3. Задаем имя новому виртуальному коммутатору `HQ-SW`
4. Раскрываем пункт `Security` изменяем все пункты на `Accept`
5. Нажимаем кнопку `ADD`
6. Переходим в раздел `Networking`
7. Переходим во вкладку `Port groups`
8. Добавляем новую порт группу `add port group`
9. Задаем имя порт группе в соответствие с топологией. В этом примере создаем сеть для `HQ-SRV`
10. Задаем номер `VLAN` в соответствие с заданием. В этом примере это номер 100
11. Подключаем эту порт группу в коммутатору, который создали на предыдущем этапе
12. Нажиамем кнопку `ADD` 
13. Проделываем этапы с 6 по 12 для `CLI-net`. Для `HQ-net` нужно сделать `trunk` режим. Для этого в поле `VLAN ID` задаем значение `4095`
В результате должно получиться `четыре` виртуальныйх коммутатора и `шесть` порт групп
### 3. Делаем внеполосное управление.
### Включаем службу в firewall на ESXi
1. Переходим в раздел `Networking`.
2. Открываем вкладку `Firewall rules`.
3. Ищем правило `remoteSerialPort`. Нажимаем ПКМ
4. Активируем это правило.
### Настраиваем виртуальные машины
1. Выключаем виртуальную машину нажав кнопку `Power off`
2. Переходим в редактирование виртуальноый машины нажав кнопку `Edit`
3. Добавляем новое устройство нажав на `Add other device`
4. Выбираем пункт `Serial port`
5. Изменяем тип порта на `Use network`
6. В строку `Port URL` добавляем значение в формате `telnet://0.0.0.0:<port>`. Номер порта для каждой виртуальной машины должен быть уникальным
7. Сохраняем изменения нажав кнопку `SAVE`
8. Включаем виртуальную машину.
Проделываем эту процедуру для всех виртуальных машин.
В виртуальных машинах с `Linux` нужно включить службу, добавить ее в автозагрузку и проверить статус:
```
systemctl start serial-getty@ttyS0.service
systemctl enable serial-getty@ttyS0.service
systemctl status serial-getty@ttyS0.service
```
Открывам любую консольку и подключаемся к `ESXi` с ее помощью по протоколу `telnet`
В этом примере используется `PUTTY`
1. Вписываем ip адрес вашего стенда.
2. Указываем порт, который указывали в настройуках виртуальной машины.
### Настройка консоли vESR

Для того, чтобы логи выводились на удаленную консоль делаем следующие настройки.

```
config
syslog console
virtual-serial
```

Перезагружаем устройство.
# Задание 1 - 2, 4. Произведите базовую настройку устройств. Настройка ISP. Настройте на интерфейсе HQ-RTR в сторону офиса HQ виртуальный коммутатор.
### 1.1. Настройте имена устройств согласно топологии. Используйте 
полное доменное имя
Alt Linux -
```
hostnamectl hostname HQ-SRV, BR-SRV; exec bash
```
EcoRouter, vESR -
```
hostname 123
```
### 1.2. На всех устройствах необходимо сконфигурировать IPv4
# Распределение IP адресов
 
| Имя устройства | Интерфейс | IP          | Маска           | Шлюз        |
| -------------- | --------- | ----------  | --------------- | ----------- |
| ISP            | gi1/0/1   | DHCP        |                 |             |
|                | gi1/0/2   | 172.16.5.1  | 255.255.255.240 |             |
|                | gi1/0/3   | 172.16.4.1  | 255.255.255.240 |             |
| HQ-RTR         | ge0       | 172.16.4.2  | 255.255.255.240 | 172.16.4.1  |      
|                | ge1.100   | 192.168.0.1 | 255.255.255.192 |             |      
|                | ge1.200   | 192.168.0.65| 255.255.255.240 |             |      
|                | ge1.999   | 192.168.0.81| 255.255.255.248 |             |      
|                | tunnel.1  | 172.16.1.1  | 255.255.255.252 |             |      
| BR-RTR         | gi1/0/1   | 172.16.5.2  | 255.255.255.240 | 172.16.5.1  |      
|                | gi1/0/2   | 192.168.1.1 | 255.255.255.224 |             |
|                | gre1      | 172.16.1.2  | 255.255.255.252 |             |
| HQ-SRV         | ens192    | 192.168.0.2 | 255.255.255.192 | 192.168.0.1 |      
| HQ-CLI         | ens192    | DHCP        | 255.255.255.240 | 192.168.0.65|      
| BR-SRV         | ens192    | 192.168.1.2 | 255.255.255.224 | 192.168.1.1 |      
# Настраиваем IP адреса

## ISP

```
configure

int gi1/0/3
ip address 172.16.4.1/28
ip firewall disable
no shutdown

int gi1/0/2
ip address 172.16.5.1/28
ip firewall disable
no shutdown

commit
confirm
```
## HQ-RTR - EcoRouter

Создаем сущность интерфейса и назначаем IP

```
int TO-ISP
ip address 172.16.4.2/28
no shutdown
```

Привязываем созданный интерфейс к физическому протоколу

1. Заходим в `port ge0`
2. Создаем service-instance `service-instance SI-ISP`
3. Указываем, что кадры на этом интерфейсе будут без тега `encapsulation untagged`
4. Привязываем сущьность интрефейса к порту `connect ip interface TO-ISP`

```
port ge0
service-instance SI-ISP
encapsulation untagged
connect ip interface TO-ISP
```

Создаем интерфейсы, которые будут обрабатывать трафик vlan 100, 200, 999

```
interface HQ-SRV
 ip mtu 1500
 ip address 192.168.0.1/26
!
interface HQ-CLI
 ip mtu 1500
 ip address 192.168.0.65/28
!
interface HQ-MGMT
 ip mtu 1500
 ip address 192.168.0.81/29
!
```

Заходим на порт и создаем для каждой `vlan` свой `service-instance`.

1. Заходим в `port ge1`.
2. Создаем service-instance `service-instance ge1/vlan100`
3. Указываем инкапсуляцию для `100 vlan`
4. Чтобы кадры из этого интерфейса выходили с тегом задаем настройку `rewrite pop 1`
5. Привязываем сущьность интрефейса к порту `connect ip interface HQ-SRV`

Проделываем эти действия для vlan 200 и 999

```
port ge1
 mtu 9234
 service-instance ge1/vlan100
  encapsulation dot1q 100
  rewrite pop 1
  connect ip interface HQ-SRV
 service-instance ge1/vlan200
  encapsulation dot1q 200
  rewrite pop 1
  connect ip interface HQ-CLI
 service-instance ge1/vlan999
  encapsulation dot1q 999
  rewrite pop 1
  connect ip interface HQ-MGMT
```


Создаем GRE туннель

```
interface tunnel.1
 ip mtu 1400
 ip address 172.16.1.1/30
 ip tunnel 172.16.4.2 172.16.5.2 mode gre
```

Задаем маршрут по умолчанию в сторону ISP

```
ip route 0.0.0.0/0 172.16.4.1
```
## HQ-SRV

```
echo "TYPE=eth
DISABLED=no
NM_CONTROLLED=no
BOOTPROTO=static
CONFIG_IPv4=yes" > /etc/net/ifaces/ens192/options
```

```
echo 192.168.0.2/26 > /etc/net/ifaces/ens192/ipv4address
```

```
echo default via 192.168.0.1 > /etc/net/ifaces/ens192/ipv4route
```

```
systemctl restart network
```

## HQ-CLI
![image](https://github.com/user-attachments/assets/f34b9e68-c1a9-4f28-93ef-d588b3eac9b7)
![image](https://github.com/user-attachments/assets/019b130b-2251-4719-a461-b46ae3ea22f3)
![image](https://github.com/user-attachments/assets/3865e576-e667-4a92-a6a2-f5581e3dda88)
![image](https://github.com/user-attachments/assets/ecc16e26-8cf4-4c6a-8185-d4492dad9bd6)
Если все сработало, то в выводе `ip a` увидим следующее
![image](https://github.com/user-attachments/assets/8656855f-7c88-4055-b111-72dba4e8c575)

## BR-RTR

```
configure

interface gigabitethernet 1/0/1
  ip firewall disable
  ip address 172.16.5.2/28
  no shutdown
exit
interface gigabitethernet 1/0/2
  ip firewall disable
  ip address 192.168.1.1/27
  no shutdown
exit
tunnel gre 1
  ttl 16
  mtu 1400
  ip firewall disable
  local address 172.16.5.2
  remote address 172.16.4.2
  ip address 172.16.1.2/30
  enable
exit
ip route 0.0.0.0/0 172.16.5.1

commit
confirm
```
## BR-SRV

```
echo "TYPE=eth
DISABLED=no
NM_CONTROLLED=no
BOOTPROTO=static
CONFIG_IPv4=yes" > /etc/net/ifaces/ens192/options
```

```
echo 192.168.1.2/26 > /etc/net/ifaces/ens192/ipv4address
```

```
echo default via 192.168.1.1 > /etc/net/ifaces/ens192/ipv4route
```

```
echo nameserver 192.168.0.2 > /etc/net/ifaces/ens192/resolv.conf
```

```
systemctl restart network
```

```
ip address
```
# Задание 3. Создание локальных учетных записей.
## HQ-SRV и BR-SRV

```
useradd -m -u 1010 sshuser
passwd sshuser
```
## HQ-RTR (EcoRouter)
```
conf t
username net_admin
password P@ssw0rd
role admin
activate
```
## BR-RTR (Eltex - vESR)

```
configure
username net_admin
password P@ssw0rd
privilege 15
end
!
commit
confirm
!
```
# Задание 5. Настройка безопасного удаленного доступа на серверах HQ-SRV и BR-SRV.
# Настройка SSH на HQ-SRV и BR-SRV
Делаем на всякий случай бэкап конфига:
```
cd /etc/openssh
cp -a sshd_config sshd_config.bak
```
Редактируем файл конфигурации SSH сервера
```
nano sshd_config
```
Изменяем следующие параметры. Не забываем их раскоментировать. Если какой-то параметр не находиться, то просто добавьте его сами
```
Port 2024
MaxAuthTries 3
Banner /etc/openssh/banner
AllowUsers sshuser
```
Создаем файл с баннером
```
nano /etc/openssh/banner
```
Вставляем в него следующие содержимое
```
WARNING!                     
```
Перезагружаем `SSH`
```
systemctl restart sshd
```
## Проверка
![image](https://github.com/user-attachments/assets/42c68fa2-4c7f-4680-8668-636a7c9f66a3)
# Задание 6-7. Между офисами HQ и BR необходимо сконфигурировать IP туннель. Обеспечьте динамическую маршрутизацию: ресурсы одного офиса должны быть доступны из другого офиса. Для обеспечения динамической маршрутизации используйте link state протокол на ваше усмотрение.
# Настраиваем OSPF
## HQ-RTR
```
router ospf 1
 network 172.16.1.0 0.0.0.3 area 0.0.0.0
 network 192.168.0.0 0.0.0.255 area 0.0.0.0
!
interface tunnel.1
 ip ospf authentication message-digest
 ip ospf message-digest-key 1 md5 Demo2025
 ip ospf network point-to-point
```
## BR-RTR
```
key-chain ospfkey
  key 1
    key-string ascii-text Demo2025
  exit
exit
router ospf 1
  area 0.0.0.0
    enable
  exit
  enable
exit

interface gigabitethernet 1/0/2
  ip ospf instance 1
  ip ospf
exit
tunnel gre 1
  ip ospf instance 1
  ip ospf network point-to-point
  ip ospf authentication key-chain ospfkey
  ip ospf authentication algorithm md5
  ip ospf
exit

commit
confirm
```
### HQ-RTR

```
sh ip ospf neighbor 
sh ip ospf interface brief
sh ip route
```
### BR-RTR
```
sh ip ospf neighbor
sh ip route
```
```
sh ip ospf interface
```
# Задание 8, 10. Настройка динамической трансляции адресов. Настройка DNS для офисов HQ и BR.
# Настройка DNS с помощью bind
## HQ-SRV
Устанавливаем bind:
```
apt-get install bind bind-utils
```
Редактируем конфиг:
```
nano  /var/lib/bind/etc/options.conf
```
Изменяем следующие параметры
![image](https://github.com/user-attachments/assets/38a756b6-3a5c-42df-ac66-4c953e208a8f)
![image](https://github.com/user-attachments/assets/8bba4086-d734-4bac-8052-d3a8be149eb7)
Проверяем на ошибки
```
named-checkconf
```
Если появилась вот такая ошибка:
![image](https://github.com/user-attachments/assets/2a954e7b-ec68-4532-84aa-5b06ce785cec)

Нужно скофигураровать ключи с помощью `rndc-confgen`
![image](https://github.com/user-attachments/assets/c83df0c4-ea06-4e70-ade6-dddfd7db4f3a)


Редактируем файл `var/lib/bind/etc/rndc.key `
```
nano /var/lib/bind/etc/rndc.key 
```
Встравляем в него ключ, который получили при помощи `rndc-confgen`

![image](https://github.com/user-attachments/assets/3f5be9e1-9828-4b3a-9fb5-5e9c6c03d713)

Проверяем на ошибки
```
named-checkconf
```
Если ошибок нет, то запускаем `bind`
```
systemctl enable --now bind
```
Проверяем что `bind` работает
```
systemctl status bind
```
![image](https://github.com/user-attachments/assets/803f3c2f-0535-4a2b-846c-2589bc13ed92)

Редактируем `resolv.conf`

```
nano /etc/net/ifaces/ens192/resolv.conf 
```
```
search au-team.irpo
nameserver 127.0.0.1
nameserver 192.168.0.2
nameserver 77.88.8.8
```
Перезагружаем сеть
```
systemctl restart network
```
Проверяем
```
dig ya.ru
```
### Создаем зону прямого просмотра
```
nano  /var/lib/bind/etc/local.conf
```
```
zone "au-team.irpo" {
        type master;
        file "au-team.irpo.db";
};
```
Создаем копию файла-шаблона прямой зоны `/var/lib/bind/etc/zone/localdomain`
```
# cp /var/lib/bind/etc/zone/localdomain  /var/lib/bind/etc/zone/au-team.irpo.db
```
Задаем права на файл
```
chown named. /var/lib/bind/etc/zone/au-team.irpo.db

chmod 600 /var/lib/bind/etc/zone/au-team.irpo.db
```
Открываем для редактирования

```
nano /var/lib/bind/etc/zone/au-team.irpo.db
```
```
$TTL    1D
@       IN      SOA     au-team.irpo. root.au-team.irpo. (
                                2024102200      ; serial
                                12H             ; refresh
                                1H              ; retry
                                1W              ; expire
                                1H              ; ncache
                        )
        IN      NS      au-team.irpo.
        IN      A       192.168.0.2
hq-rtr  IN      A       192.168.0.1
br-rtr  IN      A       192.168.1.1
hq-srv  IN      A       192.168.0.2
hq-cli  IN      A       192.168.0.66
br-srv  IN      A       192.168.1.2
moodle  IN      CNAME   hq-rtr
wiki    IN      CNAME   hq-rtr
```

Проверяем, что зона настроена Предварительно

```
named-checkconf -z
```
Перезагружаем `bind`
```
systemctl restart bind
```
Проверяем
```
dig hq-srv.au-team.irpo
```
### Создаем зону обратного просмотра и PTR записи

```
nano  /var/lib/bind/etc/local.conf
```
```
zone "0.168.192.in-addr.arpa" {
        type master;
        file "au-team.irpo_rev.db";
};
```

Копируем шаблон файла

```
cp /var/lib/bind/etc/zone/{127.in-addr.arpa,au-team.irpo_rev.db}
```

Задаем права на файл
```
chown named. /var/lib/bind/etc/zone/au-team.irpo_rev.db

chmod 600 /var/lib/bind/etc/zone/au-team.irpo_rev.db
```
Открываем для редактирования

```
nano /var/lib/bind/etc/zone/au-team.irpo_rev.db
```
Вставляем в него следующее содержимое.

```
$TTL    1D
@       IN      SOA     au-team.irpo. root.au-team.irpo. (
                                2024102200      ; serial
                                12H             ; refresh
                                1H              ; retry
                                1W              ; expire
                                1H              ; ncache
                        )
        IN      NS      au-team.irpo.
1       IN      PTR     hq-rtr.au-team.irpo.
2       IN      PTR     hq-srv.au-team.irpo.
66      IN      PTR     hq-cli.au-team.irpo.
```

Проверяем

```
named-checkconf -z
```
Перезагружаем `bind`

```
systemctl restart bind
```
Проверяем
```
dig -x 192.168.0.2
```
### Комплекская проверка с HQ-CLI
![image](https://github.com/user-attachments/assets/0eb7c2dd-789a-45b6-93f8-6d81ba14d2dc)

# Задание 9. Настройка протокола динамической конфигурации хостов.
ip pool HQ-NET200 1
 range 192.168.0.66-192.168.0.70
!
dhcp-server 1
 lease 86400
 mask 255.255.255.0
 pool HQ-NET200 1
  dns 192.168.0.2
  domain-name au-team.irpo
  gateway 192.168.0.65
  mask 255.255.255.240
!
interface HQ-CLI
 dhcp-server 1
!
# Задание 11. Настройте часовой пояс на всех устройствах, согласно месту проведения экзамена
# Настройка часового пояса
## HQ-SRV, HQ-CLI, BR-SRV
Проверяем какой часовой пояс установлен
```
timedatectl status
```
Если отличается, то устанавливаем
```
timedatectl set-timezone Asia/Yekaterinburg
```
## HQ-RTR (EcoRouter)
```
conf t
ntp timezone utc+5
```
Проверяем:
```
show ntp timezone
```
## BR-RTR (Eltex)
```
configure
clock timezone gmt +5
end
commit
confirm
```
Проверяем:
```
show date
```


