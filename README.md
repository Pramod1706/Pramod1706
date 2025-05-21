{
  "name": "Java API - Hardware and Performance Dashboard",
  "description": "Custom dashboard for monitoring CPU, memory, GC, and API performance metrics.",
  "template": false,
  "refreshInterval": 60,
  "minutesBeforeNow": 15,
  "backgroundColor": "rgba(255,255,255,1)",
  "height": 768,
  "width": 1366,
  "widgets": [
    {
      "title": "CPU % Busy",
      "height": 200,
      "width": 400,
      "x": 0,
      "y": 0,
      "widgetType": "GraphWidget",
      "dataSeriesTemplates": [
        {
          "metricMatchCriteria": {
            "applicationName": "WREN-IKP-UAT-103s",
            "metricPath": "Hardware Resources|*|CPU|%Busy"
          },
          "color": "rgba(0,116,217,1)",
          "label": "CPU % Busy"
        }
      ]
    },
    {
      "title": "Heap Used (MB)",
      "height": 200,
      "width": 400,
      "x": 0,
      "y": 220,
      "widgetType": "GraphWidget",
      "dataSeriesTemplates": [
        {
          "metricMatchCriteria": {
            "applicationName": "WREN-IKP-UAT-103s",
            "metricPath": "Application Infrastructure Performance|*|JVM|Memory|Heap|Used"
          },
          "color": "rgba(0,116,217,1)",
          "label": "Heap Used (MB)"
        }
      ]
    },
    {
      "title": "Heap Committed (MB)",
      "height": 200,
      "width": 400,
      "x": 0,
      "y": 440,
      "widgetType": "GraphWidget",
      "dataSeriesTemplates": [
        {
          "metricMatchCriteria": {
            "applicationName": "WREN-IKP-UAT-103s",
            "metricPath": "Application Infrastructure Performance|*|JVM|Memory|Heap|Committed"
          },
          "color": "rgba(0,116,217,1)",
          "label": "Heap Committed (MB)"
        }
      ]
    },
    {
      "title": "Non-Heap Used (MB)",
      "height": 200,
      "width": 400,
      "x": 420,
      "y": 0,
      "widgetType": "GraphWidget",
      "dataSeriesTemplates": [
        {
          "metricMatchCriteria": {
            "applicationName": "WREN-IKP-UAT-103s",
            "metricPath": "Application Infrastructure Performance|*|JVM|Memory|Non-Heap|Used"
          },
          "color": "rgba(0,116,217,1)",
          "label": "Non-Heap Used (MB)"
        }
      ]
    },
    {
      "title": "Non-Heap Committed (MB)",
      "height": 200,
      "width": 400,
      "x": 420,
      "y": 220,
      "widgetType": "GraphWidget",
      "dataSeriesTemplates": [
        {
          "metricMatchCriteria": {
            "applicationName": "WREN-IKP-UAT-103s",
            "metricPath": "Application Infrastructure Performance|*|JVM|Memory|Non-Heap|Committed"
          },
          "color": "rgba(0,116,217,1)",
          "label": "Non-Heap Committed (MB)"
        }
      ]
    },
    {
      "title": "GC Collection Count",
      "height": 200,
      "width": 400,
      "x": 420,
      "y": 440,
      "widgetType": "GraphWidget",
      "dataSeriesTemplates": [
        {
          "metricMatchCriteria": {
            "applicationName": "WREN-IKP-UAT-103s",
            "metricPath": "Application Infrastructure Performance|*|JVM|Garbage Collection|GC Collection Count"
          },
          "color": "rgba(0,116,217,1)",
          "label": "GC Collection Count"
        }
      ]
    },
    {
      "title": "GC Time Spent (ms)",
      "height": 200,
      "width": 400,
      "x": 840,
      "y": 0,
      "widgetType": "GraphWidget",
      "dataSeriesTemplates": [
        {
          "metricMatchCriteria": {
            "applicationName": "WREN-IKP-UAT-103s",
            "metricPath": "Application Infrastructure Performance|*|JVM|Garbage Collection|GC Time Spent (ms)"
          },
          "color": "rgba(0,116,217,1)",
          "label": "GC Time Spent (ms)"
        }
      ]
    }
  ]
}
