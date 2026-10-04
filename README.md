# demotestKait20


### 1. Настройка интерфейсов на ALT

Для конфигурации IPv4 на устройствах будут отредактированы файлы options и созданы файлы ipv4address.

**/etc/net/ifaces/<ИМЯ_ИНТЕРФЕЙСА>/options**

Должны быть заданы хотя бы два основных параметра. Параметр `TYPE=eth` указывает на тип интерфейса ethernet, параметр `BOOTPROTO=static` означает, что настройка статического IP-адреса будет взята из файла ipv4address:

```
[root@hq-srv ~]# cat /etc/net/ifaces/ens19/options
BOOTPROTO=static
TYPE=eth
CONFIG_WIRELESS=no
SYSTEMD_BOOTPROTO=static
CONFIG_IPV4=yes
DISABLED=no
NM_CONTROLLED=no
SYSTEMD_CONTROLLED=no
[root@hq-srv ~]#
```

**Правила настройки:**
`vim /etc/net/ifaces/<ИМЯ_ИНТЕРФЕЙСА>/ipv4address <IP-адрес>/<Префикс>`

```
[root@hq-srv ~]# cat /etc/net/ifaces/ens19/ipv4address
192.168.100.1/26
```

Для применения настроек необходимо перезагрузить службу network командой `systemctl restart network`.

Проверка IP-адреса осуществляется командой `ip a`.

---

### 2. DHCP-server ALT

**Установка**

`# apt-get install dhcp-server`

**Настройка**

1. Настройте статический IP-адрес.
2. Настройте DHCP-сервер.

`# vim /etc/dhcp/dhcpd.conf`

```
ddns-update-style none;

subnet 10.0.0.0 netmask 255.255.255.0 { #сеть и маска подсети
option routers 10.0.0.1; #адрес маршрутизатора
option subnet-mask 255.255.255.0; #маска подсети
option nis-domain "domain.org"; #NIS-домен
option domain-name "domain.org"; #домен
option domain-name-servers 195.54.2.1; #DNS-сервера для клиентов
range dynamic-bootp 10.0.0.128 10.0.0.250; #диапазон DHCP-подсети

#ручное резервирование адресов
host boss
{
hardware ethernet 00:1C:C0:45:27:14;
fixed-address 10.0.0.2;
}
host pavel
{
hardware ethernet 00:1C:C0:45:28:2B;
fixed-address 10.0.0.3;
}

#стандартное и максимальное время аренды (в секундах)
#6 часов
default-lease-time 21600;
#12 часов
max-lease-time 43200;
}
```

Укажите сетевой интерфейс, через который будет работать DHCP-сервер:

`# vim /etc/sysconfig/dhcpd`

`DHCPDARGS=eth0`

3. Для того чтобы DHCP-сервер автоматически запускался:

`# chkconfig dhcpd on`
`# service dhcpd start`

---

### 3. Перевод в режим роутера и включение NAT

**Как делать?**

Для того чтобы устройство могло пересылать пакеты с интерфейса на интерфейс, необходимо включить пересылку пакетов (маршрутизацию/forwarding). Для этого следует в конфигурационном файле

`/etc/net/sysctl.conf`

в параметре `net.ipv4.ip_forward = 0` заменить значение с 0 на 1.

Для применения настроек необходимо перезагрузить службу network, командой `systemctl restart network`.

Для динамической сетевой трансляции можно использовать iptables. Необходимо установить, выполнить установку можно с помощью команды `apt-get install iptables`, предварительно обновив список пакетов с помощью команды `apt-get update`.

Реализация сетевой трансляции адресов с помощью iptables можно выполнить одной командой:

`iptables --t nat --A POSTROUTING --o <ИМЯ_ВНЕШНЕГО_ИНТЕРФЕЙСА> --j MASQUERADE`

где `<ИМЯ_ВНЕШНЕГО_ИНТЕРФЕЙСА>` — внешний интерфейс, смотрящий в сторону магистрального провайдера.

После сохраните все изменения:

`iptables-save >> /etc/sysconfig/iptables`

Далее необходимо запустить и добавить в автозагрузку службу iptables:

`systemctl enable --now iptables`

**Как проверить?**

Проверить включение функции пересылки пакетов:

`sysctl net.ipv4.ip_forward`

Проверить наличие правила в таблице nat в цепочке POSTROUTING:

`iptables --t nat --L --n --v`

---

### 4. Chrony на ALT

**Работа со временем**

`date` – посмотреть время и дату
`date 09011002022` – установить время и дату (09 – месяц, 01 – день, 10 – часы, 00 – минуты, 2022 - год)
`timedatectl list-timezones` – посмотреть список доступных временных поясов
`timedatectl set-timezone name_zone` – установить временную зону

**Настройка первичного и вторичного серверов времени:**

На первом сервере устанавливаем службу Chrony: `apt-get install -y chrony`
После установки редактируем конфигурационный файл: `vim /etc/chrony.conf`

Заполняем строки:
`pool <АДРЕС СЕРВЕРА С КОТОРОГО БЕРЕМ ВРЕМЯ> iburst`
`allow <СЕТЬ КОТОРАЯ ИМЕЕТ ПРАВО ЗАПРАШИВАТЬ У НАС ВРЕМЯ>`
`stratum <НОМЕР>`

Пример содержимого файла:
```
Use public servers from the pool.ntp.org project. Please consider joining the pool (https://www.pool.ntp.org/join.html).
pool pool.ntp.org iburst
Record the rate at which the system clock gains/losses time.
driftfile /var/lib/chrony/drift
Allow the system clock to be stepped in the first three updates if its offset is larger than 1 second.
makestep 1.0 3
Enable kernel synchronization of the real-time clock (RTC).
rtcsync
Enable hardware timestamping on all interfaces that support it.
#hwtimestamp
Increase the minimum number of selectable sources required to adjust the system clock.
#minsources 2
Allow NTP client access from local network.
#allow 192.168.0.0/16
Serve time even if not synchronized to a time source.
#local stratum 10
```
(Далее в файле следует длинная строка из повторяющихся символов, которая, вероятно, является артефактом копирования).

---

### 5. Создание локальных учетных записей

**Задание 1.** Создайте пользователя sshuser на сервере.

**Подробное описание пункта задания:**

1. Пароль пользователя sshuser должен быть P@ssw0rd.
2. Идентификатор пользователя 1010.
3. Пользователь sshuser должен иметь возможность запускать sudo без дополнительной аутентификации.
4. Пользователь должен принадлежать группе wheel.

Создать пользователя с явным указанием UID можно с помощью команды:

`useradd <ИМЯ_ПОЛЬЗОВАТЕЛЯ> --u <UID>`

Задать пароль пользователю можно с помощью утилиты passwd:

`passwd <ИМЯ_ПОЛЬЗОВАТЕЛЯ>`

В результате запуска утилиты passwd необходимо будет задать пароль, а затем подтвердить заданный пароль.

Для редактирования sudo можно воспользоваться командой `visudo` или явно открыть файл `/etc/sudoers` в текстовом редакторе vim.

В файле следует найти и раскомментировать строку:

`WHEEL_USERS ALL=(ALL:ALL) NOPASSWD: ALL`

Добавить пользователя в группу можно с помощью команды:

`gpasswd --a <ИМЯ_ПОЛЬЗОВАТЕЛЯ> <ИМЯ_ГРУППЫ>`

---

### 6. Настройка безопасного удаленного доступа по протоколу SSH

**Подробное описание пункта задания:**

Настройка безопасного удаленного доступа на сервер:
• Для подключения используйте порт 2026;
• Разрешите подключения только пользователю sshuser;
• Ограничьте количество попыток входа до двух;
• Настройте баннер Authorized access only.

**Как делать?**

Редактируем конфигурационный файл openssh, расположенный по пути `/etc/openssh/sshd_config`, текстовым редактором vim.

Находим следующие параметры и приводим их к следующему виду:

`Port 2026` — порт, на котором следует ожидать запросы на соединение. Значение по умолчанию — 22;

`AllowUsers sshuser` — список имен пользователей через пробел. Если параметр определен, регистрация в системе будет разрешена только пользователям, чьи имена соответствуют одному из шаблонов;

`MaxAuthTries 2` — ограничение на число попыток идентифицировать себя в течение одного соединения;

`PasswordAuthentication yes` — допускать аутентификацию по паролю;

`Banner /etc/openssh/banner` — содержимое указанного файла будет отправлено удаленному пользователю прежде, чем будет разрешена аутентификация.

Редактируем баннер (файл) по пути `/etc/openssh/banner` текстовым редактором vim и добавляем в него следующее содержимое:

`Authorized access only.`

Для применения всех изменений необходимо перезапустить службу sshd, для этого можно использовать команду:

`systemctl restart sshd`

Проверить подключение с клиента:

`ssh <sshuser@192.168.0.1> -p 2026`

---

### 7. Настройка часовых поясов

**Подробное описание пункта задания:** Настройте часовой пояс на всех устройствах согласно месту вашего нахождения.

**Где выполнять?** На всех машинах.

**Как делать?** На устройствах с ОС «Альт» необходимо выполнить следующую команду:

`timedatectl set-timezone <ЧАСОВАЯ_ЗОНА>`

Например: `timedatectl set-timezone Europe/Moscow`

**Как проверить?** На устройствах с ОС «Альт» воспользоваться утилитой timedatectl:

```
[root@hq-srv ~]# timedatectl
Local time: Tue 2025-04-08 09:46:45 MSK
Universal time: Tue 2025-04-08 06:46:45 UTC
RTC time: Tue 2025-04-08 06:46:45
Time zone: Europe/Moscow (MSK, +0300)
System clock synchronized: yes
NTP service: active
RTC in local TZ: no
[root@hq-srv ~]#
```

---

### 8. Настройка DNS на ALT

**Подробное описание пункта задания:**

• Сервер должен обеспечивать разрешение имен в сетевые адреса устройств и обратно в соответствии с таблицей 2.
• В качестве DNS-сервера пересылки используйте любой общедоступный DNS сервер.

**Таблица 2:**
| Устройство | Зона | Запись | Тип | IP-адрес |
| :--- | :--- | :--- | :--- | :--- |
| HQ-RTR | au-team.irpo | hq-rtr.au-team.irpo | A | 192.168.100.62 |
| BR-RTR | au-team.irpo | br-rtr.au-team.irpo | A | 192.168.200.30 |
| HQ-SRV | au-team.irpo | hq-srv.au-team.irpo | A | 192.168.100.1 |
| HQ-CLI | ru | hq-cli.ru | A | 172.16.0.1 |
| BR-SRV | ru | br-srv.ru | A | 159.159.159.159 |
| HQ-RTR | ru | moodle.ru | CNAME | hq-rtr.au-team.irpo |

**Как делать?**

Для установки и дальнейшей настройки DNS-сервера необходимо выполнить установку пакета BIND, сделать это можно при помощи команды:

`apt-get update && apt-get install bind -y`

Далее выполняется редактирование конфигурационного файла `/var/lib/bind/etc/options.conf` согласно скриншоту с использованием текстового редактора vim:

```
listen-on { 192.168.100.1; };
listen-on-v6 { none; };

/*
 * If the forward directive is set to "only", the server will only
 * query the forwarders.
 */
//forward only;
forwarders { 77.88.8.8; };

/*
 * Specifies which hosts are allowed to ask ordinary questions.
 */
allow-query { any; };

/*
 * This lets "allow-query" be used to specify the default zone access
 * level rather than having to have every zone override the global
 * value. "allow-query-cache" can be set at both the options and view
 * levels. If "allow-query-cache" is not set then "allow-recursion" is
 * used if set, otherwise "allow-query" is used if set unless
 * "recursion no;" is set in which case "none;" is used, otherwise the
 */
```

`listen-on` параметр определяет адреса и порты, на которых DNS-сервер будет слушать запросы (УКАЗЫВАЕМ СВОЙ IP-АДРЕС СМОТРЯЩИЙ ВО ВНУТРЕННЮЮ СЕТЬ).

В параметре `forwarders` указываются сервера, куда будут перенаправляться запросы, о которых нет информации в локальной зоне.

`allow-query` — IP-адреса и подсети, от которых будут обрабатываться запросы.

Далее необходимо добавить зоны прямого просмотра в файл `/var/lib/bind/etc/rfc1912.conf`, используя текстовый редактор:

```
zone "au-team.irpo" {
type master;
file "au-team.irpo";
};
```

**АНАЛОГИЧНО ДОБАВИТЬ ЗОНУ RU**

Необходимо перейти в директорию `/var/lib/bind/etc/zone` и путем копирования создать файлы зон:

```
[root@hq-srv ~]# cd /var/lib/bind/etc/zone/
[root@hq-srv zone]# cp empty au-team.irpo
```

Необходимо сконфигурировать файл `au-team.irpo`, который является прямой зоной, следующим образом:

```
$TTL 1D
@ IN SOA au-team.irpo. root.au-team.irpo. (
2025020600 ; serial
12H ; refresh
1H ; retry
1W ; expire
1H ) ; ncache
IN NS au-team.irpo.
IN A 192.168.100.1
hq-rtr IN A 192.168.100.62
hq-rtr IN A 192.168.100.70
hq-rtr IN A 192.168.100.86
br-rtr IN A 192.168.200.30
hq-srv IN A 192.168.100.1
```

**АНАЛОГИЧНО ЗАПОЛНИТЬ ЗОНУ RU**

Для DNS-сервера важно обеспечить непрерывный аптайм, не допуская даже минутных простоев. Если вы попытаетесь перезапустить systemd-юнит обычной командой systemctl, а в конфигурации будут ошибки, то BIND не запустится. Чтобы избежать столь неприятных последствий, надо правильно настроить утилиту rndc, которая позволяет обойти эти сложности. После того как конфигурация зон будет завершена, для корректной работы службы bind необходимо выполнить команду:

`rndc-confgen > /etc/bind/rndc.key`

Затем выполнить команду:

`sed -i '6,$d' rndc.key`

```
[root@hq-srv zone]# rndc-confgen > /var/lib/bind/etc/rndc.key
[root@hq-srv zone]# sed -i '6,$d' /var/lib/bind/etc/rndc.key
[root@hq-srv zone]# cat /var/lib/bind/etc/rndc.key
# Start of rndc.conf
key "rndc-key" {
    algorithm hmac-sha256;
    secret "rgTaoTWhtkH2/jShpZ4CoY1E54BLlH+G5s/aQIBU=";
};
[root@hq-srv zone]#
```

Перед запуском службы остается поменять группу у файлов зон, которые были созданы ранее, на named, а также проверить конфигурационные файлы и файлы зон командами

`named-checkconf`
`named-checkconf -z`

```
[root@hq-srv etc]# chgrp -R named /var/lib/bind/etc/zone/
[root@hq-srv etc]# named-checkconf
[root@hq-srv etc]# named-checkconf -z
zone localhost/IN: loaded serial 2025020600
zone localdomain/IN: loaded serial 2025020600
zone 127.in-addr.arpa/IN: loaded serial 2025020600
zone 0.in-addr.arpa/IN: loaded serial 2025020600
zone 255.in-addr.arpa/IN: loaded serial 2025020600
zone au-team.irpo/IN: loaded serial 2025020600
zone 100.168.192.in-addr.arpa/IN: loaded serial 2025020600
[root@hq-srv etc]#
```

После этого можно запустить службу bind командой

`systemctl enable --now bind.service`

Проверить статус службы можно при помощи команды

`systemctl status bind`

**Как проверить?**

Проверить доступ в сеть Интернет средствами утилиты ping, учитывая, что в качестве DNS-сервера используется HQ-SRV.

---

### 9. Настройка веб-сервера nginx

**Задание 1.** Разместите на сервере 2 сайта.

**Подробное описание пункта задания:**

1. Первый сайт должен быть сайтом проверкой и открываться по IP.
2. 2-й сайт необходимо перенести из папки koni и разместить на сервере.
3. 2-й сайт должен быть доступен по доменному имени `koni.au-team.irpo`.

**Установка**

`apt-get update`
`apt-get install nginx`

**Включение и запуск службы**

`systemctl enable --now nginx.service`

**Каталоги и файлы Nginx**

• `/etc/nginx/` — домашний каталог
• `/etc/nginx/nginx.conf` — каталог настроек сервера
• `/etc/nginx/sites-available.d/` — каталог настроек сайта
• `/etc/nginx/sites-enabled.d/` — каталог запуска сайта
• `/var/log/nginx` — каталог журналов

**Настройка**

• Создадим конфигурационный файл сайта:

`cd /etc/nginx/sites-available.d/`
`vim site.conf`

• Запишем в файл site.conf следующие строки:

```
server {
listen <ip>:80;
server_name <имя сервера>;
root /var/www/site/;
index index.html;
}
```

• Здесь указано:
o `Listen` – ip и порт веб сервера
o `server_name` – имя сайта
o `root` - путь к каталогу с сайтом на сервере
o `index` – имя файла главной страницы

• Создадим символическую ссылку для site.conf в рабочем каталоге `/etc/nginx/sites-enabled.d/`:

`ln -s /etc/nginx/sites-available.d/site.conf /etc/nginx/sites-enabled.d/site.conf`

**Наполнение сайта**

Создадим тестовый файл в каталоге сайта - `/var/www/html`:

`vim /var/www/site/index.html`

Заполним его приблизительно следующим содержимым:

```
<html><body><h1>It works! Nginx</h1></body></html>
```

**Проверка синтаксиса и перезапуск службы**

Проверим синтаксис в файлах nginx:

`nginx -t`

Перезапустим службу:

`systemctl restart nginx`

**Копирование файлов с основной машины на виртуальную**

Установите программу WINSCP с сайта `https://winscp.net/eng/download.php`.

Создайте новую вкладку и введите свои данные (скриншот окна входа WinSCP с настройками протокола SFTP, IP-адреса, порта и учетных данных).

Перетащите папку koni с основного на виртуальный хост (скриншот интерфейса WinSCP с процессом перетаскивания папки).
