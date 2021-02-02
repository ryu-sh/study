1. mod_jk 활용 apache - tomcat 연동
 - https://www.lesstif.com/system-admin/apache-httpd-tomcat-connector-mod_jk-reverse-proxy-mod_proxy-12943367.html (참고사이트)
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
  worker.worker1.host=10.131.64.178 (tomcat IP)
  worker.worker1.port=8090 (사용포트)
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
- tomcat vi server.xml (주석 되어있을텐데 해제하면 됨, 8443포트는 아마 내부통신에 필요한듯)
```
  <Connector port="8090" protocol="AJP/1.3" redirectPort="8443" URIEncoding="UTF-8"/>
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

2. reverse proxy (apache 설치 생략)
 - vi httpd.conf (아래 내용 추가)
 ```
 Include conf/extra/httpd-vhosts.conf
 LoadModule proxy_module modules/mod_proxy.so
 LoadModule proxy_http_module modules/mod_proxy_http.so
 ```
 - vi /etc/httpd/conf/extra/httpd-vhosts.conf (https redirect는 안된다. 에러남)
 ```
 <VirtualHost *:8081> (포트)
     # Put this in the main section of your configuration (or desired virtual host, if using Apache virtual hosts)
     ProxyRequests Off
     ProxyPreserveHost On
         <Proxy *>
                 Order deny,allow
                 Allow from all
         </Proxy>
         ProxyPass /mvcmys http://visit.ldcc.co.kr/mvcmys (ip주소:8081/mvcmys 로 요청 올 경우 http://visit.ldcc.co.kr/mvcmys 로 redirect)
         ProxyPassReverse /mvcmys http://visit.ldcc.co.kr/mvcmys (ip주소:8081/mvcmys 로 요청 올 경우 http://visit.ldcc.co.kr/mvcmys 로 redirect)
 </VirtualHost>
 ```
 - v

3. 세션 클러스터링
 - https://idchowto.com/?p=53319
