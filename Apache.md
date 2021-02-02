1. mod_jk 활용 apache - tomcat 연동
 - https://www.lesstif.com/system-admin/apache-httpd-tomcat-connector-mod_jk-reverse-proxy-mod_proxy-12943367.html
 - wget http://ftp.daum.net/apache//tomcat/tomcat-connectors/jk/tomcat-connectors-1.2.37-src.tar.gz (mod_jk Down)
 - tar zxvf tomcat-connectors-1.2.37-src.tar.gz
 - cd tomcat-connectors-1.2.37-src/native
 - ./configure --with-apxs=/usr/sbin/apxs (apxs 설치 안되어있을 경우 yum install httpd-devel, apxs경로 확인 필수)
 - make & make install
 - vi /etc/httpd/conf/httpd.conf (아래 내용 추가)
  ```xml
  LoadModule jk_module modules/mod_jk.so
  ```
 - vi vi workers_jk.properties
 ```xml
  worker.list=worker1

  worker.worker1.type=ajp13
  worker.worker1.host=10.131.64.178
  worker.worker1.port=8090
  ```
 - vi mod_js.conf
 ```
 <IfModule mod_jk.c>
   # Where to find workers.properties
   JkWorkersFile conf/workers_jk.properties
   # Where to put jk shared memory
   JkShmFile run/mod_jk.shm
   # Where to put jk logs
   JkLogFile logs/mod_jk.log
   # Set the jk log level [debug/error/info]
   JkLogLevel info
   # Select the timestamp log format
   JkLogStampFormat "[%a %b %d %H:%M:%S %Y] "
   ## url pattern 에 따른 connector mapping
   ##JkMountFile conf/uriworkermap.properties
   JkMount /* worker1
 </IfModule>
 ```
- tomcat vi server.xml
```
  <Connector port="8009" protocol="AJP/1.3" redirectPort="8443" URIEncoding="UTF-8"/>
```
- 만약 boot tomcat의 경우
``` java
 @Configuration
  public class ContainerConfig {
    @Bean
    public EmbeddedServletContainerCustomizer containerCustomizer() {
      return container -> {
        TomcatEmbeddedServletContainerFactory tomcat =
            (TomcatEmbeddedServletContainerFactory) container;

        Connector ajpConnector = new Connector("AJP/1.3");
        ajpConnector.setProtocol("AJP/1.3");
        ajpConnector.setPort(8090);
        ajpConnector.setSecure(false);
        ajpConnector.setAllowTrace(false);
        ajpConnector.setScheme("http");
        ajpConnector.setURIEncoding("UTF-8");
        tomcat.addAdditionalTomcatConnectors(ajpConnector);
      };
    }
  }
```

2. reverse proxy
 - 

3. 세션 클러스터링
 - https://idchowto.com/?p=53319
