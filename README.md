-Xms2500m
-Xmx2500m
-XX:+UseG1GC
-XX:InitiatingHeapOccupancyPercent=50
-XX:+AlwaysPreTouch
-XX:+HeapDumpOnOutOfMemoryError
-Djava.awt.headless=true
-Dfile.encoding=UTF-8
-Djruby.compile.invokedynamic=true
-Djruby.jit.threshold=0
-Djava.security.egd=file:/dev/urandom

# AppDynamics settings remain the same
-javaagent:/opt/appdynamics/java-oracle-agent/javaagent.jar
-Dappdynamics.install.dir=/opt/appdynamics/java-oracle-agent/
-Dappdynamics.opentelemetry.enabled=false
-Dappdynamics.agent.runtime.dir=/home/javadm/java-oracle-agent
-Dappdynamics.agent.logs.dir=/home/javadm/java-oracle-agent/logs
-Dappdynamics.agent.conf.dir=/home/javadm/java-oracle-agent/conf
-Dappdynamics.controller.ssl.enabled=true
-Dappdynamics.force.default.ssl.certificate.validation=false
-Dappdynamics.agent.applicationName=WREN-ELK-UAT
-Dappdynamics.agent.tierName=logstash
-Dappdynamics.agent.uniqueHostId=logstash-uat
-Dappdynamics.agent.nodeName=logstash-uat
-Dappdynamics.controller.hostName=gratest.saas.appdynamics.com
-Dappdynamics.controller.port=443
-Dappdynamics.agent.accountName=gratest
-Dappdynamics.agent.accountAccessKey=i04zdo61nahs
-Dappdynamics.http.proxyPort=37071
-Dappdynamics.http.proxyHost=app-dynamics-proxy.hk.hsbc
-Dappdynamics.https.proxyPort=37071
-Dappdynamics.https.proxyHost=app-dynamics-proxy.hk.hsbc
