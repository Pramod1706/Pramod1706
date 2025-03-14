filebeat.inputs:
  - type: log
    enabled: true
    paths:
      - "/appvol/logs/ikp/uat/*.log"
      - "/appvol/logs/ikp/oat/*.log"
      - "/appvol/logs/ikp/adt/*.log"
    multiline.pattern: '^[0-9]{4}-[0-9]{2}-[0-9]{2}'
    multiline.negate: true
    multiline.match: after
    scan_frequency: 10s         # Reduce CPU usage
    close_inactive: 5m          # Free memory by closing inactive logs
    ignore_older: 2d            # Ignore logs older than 2 days (indirectly skips last 1 day)
    clean_removed: true         # Prevent Filebeat from tracking deleted logs

output.logstash:
  hosts: ["logstash.gra-elk-snbx5.svc.cluster.local:5044"]
  bulk_max_size: 5000           # Optimize bulk processing
  worker: 2                     # Improve parallel processing
  timeout: 30s                  # Prevent timeouts for large logs

queue.mem:
  events: 8192                  # Controls memory buffer size
  flush.min_events: 512         # Flush data frequently to avoid memory overflow
  flush.timeout: 5s             # Reduce log delay

logging.level: info
logging.to_stderr: true
