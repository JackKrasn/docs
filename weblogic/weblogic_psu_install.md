# Установка PSU на Weblogic

Author: Konstantin Mankov(k.mankov@cft.ru)

**Посмотреть текущую версию JAVA, Weblogic и установленные патчи**

~~~
$ cd /u/app/oracle/Middleware/wlserver_10.3/server/bin 
$ . ./setWLSEnv.sh 
$ java -version

$ java -cp ../lib/weblogic.jar weblogic.version

$ cd /u/app/oracle/Middleware/utils/bsu
$ ./bsu.sh -report
~~~

**Остановить все процессы Weblogic**

**Создать резервную копию каталогов Middleware_Home, User_Projects, oraInventory**

~~~
$ cd /u/app/oracle
$ tar -czvf Middleware_20181009.tar.gz ./Middleware
$ tar -czvf user_projects_20181009.tar.gz ./user_projects
cd /u/app
$ tar -czvf OraInventory_20181009.tar.gz ./OraInventory
~~~

**Откатить установленный PSU, если имеется**

Stop all WebLogic Servers
Navigate to the /u/app/oracle/Middleware/utils/bsu directory.
Execute 

~~~
./bsu.sh -remove -patchlist=FMJJ -prod_dir=/u/app/oracle/Middleware/wlserver_10.3

Checking for conflicts........
No conflict(s) detected

Removing Patch ID: FMJJ..
Result: Success

$ ./bsu.sh -report
Patch Report
============
  Report Info
    Report Options
      bea_home.................. ### OPTION NOT SET
      product_mask.............. ### OPTION NOT SET
      release_mask.............. ### OPTION NOT SET
      profile_mask.............. ### OPTION NOT SET
      patch_id_mask............. ### OPTION NOT SET
    Report Messages
  BEA Home.................. /db1/app/oracle/Middleware

  Product Description
  Product Name.............. WebLogic Server
  Product Version........... 10.3.6.0
  Installed Components...... Core Application Server, Administration Console, Configuration Wizard and Upgrade Framework, Web 2.0 HTTP Pub-Sub Server, WebLogic SCA, WebLogic JDBC Drivers, Third Party JDBC Drivers, WebLogic Server Clients, WebLogic Web Server Plugins, UDDI and Xquery Support, Workshop Code Completion Support
  Product Install Directory. /db1/app/oracle/Middleware/wlserver_10.3
  Java Home................. null
  Jave Vendor............... Sun
  Java Version.............. 1.6.0_29
  Patch Directory........... /db1/app/oracle/Middleware/patch_wls1036

~~~

**Обновить JAVA**
Оракл рекомендует обновить JAVA для устранения уязвимостей безопасности.

Текущий апдейт можно посмотреть здесь [[https://support.oracle.com/oip/faces/secure/km/DocumentDisplay.jspx?id=1506916.1]]

JAVA была настроена через линк `~/java -> ~/jdk1.7.0_60` (предполагалось, что Weblogic, будет использовать путь `~/java`), но Weblogic вместо линка использовал прямой путь и прописал в свои конфиги. Так что просто надо подменить каталоги:
~~~
cd /u/app/oracle
$ mv jdk1.7.0_60 jdk1.7.0_60.old
$ mv jdk1.7.0_191 jdk1.7.0_60

$ java -version
java version "1.7.0_191"
Java(TM) SE Runtime Environment (build 1.7.0_191-b08)
Java HotSpot(TM) 64-Bit Server VM (build 24.191-b08, mixed mode)

~~~

**Установка PSU**

**Распаковать PSU**

скопировать `p27919965_1036_Generic.zip` в каталог `/u/app/oracle/Middleware/utils/bsu/cache_dir`

~~~
$ cd /u/app/oracle/Middleware/utils/bsu/cache_dir
$ unzip p27919965_1036_Generic.zip
$ ls
B47X.jar  patch-catalog_26112.xml  README.txt
~~~

**Установить PSU**

~~~
$ cd ~/Middleware/utils/bsu/
$ ./bsu.sh -install -patch_download_dir=/u/app/oracle/distr/10.3.6.180717 -patchlist=B47X -prod_dir=/u/app/oracle/Middleware/wlserver_10.3

Checking for conflicts............
No conflict(s) detected

Installing Patch ID: B47X..
Result: Success

~~~

**Проверить установленный PSU**

~~~
$ ./bsu.sh -report

Patch Report
============
  Report Info
    Report Options
      bea_home.................. ### OPTION NOT SET
      product_mask.............. ### OPTION NOT SET
      release_mask.............. ### OPTION NOT SET
      profile_mask.............. ### OPTION NOT SET
      patch_id_mask............. ### OPTION NOT SET
    Report Messages
  BEA Home.................. /db1/app/oracle/Middleware

  Product Description
  Product Name.............. WebLogic Server
  Product Version........... 10.3.6.0
  Installed Components...... Core Application Server, Administration Console, Configuration Wizard and Upgrade Framework, Web 2.0 HTTP Pub-Sub Server, WebLogic SCA, WebLogic JDBC Drivers, Third Party JDBC Drivers, WebLogic Server Clients, WebLogic Web Server Plugins, UDDI and Xquery Support, Workshop Code Completion Support
  Product Install Directory. /db1/app/oracle/Middleware/wlserver_10.3
  Java Home................. null
  Jave Vendor............... Sun
  Java Version.............. 1.6.0_29
  Patch Directory........... /db1/app/oracle/Middleware/patch_wls1036

    Profile................... Default

      Patch ID.................. B47X
      CR(s)..................... 27919965
      Description............... WLS PATCH SET UPDATE 10.3.6.0.180717
WLS PATCH SET UPDATE 10.3.6.0.180717
      Classpath
        Classpath type............ SYSTEM
        Classpath control jar..... weblogic_patch.jar
          Jar....................... BUG27919965_10360180717.jar
.......

        Destination............... $WLS_INSTALL_DIR$/server/lib/consoleapp/APP-INF/lib/commons-fileupload.jar
        Destination............... $BEAHOME$/modules/glassfish.jstl_1.2.0.1.jar
~~~

**Запустить сервер Weblogic**
