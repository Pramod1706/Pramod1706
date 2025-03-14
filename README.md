I’ll update your configuration files with the necessary details based on our previous discussions. Since you've shared multiple files, I’ll modify them accordingly:


---

1. JVM Options Configuration (jvm.options)

This file sets up JVM heap memory, GC settings, and logging.

Updated jvm.options

## GC Configuration
8-13:-XX:+UseConcMarkSweepGC
8-13:-XX:CMSInitiatingOccupancyFraction=75
8-13:-XX:+UseCMSInitiatingOccupancyOnly
8-13:-XX:NewRatio=2
8-13:-XX:ParallelGCThreads=2

## JVM Temporary Directory
-Djava.io.tmpdir=${ES_TMPDIR}

## Heap Dumps
-XX:+HeapDumpOnOutOfMemoryError
-XX:HeapDumpPath=data

## Fatal Error Logs
-XX:ErrorFile=logs/hs_err_pid%p.log

## JDK 8 GC Logging
8:-XX:+PrintGCDetails
8:-XX:+PrintGCDateStamps
8:-XX:+PrintTenuringDistribution
8:-XX:+PrintGCApplicationStoppedTime
8:-Xloggc:logs/gc.log
8:-XX:+UseGCLogFileRotation
8:-XX:NumberOfGCLogFiles=32
8:-XX:GCLogFileSize=64m

## JDK 9+ GC Logging
9-: -Xlog:gc*,gc+age=trace,safepoint:file=logs/gc.log:utctime,pid,tags:filecount=32,filesize=64m


---

2. Elasticsearch Configuration (elasticsearch.yml)

This file configures cluster settings, node roles, and networking.

Updated elasticsearch.yml

cluster.name: k8s-logs
node.name: ${HOSTNAME}
network.host: ${HOSTIP}

## Master Node Configuration
cluster.initial_master_nodes:
  - es-cluster-0-0
  - es-cluster-1-0
  - es-cluster-2-0

## Discovery Settings for Cluster Nodes
discovery.zen.ping.unicast.hosts:
  - es-cluster-0-0.elasticsearch.gra-elk-snbx2.svc.cluster.local
  - es-cluster-1-0.elasticsearch.gra-elk-snbx3.svc.cluster.local
  - es-cluster-2-0.elasticsearch.gra-elk-snbx4.svc.cluster.local

## Node Roles
node.roles:
  - master
  - data_hot
  - data_content
  - data_warm
  - ingest
  - remote_cluster_client
  - transform

## S3 Repository Configuration
s3.client.hsbc.endpoint: ecs-storage.it.global.hsbc:9021
s3.client.hsbc.path_style_access: true
s3.client.hsbc-sg.endpoint: sg-storage.it.global.hsbc:9021
s3.client.hsbc-sg.path_style_access: true


---

3. Elasticsearch Plugins Configuration (elasticsearch-plugins-config.yml)

This file configures Elasticsearch plugins.

Updated elasticsearch-plugins-config.yml

plugins:
  - id: repository-s3
    location: file:///tmp/packages/repository-s3-7.16.2.zip


---

Next Steps

Apply the updates to your Kubernetes ConfigMaps.

Restart your Elasticsearch pods to reflect changes.


Do you need any additional modifications?
