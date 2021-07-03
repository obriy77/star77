Тестовое задание.

- Напиши демон, который раз в минуту выводит список ARP адресов из ARP кэша 
интерфейса eth0;
- Залей на GitHub и заполни 3-ю форму (на странице позиции).

Название проекта должно состоять из 4 букв латинского алфавита 
и от 1 до 5 цифр. Например ydhr77. Не указывать в имени 
репозитория или в файлах строчки ask42, 42, 
тестовое задание и прочие, чтобы другие кандидаты могли найти 
в поисковике этот репозиторий. Тестовые задания, которые нарушают 
эти правила, приниматься не будут.

=================================================================

                        Лабораторная среда:
Distributor ID: Ubuntu
Description:    Ubuntu 18.04.5 LTS
Release:        18.04
Codename:       bionic


arp --version
net-tools 2.10-alpha
+I18N +SELINUX
AF: (inet) +UNIX +INET +INET6 +IPX +AX25 +NETROM +X25 +ATALK +ECONET +ROSE -BLUETOOTH
HW: (ether) +ETHER +ARC +SLIP +PPP +TUNNEL -TR +AX25 +NETROM +X25 +FR +ROSE +ASH +SIT +FDDI +HIPPI +HDLC/LAPB +EUI64

Linux ubuntu1804.localdomain 4.15.0-147-generic 
#151-Ubuntu SMP Fri Jun 18 19:21:19 UTC 2021 x86_64 x86_64 x86_64 GNU/Linux


=================================================================

                  **Пишем своего демона для Linux**

1) Создаем по  пути **/home/user/output_list_ARP.sh** скрипт с содержимым:
 

#!/bin/bash

LOGFILE="/home/user/mylog_1.log"

while true
  do
    LAST="$(sudo arp)"
    echo "$LAST at $(date)" >> $LOGFILE
  if [ $# -lt 2 ]; then
    return 1 2>/dev/null
    exit 1
  fi
  done

==================================================================

2) Превращаем его в исполняемый файл и проверяем работу:

**chmod u+x /home/user/output_list_ARP.sh**

**/home/user/output_list_ARP.sh**

2.1) Создаем файл для логов:

**touch /home/user/mylog_1.log**

**tail /home/user/mylog_1.log**

==================================================================

3)Запускаем Cron Jobs, который раз в минуту выводит список ARP адресов из ARP кэша 
интерфейса eth0 
- устанавливаем необходимые параметры для работы Cron Jobs, а  именно редактируем файл командой в терминате:

$ crontab -e

...
# m  h  dom mon dow   command

# ┌───────────── minute (0 - 59) - m
# │ ┌───────────── hour (0 - 23) - h
# │ │ ┌───────────── day of the month (1 - 31) - dom
# │ │ │ ┌───────────── month (1 - 12) - mon
# │ │ │ │ ┌───────────── day of the week (0 - 6) (Sunday to Saturday; 
# │ │ │ │ │                                   7 is also Sunday on some systems) - dow
# │ │ │ │ │
# │ │ │ │ │
# * * * * *          command

  * * * * *          sh /home/user/output_arp.sh >> /home/user/mylog_1.log

...

3.1)Пример вывода результатов работы Cron Jobs:

$ tail mylog_1.log


_gateway                 ether   34:ce:00:65:2e:33   C                     eth0 at Sat Jul  3 18:50:01 MSK 2021
Address                  HWtype  HWaddress           Flags Mask            Iface
192.168.1.98             ether   48:e2:44:e1:74:85   C                     eth0
_gateway                 ether   34:ce:00:65:2e:33   C                     eth0 at Sat Jul  3 18:51:01 MSK 2021
Address                  HWtype  HWaddress           Flags Mask            Iface
192.168.1.98             ether   48:e2:44:e1:74:85   C                     eth0
_gateway                 ether   34:ce:00:65:2e:33   C                     eth0 at Sat Jul  3 18:52:01 MSK 2021
Address                  HWtype  HWaddress           Flags Mask            Iface
192.168.1.98             ether   48:e2:44:e1:74:85   C                     eth0
_gateway                 ether   34:ce:00:65:2e:33   C                     eth0 at Sat Jul  3 18:53:01 MSK 2021



_gateway                 ether   34:ce:00:65:2e:33   C                     eth0 at Sat Jul  3 18:59:01 MSK 2021
Address                  HWtype  HWaddress           Flags Mask            Iface
192.168.1.98             ether   48:e2:44:e1:74:85   C                     eth0
_gateway                 ether   34:ce:00:65:2e:33   C                     eth0 at Sat Jul  3 19:00:01 MSK 2021
Address                  HWtype  HWaddress           Flags Mask            Iface
192.168.1.98             ether   48:e2:44:e1:74:85   C                     eth0
_gateway                 ether   34:ce:00:65:2e:33   C                     eth0 at Sat Jul  3 19:01:01 MSK 2021
Address                  HWtype  HWaddress           Flags Mask            Iface
192.168.1.98             ether   48:e2:44:e1:74:85   C                     eth0
_gateway                 ether   34:ce:00:65:2e:33   C                     eth0 at Sat Jul  3 19:02:01 MSK 2021




3) Пишу в файл /etc/init.d/output_list_ARP скрипт для демона:

#!/bin/bash

# chkconfig: 2345 20 80
# description:checking cpu load
# Source function library.
. /etc/init.d/functions

case "$1" in
start)
 echo "$(date) service output_list_ARP started" >> /home/user/mylog_1.log
 sh /home/user/output_list_ARP.sh &
 echo $!>/var/run/output_list_ARP.pid
 ;;
stop)
 echo "$(date) service output_list_ARP stopped" >> /home/user/mylog_1.log
 kill 'cat /var/run/output_list_ARP.pid'
 rm /var/run/output_list_ARP.pid
 ;;
restart)
 $0 stop
 $0 start
 ;;
status)
 if [-e /var/run/output_list_ARP.pid]; then
  echo output_arp is running, pid='cat /var/run/output_list_ARP.pid'
 else
  echo output_list_ARP is NOT running
  exit 1
 fi
 ;;
*)
 echo "Usage: $0 {start|stop|status|restart}"
esac

exit 0


4) Делаю скрипт исполняемым и проверяю его работу:

**sudo chmod u+x /etc/init.d/output_list_ARP**

**sudo systemctl status output_list_ARP**

**sudo systemctl start output_list_ARP**

**sudo systemctl stop output_list_ARP**


=============================================================================
