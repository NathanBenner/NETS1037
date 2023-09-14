
# Lab 3 - Log Collection

This lab will include instructions for configing log collection using Rsyslog, Promtail and Loki.

## Grading

Submit screenshots showing:
  - OPNsense log forwarding configuration
  - Log data in Grafana Explore web page

## Install and Configure Loki and Promtail

Create a directory for the configuration files and executables

    mkdir loki
    cd loki

Download and extract Loki

    wget https://github.com/grafana/loki/releases/download/v2.8.2/loki-linux-amd64.zip

    unzip loki-linux-amd64.zip
    rm loki-linux-amd64.zip

Download an example Loki configuration file

    wget https://raw.githubusercontent.com/grafana/loki/main/cmd/loki/loki-local-config.yaml

Then start the loki service

    ./loki-linux-amd64 -config.file=loki-local-config.yaml

Download and extract Promtail
   
    wget https://github.com/grafana/loki/releases/download/v2.8.2/promtail-linux-amd64.zip

    unzip promtail-linux-amd64.zip
    rm promtail-linux-amd64.zip

Download an example Promtail configuration file
    wget https://raw.githubusercontent.com/grafana/loki/main/clients/cmd/promtail/promtail-local-config.yaml

Then start the promtail service

    ./promtail-linux-amd64 -config.file=promtail-local-config.yaml

## Start Grafana

    cd ../grafana-9.5.1/
    ./bin/grafana-server

## Add Loki to Grafana as a data source

1. Login to Grafana on port 3000 of your Fedora machine using credentials "admin:admin"
2. Navigate to Connections -> Your connections -> Data Sources
3. Click "Add Data Source"
4. Click on Loki from the list of data sources
5. Enter " http://localhost:3100" in the HTTP URL then click "Test and Save"
6. Navigate to Explore
7. In the query builder click "select label" and select "job" from the list
8. Click "select value" and select "varlogs" from the list
9. Click "Run Query" to browse the available logs

# Enable Remote Log Collection

## Enable Promtail syslog input

Make a copy of your promtail configuration file

    cp promtail-local-config.yml promtail-remote-config.yml

Edit the promtail-remote-config.yml file to add the "syslog" job under "scrape_configs" as shown below.

## promtail.yml
---
server:
  http_listen_port: 9080
  grpc_listen_port: 0

positions:
  filename: /tmp/positions.yaml

clients:
  - url: http://127.0.0.1:3100/loki/api/v1/push

scrape_configs:

  - job_name: syslog
    syslog:
      listen_address: 0.0.0.0:1514
      labels:
        job: "syslog"
    relabel_configs:
      - source_labels: ['__syslog_message_hostname']
        target_label: 'host'

  - job_name: system
    static_configs:
    - targets:
        - localhost
      labels:
        job: varlogs
        __path__: /var/log/*log

# end promtail.yml

Stop promtail using control-C then start Promtail using the remote config

    ./promtail-linux-amd64 -config.file=promtail-remote-config.yaml

## Install and configure Rsyslog

    sudo dnf install rsyslog

Edit the /etc/rsyslog.conf file to enable receiving logs using TCP by uncommenting the following lines:

    module(load="imtcp") # needs to be done just once
    input(type="imtcp" port="514")

Edit the /etc/rsyslog.conf file to enable forwarding logs to Promtail by adding the following line to the bottom of the file:

    *.* action(type="omfwd" protocol="tcp" target="127.0.0.1" port="1514" Template="RSYSLOG_SyslogProtocol23Format" TCP_Framing="octet-counted" KeepAlive="on")

Set SELinux to Permissive mode

    sudo setenforce 0

Enable and start Rsyslog

    sudo systemctl enable --now rsyslog

## Configure OPNsense log Forwarding

1. Navigate to System -> Settings -> Logging/targets
2. Click the "+" icon to add a log forwarding target
3. Select TCP(4) for Transport
4. Enter the IP address of Fedora for Hostname
5. Check the box for RFC5424
6. Click "Save"

## Allow TCP Port 514 through the Fedora firewall

Run the below commands on your Fedora machine

  sudo firewall-cmd --add-port 514/tcp --permanent
  sudo firewall-cmd --reload

## Verify OPNsense Remote Logging

1. Login to Grafana on port 3000 of your Fedora machine using credentials "admin:admin"
2. Navigate to Explore
3. In the query builder click "select label" and select "host" from the list
4. Click "select value" and select the hostname or IP of your OPNsense machine from the list
5. Click "Run Query" to browse the available logs

## Configure Grafana Agent on Windows VM

On your Windows VM, download the Grafana Agent installer using this link: https://github.com/grafana/agent/releases/latest/download/grafana-agent-installer.exe.zip.
Unzip the installer and then run it to install Grafana Agent.

Verify that the Grafana Agent is running by navigating to http://localhost:12345/-/healthy and http://localhost:12345/agent/api/v1/metrics/targets.

Edit the Grafana Agent configuration file to enable remote write of metrics to Prometheus and log forwarding to Loki. The configuration file can be found at C:\Program Files\Grafana Agent\agent-config.yaml.

Replace the configuration with the below configuration:

### C:\Program Files\Grafana Agent\agent-config.yaml
server:
  log_level: warn
metrics:
  wal_directory: C:\ProgramData\grafana-agent-wal
  global:
    scrape_interval: 1m
    remote_write:
      - url: http://<Fedora_VM_IP>:9090/api/prom/push
  configs:
    - name: integrations
integrations:
  windows_exporter:
    enabled: true
logs:
  configs:
  - name: default
    positions:
      filename: /tmp/positions.yaml
    scrape_configs:
      - job_name: windows
        windows_events:
          use_incoming_timestamp: false
          bookmark_path: "./bookmark.xml"
          eventlog_name: "Application"
          xpath_query: '*'
          labels:
            job: windows
        relabel_configs:
          - source_labels: ['computer']
            target_label: 'host'
    clients:
      - url: http://<Fedora_VM_IP>:3100/loki/api/v1/push

### end C:\Program Files\Grafana Agent\agent-config.yaml

After editing the Grafana Agent configuration file, restart the Grafana Agent service or reboot your Windows VM to apply the changes.

## Verify Windows Remote Logging

1. Login to Grafana on port 3000 of your Fedora machine using credentials "admin:admin"
2. Navigate to Explore
3. In the query builder click "select label" and select "host" from the list
4. Click "select value" and select the hostname or IP of your Windows VM from the list
5. Click "Run Query" to browse the available logs
