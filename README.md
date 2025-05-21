To create a custom dashboard in AppDynamics that shows Java API application metrics including GC details, follow these steps:


---

Step-by-Step: Create a Custom Dashboard

1. Login to AppDynamics Controller UI

Go to your AppDynamics controller URL and log in.


---

2. Navigate to Dashboards

Top menu → Dashboards & Reports

Click “+ Create Dashboard”



---

3. Enter Dashboard Details

Name: e.g., Java API - GC Monitoring

Description (optional)

Choose Layout: Grid Layout is easier to start with.


Click “Create”


---

4. Add Widgets to Monitor GC Metrics

Click “+ Widget” and choose “Metric Widget”

Example Metrics to Add:

1. GC Time (ms)

Source: Select your Application > Tier > Node

Path:

Hardware Resources > JVM > Garbage Collection > GC Time Spent (ms)



2. GC Collection Count

Path:

Hardware Resources > JVM > Garbage Collection > GC Collection Count



3. Heap Memory Used

Path:

Hardware Resources > JVM > Memory > Heap Used (MB)



4. Heap Memory Committed

Path:

Hardware Resources > JVM > Memory > Heap Committed (MB)



5. CPU Usage

Path:

Hardware Resources > CPU % Busy



6. Response Time (for Java API health)

Path:

Business Transaction Performance > Average Response Time (ms)




> Use filters to find your Tier or Node, then navigate through the metric tree.




---

5. Customize Widgets

For each widget:

Set time range (e.g., last 15 mins)

Choose chart type (graph, bar, gauge, etc.)

Add titles like “GC Pause Time”, “Heap Used”, etc.



---

6. Save and Share

Once your dashboard is ready:

Click “Save”

Optionally, set sharing permissions for teams

You can also schedule this dashboard as a report



---

Optional: Add Health Rule Widgets

To show GC alerts:

Create Health Rules for GC in Alert & Respond > Health Rules

Add Health Status widgets showing the rule’s state on the dashboard



---

Would you like a sample dashboard JSON to import, or guidance on setting thresholds (e.g., alert if GC time > 500ms)?
