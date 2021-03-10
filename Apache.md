0. 포트는 4000~4100, 45564 (설정 세팅 포트), 기타 서비스 포트 해제 필요
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
  worker.worker1.host=10.131.64.178 #(tomcat IP)
  worker.worker1.port=8090 #(사용포트)
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

2. reverse proxy (apache, mod 설치 생략)
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
         ProxyPass /mvcmys http://visit.ldcc.co.kr/mvcmys #(ip주소:8081/mvcmys 로 요청 올 경우 http://visit.ldcc.co.kr/mvcmys 로 redirect)
         ProxyPassReverse /mvcmys http://visit.ldcc.co.kr/mvcmys #(ip주소:8081/mvcmys 로 요청 올 경우 http://visit.ldcc.co.kr/mvcmys 로 redirect)
 </VirtualHost>
 ```

3. 세션 클러스터링 (apache, mod 설치 생략)
 - vi /etc/httpd/conf/workers.properties (아파치 서버만)
 ```
 worker.list=tomcat_lb

 worker.tomcat1.type=ajp13
 worker.tomcat1.host=20.194.19.182
 worker.tomcat1.port=8009
 worker.tomcat1.lbfactor=100

 worker.tomcat2.type=ajp13
 worker.tomcat2.host=20.194.52.63
 worker.tomcat2.port=8009
 worker.tomcat2.lbfactor=100

 worker.tomcat_lb.type=lb
 worker.tomcat_lb.balanced_workers=tomcat1,tomcat2
 ```
 - vi /usr/local/tomcat8/conf/server.xml (아래내용 추가 receiver 아이피 주소만 변경, 포트는 건들지 않는다.) (모든 톰캣 서버 적용)
 ```
 <Engine name="Catalina" defaultHost="localhost" jvmRoute="tomcat1"> #(여기는 jvmRoute="tomcat1"(worker에 명시한 이름) 만 추가)
 <Cluster className="org.apache.catalina.ha.tcp.SimpleTcpCluster"/> #(여기는 주석처리 해제)
 <Cluster className="org.apache.catalina.ha.tcp.SimpleTcpCluster"
               channelSendOptions="8">
        <Manager className="org.apache.catalina.ha.session.DeltaManager"
                      expireSessionsOnShutdown="false" notifyListenersOnReplication="true"/>
        <Channel className="org.apache.catalina.tribes.group.GroupChannel">
           <Membership className="org.apache.catalina.tribes.membership.McastService"
                              address="228.0.0.4"
                              port="45564"
                              frequency="500"
                              dropTime="3000"/>
           <Receiver className="org.apache.catalina.tribes.transport.nio.NioReceiver"
                         address="localhost" #(아이피 주소를 적어야 하는데 dns 설정이 이상하면 아이피 주소를 인식못해 localhost로 적는다)
                         port="4000"
                         autoBind="100"
                         selectorTimeout="5000"
                         maxThreads="6"/>
           <Sender className="org.apache.catalina.tribes.transport.ReplicationTransmitter">
               <Transport className="org.apache.catalina.tribes.transport.nio.PooledParallelSender"/>
           </Sender>
           <Interceptor className="org.apache.catalina.tribes.group.interceptors.TcpFailureDetector"/>
           <Interceptor className="org.apache.catalina.tribes.group.interceptors.MessageDispatchInterceptor"/>
 </Channel>
         <Valve className="org.apache.catalina.ha.tcp.ReplicationValve" filter=""/>
         <Valve className="org.apache.catalina.ha.session.JvmRouteBinderValve"/>
         <Deployer className="org.apache.catalina.ha.deploy.FarmWarDeployer"
                        tempDir="/tmp/war-temp/"
                        deployDir="/tmp/war-deploy/"
                        watchDir="/tmp/war-listen/"
                        watchEnabled="false"/>
          <ClusterListener className="org.apache.catalina.ha.session.ClusterSessionListener"/>
 </Cluster>
 ```
 - vi /usr/local/tomcat8/conf/web.xml (가장아래에 추가) (모든 톰캣 적용)
```
<distributable/> #(</web-app> 위에 추가)
```
- vi /usr/local/tomcat8/webapps/ROOT/index.jsp (테스트 jsp 작성, 기존 index.jsp는 삭제 or 백업처리)
``` html
<%
session.setAttribute(“a”,”a”);
%>

<html>
<head>
<title>Session Clustering Test</title>
</head>
<body>
<table width=”100%” border=”0″ cellspacing=”0″ cellpadding=”0″>
<tr bgcolor=”#CCCCCC”>
<td width=”13%”>Tomcat1</td>
<td width=”87%”> </td>
</tr>
<tr>
<td>Session ID :</td>
<td><%=session.getId()%></td>
</tr>
</table>
<div>Im Tomcat1</div>
</body>
</html>
```
- route add -net 224.0.0.0 netmask 240.0.0.0 dev eth0(인터페이스명) (멀티캐스팅 라우팅 추가 부분, netstat -nr 해서 224.0.0.0 없을경우)




tomcat port 설정 참고 사이트 : https://fruitdev.tistory.com/23
