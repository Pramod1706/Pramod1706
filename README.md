Got it! Below is the optimized high-speed setup with 1 Logstash, 1 Filebeat, 1 Kibana, and 3 Elasticsearch Pods, ensuring fast ingestion, indexing, and search performance for your 200GB historical + 20GB daily logs.


---

🚀 Optimized Resource Allocation for Maximum Speed


---

🔥 Key Performance Optimizations

1️⃣ 3 Elasticsearch Pods as Master + Data

Balanced cluster, high availability, and fast indexing


2️⃣ Optimized Elasticsearch Heap

-Xms4g -Xmx4g (50% of 8GB RAM per pod)


3️⃣ Fast Log Ingestion with Logstash

Batch Size: 1000 → More logs processed per cycle

Batch Delay: 10ms → Lower delay for real-time logs

Persistent Queue: Enabled → Prevents log loss


4️⃣ Filebeat Tweaks for Faster Log Shipping

Bulk Max Size: 2000 → Sends larger batches to Logstash

Queue Flush Timeout: 5s → Reduces latency


5️⃣ SSD Storage (At Least 1TB)

Ensures fast I/O for indexing & searches



---

⚡ Performance Metrics with This Setup


---

📌 When to Scale Further?

If logs exceed 30GB/day or users increase beyond 300, consider:
✅ Increasing Logstash batch size to 1500
✅ Increasing Elasticsearch heap to 6GB (-Xms6g -Xmx6g)
✅ Adding 1 more Elasticsearch pod for better distribution


---

This setup ensures high-speed log ingestion, real-time search, and smooth dashboard performance with minimal pod usage. Let me know if you need fine-tuning! 🚀

