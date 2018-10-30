# ssh туннель

Author: Evgeniy Krasnukhin(e.krasnukhin@cft.ru)

Ситуация:

С хоста **source** нет доступа на хост **destination** по порту NN. А очень хочется быстро законнектиться разок, не больше.

НО! с хоста **middle** доступ на хост **destination** по порту NN есть.

А также на хосте **middle** есть **ssh** сервер, и он доступен с хоста **source**.

Можно воспользоваться ssh туннелем.

~~~
[user@sourcehost]$ ssh -4 -f -N -L XXXX:destination:NN middle
~~~

XXXX - произвольный свободный порт на хосте source.

-4 - только ipv4. Иначе при отключенном ipv6 на хосте и с настройками ssh клиента по умолчанию появится предупреждение, что не удалось забиндить порт на ipv6.

-f - отправить в бэкграунд и отвязаться от терминала. родительский PID = 1 (init, systemd).

-N - не выполнять команду по ssh. Только перенаправлять порт.

-L - настройка форвардинга.

В случае -f -N не забудьте завершить туннель kill'ом (SIGTERM).

Чтобы не забыть, можно сделать временный туннель:
~~~
[user@sourcehost]$ ssh -4 -f -L XXXX:destination:NN middle sleep 60
~~~

Ключа -N нет. На хосте middle будет выполняться sleep 60, по завершению туннель закроется. Если по истечении 60 секунд через туннель есть сессия, то закроется сразу после завершения сессии.

Теперь смотрим netstat:

~~~
[user@sourcehost]$ netstat -tunlp
tcp        0      0 127.0.0.1:XXXX              0.0.0.0:*                   LISTEN      9611/ssh
...
[user@sourcehost]$ ps -ef | grep ssh
user    9611     1  0 15:50 ?        00:00:00 ssh -4 -f -L XXXX:destination:NN middle sleep 60
...
~~~

Теперь c хоста source коннектимся на **localhost:XXXX**.

Например, если NN=22, а XXXX=2222:

~~~
[user@sourcehost]$ ssh -p 2222 localhost
~~~

или scp:

~~~
[user@sourcehost]$ scp -P 2222 path1 someuser@localhost:/path2
~~~

Можно забиндить не на локалхост, явно указав:

~~~
[user@sourcehost]$ ssh -4 -f -L YY.YY.YY.YY:XXXX:destination:NN middle sleep 60
~~~
YY.YY.YY.YY - адрес интерфейса на хосте source.

# Реверсивный туннель на конкретном примере

Ситуация: Необходимо установить дистрибы и патчи на хосте в DMZ rnd-db-01.  Доступа к oraclenas  из DMZ нету.

У нас есть доступ в bftops. Можно на нем построить реверсивный туннель.

На bftops:
~~~
[oracle@bftops ~]$ ssh -NR 2049:oraclenas.ftc.ru:2049 rnd-db-01
~~~

На rnd-db-01:
~~~
[root@rnd-db-01 ~]# systemctl stop autofs
[root@rnd-db-01 ~]# netstat -tunpl                                                                                                                      
Active Internet connections (only servers)                                                                                 
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name                                
...                              
tcp        0      0 127.0.0.1:2049          0.0.0.0:*               LISTEN      25305/sshd: oracle                              
...
[root@rnd-db-01 ~]# mkdir -p /net/oraclenas/distrib  
[root@rnd-db-01 ~]# mount.nfs4 -o tcp localhost:/distrib /net/oraclenas/distrib
[root@rnd-db-01 ~]# ls -l /net/oraclenas/distrib                  
total 140                                                  
dr-xr-xr-x  2 nobody nobody  2048 May  4 14:38 Agile       
dr-xr-xr-x  3 nobody nobody  2048 Jun  7  2017 Apache      
dr-xr-xr-x  2 nobody nobody  2048 Jul 28  2017 Atlassian   
dr-xr-xr-x 13 nobody nobody  2048 Dec 19 16:53 CFT_Admin_WorkPlace                                                    
dr-xr-xr-x  4 nobody nobody  2048 Jun 24  2014 CI          
dr-xr-xr-x  2 nobody nobody  2048 Jun 24  2014 Citrix_(NetScaler)                                                     
dr-xr-xr-x  4 nobody nobody  2048 Jun 24  2014 DBA_Tools   
dr-xr-xr-x 71 nobody nobody 16384 Feb 17  2017 Education
dr-xr-xr-x 37 nobody nobody  8192 Jun 24  2014 Exam_(Test)
dr-xr-xr-x 10 nobody nobody  2048 Feb 13  2015 i3
dr-xr-xr-x 14 nobody nobody  2048 Jun 21  2016 IBM
dr-xr-xr-x 10 nobody nobody  4096 Nov 12  2015 Java
dr-xr-xr-x  4 nobody nobody  4096 Jun  2  2017 JBOSS.org
dr-xr-xr-x  4 nobody nobody  2048 Jun 24  2014 Language
dr-xr-xr-x  3 nobody nobody  2048 Jul  4  2016 MGW
dr-xr-xr-x  2 nobody nobody  2048 Feb 20  2015 Office
dr-xr-xr-x 13 nobody nobody  4096 Feb 16 08:23 Oracle
~~~

После завершения работ:

Отмонтировать ресурс

`[root@rnd-db-01 ~]# umount -f /net/oraclenas/distrib`

И схлопнуть туннель на bftops, если он по-прежнему активный, то ctrl+c. Или kill -9 pid