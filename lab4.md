
# Log Analysis Lab

In this lab we will be creating a Grafana dashboard for analyzing log data. We will create 2 panels to display the size of our log data and the growth rate of our log data over time. We will then use the log browser in Grafana to identify some interesting log messages and then create 2 more panels to display the interesting log messages and the rate at which they are received.

## Grading

Submit your queries for each panel and a screenshot showing your Grafana dashboard.

## Resources

https://grafana.com/docs/loki/latest/logql/
https://prometheus.io/docs/prometheus/latest/querying/basics/
https://grafana.com/docs/grafana/latest/panels-visualizations/

# Modify the Prometheus configuration to gather metrics from Loki

Open your prometheus.yml (or prometheus.yaml) for editing and add the below lines to enable gathering of metrics from Loki.

  - job_name: 'loki'
    static_configs:
      - targets: ["localhost:3100"]

Once you have made the above change to the configuration file, you will need to restart your Prometheus service.

# Create a Dashboard

Ensure that your Grafana, Prometheus, Loki and Promtail services are running and configured following the instructions in the previous labs.
Login to Grafana. Remember that Grafana is on port 3000 of your Fedora machine and that the default credentials are admin:admin.
In Grafana, Navigate to the Dashboards page. Click the button for "New" then select "New Dashboard" to create a dashboard.
Click the floppy disk icon in the top bar to save the dashboard. Give your dashboard a name, then click Save.

# Create a Panel to show how much log data is stored in Loki

From your Grafana dashboard, click the "Add" button from the top row, then select "Visualization" from the drop down to create your panel.
Select Prometheus as the data source. Then use the query builder to create a query to display the metric "loki_log_messages_total".
Give your panel a title and then click "Save".

# Create a Panel to show the rate of growth for log data in Loki

From the dashboard view, duplicate the panel you just created and then edit the duplicate panel.
In the query builder, click "+ Operations" then select "Rate" from under "Range Functions". Select a range of one minute for the rate operation.
Give your panel a title then save it.

# Create a panel for an interesting log message

Create a new panel in your dashboard.
Select Loki as the datasource and change the visualization type from "Time Series" to "Logs". Then use the query builder to find one or more log messages that are interesting to security. You will then create a query that will show only these log messages using labels and filters.

# Create a panel to show the rate of your interesting log message

From the dashboard view, duplicate the panel you just created and then edit the duplicate panel.
Change the visualization type to "Time Series".
In the query builder, click "+ Operations" then select "Rate" from under "Range Functions". Select a range of one minute for the rate operation.
Give your panel a title then save it.
