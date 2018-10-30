# Установка docker-engine Linux

Author: Evgeniy Krasnukhin(e.krasnukhin@cft.ru)

~~~
[root@service-bs-02 ~]# yum install docker-engine
Loaded plugins: ulninfo                                    
Resolving Dependencies                                     
--> Running transaction check                              
---> Package docker-engine.x86_64 0:17.06.2.ol-1.0.1.el7 will be installed                                            
--> Processing Dependency: container-selinux >= 2.9 for package: docker-engine-17.06.2.ol-1.0.1.el7.x86_64            
--> Processing Dependency: libltdl.so.7()(64bit) for package: docker-engine-17.06.2.ol-1.0.1.el7.x86_64               
--> Running transaction check                              
---> Package container-selinux.noarch 2:2.21-1.el7 will be installed                                                  
---> Package libtool-ltdl.x86_64 0:2.4.2-22.el7_3 will be installed                                                   
--> Finished Dependency Resolution                         

Dependencies Resolved                                      

================================================================================                                      
 Package             Arch     Version                  Repository          Size                                       
================================================================================                                      
Installing:                                                
 docker-engine       x86_64   17.06.2.ol-1.0.1.el7     local_ol7_addons    21 M                                       
Installing for dependencies:                               
 container-selinux   noarch   2:2.21-1.el7             local_ol7_addons    28 k                                       
 libtool-ltdl        x86_64   2.4.2-22.el7_3           local_ol7_latest    48 k                                       

Transaction Summary                                        
================================================================================                                      
Install  1 Package (+2 Dependent packages)                 

Total download size: 21 M                                  
Installed size: 74 M                                       
Is this ok [y/d/N]: Downloading packages:                  
--------------------------------------------------------------------------------                                      
Total                                               45 MB/s |  21 MB  00:00                                           
Running transaction check                                  
Running transaction test                                   
Transaction test succeeded                                 
Running transaction                                        
  Installing : 2:container-selinux-2.21-1.el7.noarch                        1/3                                       
setsebool:  SELinux is disabled.                           
  Installing : libtool-ltdl-2.4.2-22.el7_3.x86_64                           2/3                                       
  Installing : docker-engine-17.06.2.ol-1.0.1.el7.x86_64                    3/3                                       
  Verifying  : libtool-ltdl-2.4.2-22.el7_3.x86_64                           1/3                                       
  Verifying  : docker-engine-17.06.2.ol-1.0.1.el7.x86_64                    2/3                                       
  Verifying  : 2:container-selinux-2.21-1.el7.noarch                        3/3                                       

Installed:                                                 
  docker-engine.x86_64 0:17.06.2.ol-1.0.1.el7                                                                         

Dependency Installed:                                      
  container-selinux.noarch 2:2.21-1.el7   libtool-ltdl.x86_64 0:2.4.2-22.el7_3                                        

Complete!
[root@service-bs-02 ~]# systemctl enable docker.service    
Created symlink from /etc/systemd/system/multi-user.target.wants/docker.service to /usr/lib/systemd/system/docker.service.
[root@service-bs-02 ~]# systemctl start docker.service
[root@service-bs-02 ~]# systemctl status docker.service
● docker.service - Docker Application Container Engine
   Loaded: loaded (/usr/lib/systemd/system/docker.service; enabled; vendor preset: disabled)
  Drop-In: /etc/systemd/system/docker.service.d
           └─docker-sysconfig.conf
   Active: active (running) since Thu 2018-05-31 08:46:48 +07; 6s ago
     Docs: https://docs.docker.com
 Main PID: 30172 (dockerd)
   Memory: 15.1M
   CGroup: /system.slice/docker.service
           ├─30172 /usr/bin/dockerd --selinux-enabled
           └─30183 docker-containerd -l unix:///var/run/docker/libcontainerd/docker-containerd.sock --metrics-interval=0 --start-timeout 2m --state-dir /var/run/docker/libcontainerd/containerd --shim docker-containerd-shim --runtime d...

May 31 08:46:48 service-bs-02 dockerd[30172]: time="2018-05-31T08:46:48.120470775+07:00" level=info msg="Graph migration to content-addressability took 0.00 seconds"
May 31 08:46:48 service-bs-02 dockerd[30172]: time="2018-05-31T08:46:48.120777734+07:00" level=warning msg="mountpoint for pids not found"
May 31 08:46:48 service-bs-02 dockerd[30172]: time="2018-05-31T08:46:48.120963375+07:00" level=info msg="Loading containers: start."
May 31 08:46:48 service-bs-02 dockerd[30172]: time="2018-05-31T08:46:48.493771129+07:00" level=info msg="Default bridge (docker0) is assigned with an IP address 172.17.0.0/16. Daemon option --bip can be used to set a ...erred IP address"
May 31 08:46:48 service-bs-02 dockerd[30172]: time="2018-05-31T08:46:48.570615269+07:00" level=info msg="Loading containers: done."
May 31 08:46:48 service-bs-02 dockerd[30172]: time="2018-05-31T08:46:48.575552439+07:00" level=warning msg="Not using native diff for overlay2: opaque flag erroneously copied up, consider update to kernel 4.8 or later to fix"
May 31 08:46:48 service-bs-02 dockerd[30172]: time="2018-05-31T08:46:48.624368443+07:00" level=info msg="Daemon has completed initialization"
May 31 08:46:48 service-bs-02 dockerd[30172]: time="2018-05-31T08:46:48.624475574+07:00" level=info msg="Docker daemon" commit=d02b7ab graphdriver=overlay2 version=17.06.2-ol
May 31 08:46:48 service-bs-02 dockerd[30172]: time="2018-05-31T08:46:48.635607624+07:00" level=info msg="API listen on /var/run/docker.sock"
May 31 08:46:48 service-bs-02 systemd[1]: Started Docker Application Container Engine.
Hint: Some lines were ellipsized, use -l to show in full.
[root@service-bs-02 ~]# ls -l /var/lib/docker/
total 52
drwx------ 2 root root  4096 May 31 08:46 containers
drwx------ 3 root root  4096 May 31 08:46 image
drwx------ 2 root root 16384 May 30 15:37 lost+found
drwxr-x--- 3 root root  4096 May 31 08:46 network
drwx------ 3 root root  4096 May 31 08:46 overlay2
drwx------ 4 root root  4096 May 31 08:46 plugins
drwx------ 2 root root  4096 May 31 08:46 swarm
drwx------ 2 root root  4096 May 31 08:46 tmp
drwx------ 2 root root  4096 May 31 08:46 trust
drwx------ 2 root root  4096 May 31 08:46 volumes
~~~

# Установка docker-compose

[Install Compose on Linux systems](https://docs.docker.com/compose/install/#install-compose)
~~~
[root@service-bs-01 ~]# curl -L https://github.com/docker/compose/releases/download/1.21.2/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   617    0   617    0     0    629      0 --:--:-- --:--:-- --:--:--   629
100 10.3M  100 10.3M    0     0   321k      0  0:00:32  0:00:32 --:--:--  367k
[root@service-bs-01 ~]# ls -l /usr/local/bin/
total 10608
-rw-r--r-- 1 root root 10858808 May 31 10:30 docker-compose
[root@service-bs-01 ~]# chmod +x /usr/local/bin/docker-compose
[root@service-bs-01 ~]# ls -l /usr/local/bin/
total 10608
-rwxr-xr-x 1 root root 10858808 May 31 10:30 docker-compose
[root@service-bs-01 ~]# docker-compose --version
docker-compose version 1.21.2, build a133471
~~~

Добавить в .bash_profile
~~~
PATH=$PATH:$HOME/bin:/usr/local/bin
~~~

# Настройка прокси

Для того, чтобы пользоваться прокси нужно еще прописать в конфиг
@/etc/systemd/system/docker.service.d/http-proxy.conf@
~~~
[Service]
Environment="HTTP_PROXY=http://localhost:3128"
Environment="HTTPS_PROXY=http://localhost:3128"
Environment="no_proxy=localhost,.ftc.ru,127.0.0.1,172.17.0.1"
~~~

Перезапустить docker
~~~
#systemctl daemon-reload
#systemctl restart docker
~~~

Поднять тунель
~~~
export http_proxy=http://localhost:3128
export https_proxy=http://localhost:3128
export no_proxy='localhost,.ftc.ru,127.0.0.1'
ssh -4 -o ExitOnForwardFailure=yes -fNL 3128:localhost:3128 oracle@bftops
~~~