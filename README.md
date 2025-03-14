cluster.name: k8s-logs
node.name: ${HOSTNAME}
network.host: ${HOSTIP}

## Master Node Configuration
cluster.initial_master_nodes:
  - es-cluster-0-0
  - es-cluster-1-0
  - es-cluster-2-0

## Discovery Settings for Cluster Nodes
discovery.seed_hosts:
  - es-cluster-0-0.elasticsearch.gra-elk-snbx2.svc.cluster.local
  - es-cluster-1-0.elasticsearch.gra-elk-snbx3.svc.cluster.local
  - es-cluster-2-0.elasticsearch.gra-elk-snbx4.svc.cluster.local

## Node Roles - Optimized for Performance & S3 Storage
node.roles:
  - master
  - data_hot   # For frequently accessed logs
  - data_content  # Stores logs in S3 for retrieval
  - ingest     # For pre-processing logs before indexing
  - remote_cluster_client  # Enables cross-cluster search
  - transform  # Allows log aggregation & analytics

## Performance Optimizations
thread_pool.write.queue_size: 1000   # Prevents bulk indexing failures under heavy load
thread_pool.search.size: 10          # Controls concurrent searches
indices.memory.index_buffer_size: 20%  # Allocates memory for indexing performance
indices.store.preload: ["nvd", "dvd", "tim", "doc", "dim"]  # Optimizes disk read performance

## Indexing & Refresh Settings
index.refresh_interval: 10s           # Balance between speed & performance
index.number_of_shards: 2              # Split data across nodes for faster search
index.number_of_replicas: 0            # No replicas (as per your setup)

## Disk Watermark Settings (Prevents Cluster Issues)
cluster.routing.allocation.disk.watermark.low: 85%
cluster.routing.allocation.disk.watermark.high: 90%
cluster.routing.allocation.disk.watermark.flood_stage: 95%

## S3 Repository Configuration (For Snapshot Storage)
s3.client.hsbc.endpoint: ecs-storage.it.global.hsbc:9021
s3.client.hsbc.path_style_access: true
s3.client.hsbc-sg.endpoint: sg-storage.it.global.hsbc:9021
s3.client.hsbc-sg.path_style_access: true
