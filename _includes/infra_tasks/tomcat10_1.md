
The following instructions cover the installation of Tomcat 10 (10.1) on RHEL9-like system - the same set of instructions should also work on CentOS 8.

The instructions are designed to work either when the Tomcat RPM is already installed or not - and are designed to make future updates to newer Tomcat version easier - e.g., while there are per-version directories for Tomcat distribution, the directory for application context descriptor files is shared across the multiple Tomcat versions.

To install Tomcat10:

*   Stop Tomcat if it's currently running:
    
    ```
    service tomcat stop
    ```
    
*   Make sure Tomcat account exists:
    
    ```
    id tomcat 2> /dev/null || { useradd --system --comment "Apache Tomcat" --shell /sbin/nologin tomcat && id tomcat ; }
    ```
    
*   Create directory hierarchy (incl. version-independent directory for context descriptor files):
    
    ```
    mkdir -p /opt/tomcat /opt/tomcat/context
    chown -R tomcat.tomcat /opt/tomcat
    ```
    
*   Install prerequisites:
    
    ```
    yum install wget tar
    ```
    
*   Download and deploy Tomcat binary:
    
    ```
    TOMCAT_VERSION=10.1.20
    cd /opt/tomcat
    wget https://archive.apache.org/dist/tomcat/tomcat-10/v${TOMCAT_VERSION}/bin/apache-tomcat-${TOMCAT_VERSION}.tar.gz
    tar xzf apache-tomcat-${TOMCAT_VERSION}.tar.gz
    # fix up permissons overall
    chown -R tomcat.tomcat /opt/tomcat/apache-tomcat-${TOMCAT_VERSION}
    chmod -R +r /opt/tomcat/apache-tomcat-${TOMCAT_VERSION}
    # fix up permissions on dirs
    chmod +rx /opt/tomcat/apache-tomcat-${TOMCAT_VERSION}/{.,bin,conf,lib,logs,temp,webapps,work}
    # Create a link to current version
    ln -s /opt/tomcat/apache-tomcat-${TOMCAT_VERSION} /opt/tomcat/current
    ```
    
*   Create override settings:
    
    ```
    cat > /opt/tomcat/current/bin/setenv.sh <<EOF
    CATALINA_PID=/run/tomcat/tomcat.pid
    UMASK=0022
    JAVA_HOME=/usr/lib/jvm/java-17-openjdk
    JAVA_OPTS="-Xms1024m -Xmx2048m"
    EOF
    ```
    
*   Remove sample web applications included with Tomcat:
    
    ```
    rm -rf /opt/tomcat/current/webapps/{docs,examples,host-manager,manager,ROOT}
    ```
    
*   Configure Tomcat: edit `/opt/tomcat/current/conf/server.xml` and:  
    *   comment out port 8080 HTTP Connector
    *   add port 8009 AJP connector:
        
        ```
        <Connector port="8009" address="127.0.0.1"
        enableLookups="false" redirectPort="443" protocol="AJP/1.3"
        proxyPort="443" scheme="https" secure="true"
        tomcatAuthentication="false"
        secretRequired="false" />
        ```
        
    *   in Host element, set `autoDeploy="false"`  (change from `true` to avoid reloads when touching the `idp.war` file)
    *   in Host element, set `xmlBase="/opt/tomcat/context"`  - to use version-independent context descriptor directory.
*   For use with Shibboleth IdP, download JSTL jar:
    
    ```
    wget -O /opt/tomcat/current/lib/jakarta.servlet.jsp.jstl-api-3.0.0.jar 'https://repo1.maven.org/maven2/jakarta/servlet/jsp/jstl/jakarta.servlet.jsp.jstl-api/3.0.0/jakarta.servlet.jsp.jstl-api-3.0.0.jar'
    ```
    
*   Make sure MySQL JDBC driver is installed - on RHEL9, this would be the MariaDB driver:
    
    ```
    yum install mariadb-java-client
    ```
    
*   Assuming the MySQL JDBC driver is already installed in `/usr/lib/java/mariadb-java-client.jar`, symlink it into Tomcat lib directory:
    
    ```
    ln -s /usr/lib/java/mariadb-java-client.jar /opt/tomcat/current/lib/
    ```
    
*   Create systemd unit file (this file will override the unit file shipped with the Tomcat RPM if installed).  (This block assumes `TOMCAT_VERSION` is still set as per above).
    
    ```
    cat > /etc/systemd/system/tomcat.service <<EOF
    [Unit]
    Description=Tomcat ${TOMCAT_VERSION}
    After=network.target
    
    [Service]
    Type=forking
    PIDFile=/run/tomcat/tomcat.pid
    User=tomcat
    Group=tomcat
    ExecStart=/opt/tomcat/current/bin/startup.sh
    ExecStop=/opt/tomcat/current/bin/shutdown.sh
    RuntimeDirectory=tomcat
    
    # TERM to main process only, KILL to all
    KillMode=mixed
    
    #restart: in case of failure, try starting every 30 seconds
    Restart=always
    RestartSec=30
    
    [Install]
    WantedBy=multi-user.target
    EOF
    ```
    
*   Deploy the IdP web-app into this Tomcat: create {{/opt/tomcat/context/idp.xml}} with the following (also disabling unused servlets included with the IdP):
    
    ```
    <Context docBase="/opt/shibboleth-idp/war/idp.war"
             unpackWAR="true"
             swallowOutput="true" >
       <Parameter name="net.shibboleth.idp.registerRemoteUserServlet" value="false" override="false" />
       <Parameter name="net.shibboleth.idp.registerX509Servlet" value="false" override="false" />
    </Context>
    ```
    
*   Reload systemd config and start Tomcat:
    
    ```
    systemctl daemon-reload
    systemctl enable tomcat
    systemctl start tomcat
    ```
