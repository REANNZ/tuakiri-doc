
The following instructions cover the installation of Tomcat 8 (8.5) on CentOS 7 - the same set of instructions should also work on CentOS 8.

The instructions are designed to work either when the Tomcat 7 RPM is already installed or not - and are designed to make future updates to newer Tomcat version easier - e.g., while there are per-version directories for Tomcat distribution, the directory for application context-descriptor files is shared across the multiple Tomcat versions.

To install Tomcat8.5:

*   Stop Tomcat if it's currently running:
    
    ```
    service tomcat stop
    ```
    
*   Make sure Tomcat account exists:
    
    ```
    id tomcat || useradd --system --comment "Apache Tomcat" --shell /sbin/nologin tomcat
    ```
    
*   Create directory hierarchy:
    
    ```
    mkdir -p /opt/tomcat /opt/tomcat/context
    chown -R tomcat.tomcat /opt/tomcat
    ```
    
*   Download and deploy Tomcat binary:
    
    ```
    TOMCAT_VERSION=8.5.60
    cd /opt/tomcat
    wget https://archive.apache.org/dist/tomcat/tomcat-8/v${TOMCAT_VERSION}/bin/apache-tomcat-${TOMCAT_VERSION}.tar.gz
    tar xzf apache-tomcat-${TOMCAT_VERSION}.tar.gz
    # fix up permissions overall
    chown -R tomcat.tomcat /opt/tomcat/apache-tomcat-${TOMCAT_VERSION}
    chmod -R +r /opt/tomcat/apache-tomcat-${TOMCAT_VERSION}
    # fix up permissions on dirs
    chmod +rx /opt/tomcat/apache-tomcat-${TOMCAT_VERSION}/{.,bin,conf,lib,logs,temp,webapps,work}
    ln -s /opt/tomcat/apache-tomcat-${TOMCAT_VERSION} /opt/tomcat/current
    ```
    
*   Create override settings:
    
    ```
    cat > /opt/tomcat/current/bin/setenv.sh <<EOF
    CATALINA_PID=/run/tomcat/tomcat.pid
    UMASK=0022
    JAVA_HOME=/usr/lib/jvm/java-11-openjdk
    JAVA_OPTS="-Xms1024m -Xmx2048m"
    EOF
    ```
    
*   Configure Tomcat: edit `/opt/tomcat/current/conf/server.xml` and:  
    *   comment out port 8080 HTTP Connector
    *   add port 8009 AJP connector:
        
        ```
        <Connector port="8009" address="127.0.0.1"
        enableLookups="false" redirectPort="443" protocol="AJP/1.3"
        tomcatAuthentication="false"
        secretRequired="false" />
        ```
        
    *   in Host element, set `autoDeploy="false"`  (change from `true` to avoid reloads when touching the `idp.war` file)
    *   in Host element, set `xmlBase="/opt/tomcat/context"`  - to use version-independent context descriptor directory.
*   For use with Shibboleth IdP, download JSTL jar files:
    
    ```
    wget -O /opt/tomcat/current/lib/javax.servlet.jsp.jstl-api-1.2.1.jar 'http://search.maven.org/remotecontent?filepath=javax/servlet/jsp/jstl/javax.servlet.jsp.jstl-api/1.2.1/javax.servlet.jsp.jstl-api-1.2.1.jar'
    wget -O /opt/tomcat/current/lib/javax.servlet.jsp.jstl-1.2.1.jar 'http://search.maven.org/remotecontent?filepath=org/glassfish/web/javax.servlet.jsp.jstl/1.2.1/javax.servlet.jsp.jstl-1.2.1.jar'
    ```
    
*   Assuming MySQL JDBC driver is already installed locally and deployed applications will need it, symlink it into Tomcat lib directory:
    
    ```
    ln -s /usr/share/java/mysql-connector-java.jar /opt/tomcat/current/lib/
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
    
*   Deploy the IdP web-app into this Tomcat: create {{/opt/tomcat/context/idp.xml}} with:
    
    ```
    <Context docBase="/opt/shibboleth-idp/war/idp.war"
             unpackWAR="true"
             swallowOutput="true" />
    ```
    
*   Reload systemd config and start Tomcat:
    
    ```
    systemctl daemon-reload
    systemctl enable tomcat
    systemctl start tomcat
    ```
