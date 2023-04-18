### Servers,

**Apache / mod_jk connector server :** http://192.168.40.129/

**Backend tomcat :** http://192.168.40.128:7070/

**Backend tomcat :** http://192.168.40.128:8080/

**Backend tomcat :** http://192.168.40.128:9090/

<br>

### Backend server configuration, Adding JVMRoute in server.xml,

<br>

**On, Backend tomcat :** http://192.168.40.128:7070/

    Shutdown 	=	7005
    http		=	7070
    https		=	7443
    AJP		=	7007


`[root@guest conf]# nano server.xml`

    <Connector port="7070" protocol="HTTP/1.1"
                   connectionTimeout="20000"
                   redirectPort="7443" />
        <!-- A "Connector" using the shared thread pool-->
    
    
    
        <Engine name="Catalina" defaultHost="localhost" jvmRoute="worker7070">
    
    
        <Connector protocol="AJP/1.3"
                   address="192.168.40.128"
                   port="7009"
                   redirectPort="7443" 
                   secretRequired="false" />

<br>

**On, Backend tomcat :** http://192.168.40.128:8080/

    Shutdown 	=	8005
    http		=	8080
    https		=	8443
    AJP		=	8009

`[root@guest conf]# nano server.xml`

    <Connector port="8080" protocol="HTTP/1.1"
                   connectionTimeout="20000"
                   redirectPort="8443" />
        <!-- A "Connector" using the shared thread pool-->
    
    
    
        <Engine name="Catalina" defaultHost="localhost" jvmRoute="worker8080">
    
    
        <Connector protocol="AJP/1.3"
                   address="192.168.40.128"
                   port="8009"
                   redirectPort="8443" 
                   secretRequired="false" />

<br>

**On, Backend tomcat :** http://192.168.40.128:9090/

    Shutdown 	=	9005
    http		=	9090
    https		=	9443
    AJP		=	9009


`[root@guest conf]# nano server.xml`

    <Connector port="9090" protocol="HTTP/1.1"
                   connectionTimeout="20000"
                   redirectPort="9443" />
        <!-- A "Connector" using the shared thread pool-->
    
    
    
        <Engine name="Catalina" defaultHost="localhost" jvmRoute="worker9090">
    
    
        <Connector protocol="AJP/1.3"
                   address="192.168.40.128"
                   port="9009"
                   redirectPort="9443" 
                   secretRequired="false" />

<br>

### Apache, mod_jk connector server configuration,

**On, Apache / mod_jk connector server :** http://192.168.40.129/

<br>

**Make sure the apache services are running,**

`[root@server native]# yum install -y httpd`

<br>

**Install Dependencies,**

`[root@server native]# yum install -y httpd-devel`

`[root@server native]# yum update httpd`

`[root@server native]# yum install gcc`

<br>

**Download mod_jk connector (tar.gz format) from Tomcat website,**

**Download,** https://tomcat.apache.org/download-connectors.cgi 

`[root@server Softwares]# wget https://mirrors.estointernet.in/apache/tomcat/tomcat-connectors/jk/tomcat-connectors-1.2.48-src.tar.gz`

`[root@server Softwares]# tar -xvzf tomcat-connectors-1.2.48-src.tar.gz`

`[root@server Softwares]# ls -ltr`

<br>

**Install,**

`[root@server Softwares]# cd tomcat-connectors-1.2.48-src/`

`[root@server tomcat-connectors-1.2.48-src]# cd native`

`[root@server native]# yum install autoconf`

`[root@server native]# yum install libtool`

`[root@server native]# pwd`

    /home/anup/Softwares/tomcat-connectors-1.2.48-src/native

`[root@server native]# ./buildconf.sh`

<br>

**Configure with APXS,**

`[root@server native]# ./configure --with-apxs=/usr/bin/apxs`

`[root@server native]# ./configure --with-apache=/etc/httpd`

`[root@server native]# ./configure --with-mod_jk`

`[root@server native]# sudo dnf install redhat-rpm-config`

<br>

**Execute the Make Command,**

`[root@server native]# libtool --finish /usr/lib64/httpd/modules`

`[root@server native]# make install`

`[root@server conf]# pwd /etc/httpd/conf`

`[root@server conf]# nano httpd.conf`

<br>

**Copy the Generated mod_jk.so file to apache modules,**

`[root@server ~]# find / -name mod_jk.so`

`[root@server ~]# locate -i mod_jk.so`

<br>

`[root@server apache-2.0]# pwd /home/anup/Softwares/tomcat-connectors-1.2.48-src/native/apache-2.0`

`[root@server apache-2.0]# ls -ltr | grep -i "mod_jk.so"`

<br>

`[root@server modules]# pwd /etc/httpd/modules`

`[root@server modules]# ls -ltr | grep -i "mod_jk.so"`

<br>

`[root@server ~]# cd /etc/httpd/modules/`

`[root@server modules]# mv mod_jk.so /home/anup/Softwares/`

`[root@server modules]# cp /home/anup/Softwares/tomcat-connectors-1.2.48-src/native/apache-2.0/mod_jk.so .`

<br>

**Load the Module mod_jk.so,**

`[root@server httpd]# grep -i ^Include conf/httpd.conf`

`[root@server httpd]# cd conf.modules.d`

`[root@server conf.modules.d]# pwd /etc/httpd/conf.modules.d`

`[root@server conf.modules.d]# ls -ltr`

`[root@server conf.modules.d]# nano mod_jk.conf`

    ServerName 192.168.40.129
    
    LoadModule jk_module "/etc/httpd/modules/mod_jk.so"
    
    JkWorkersFile /etc/httpd/conf/workers.properties
    JkLogFile /etc/httpd/logs/mod_jk.log
    
    #JkMount /myapp/* lb_router
    #JkMount /* lb_router
    JkMount /docs/*  lb_router
    JkMount /docs*  lb_router
    
    JkLogLevel debug
    
    JkLogStampFormat "[%a %b %d %H:%M:%S %Y]"
    JkOptions +ForwardKeySize +ForwardURICompat -ForwardDirectories
    JkRequestLogFormat "%w %V %T"

<br>

`[root@server conf]# nano /etc/httpd/conf/workers.properties`

    ### Define the list of workers you have
    worker.list=jkstatus,lb_router
    
    ### Set loadbalancer
    worker.lb_router.type=lb
    worker.jkstatus.type=status
    
    # HERE is where you decide how many Tomcat Server are in the cluster
    worker.lb_router.balance_workers=worker7070,worker8080,worker9090
    worker.lb_router.sticky_session=1
    
    ### Set Worker
    worker.worker7070.port=7009
    worker.worker7070.host=192.168.40.128
    worker.worker7070.type=ajp13
    worker.worker7070.lbfactor=1
    
    worker.worker8080.port=8009
    worker.worker8080.host=192.168.40.128
    worker.worker8080.type=ajp13
    worker.worker8080.lbfactor=1
    
    worker.worker9090.port=9009
    worker.worker9090.host=192.168.40.128
    worker.worker9090.type=ajp13
    worker.worker9090.lbfactor=1
    

<br>

**Update your Apache Virtual Host file and add JKMount,**

    JkMount /<Your Application Name>/* lb_router
    JkMount /myapp/* lb_router

<br>

**Validate the Logs – Audit the Logs,**

`[root@server conf]# tail -20f /etc/httpd/mod_jk.log`

<br>
