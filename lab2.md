# Lab 2 - SNMP

This lab will include instructions for installing and configuring SNMP (Simple Network Management Protocol) and then using it to monitor our Fedora and OPNsense machines.

## Grading

Submit screenshots (including your student number):
  - showing the results of the snmpstatus command using SNMPv1
  - showing the results of the snmpstatus command using SNMPv3
  - showing available SNMP metrics in the Grafana Explore web page

## Install SNMP on Fedora

http://www.net-snmp.org/
http://www.net-snmp.org/wiki/index.php/Tutorials

Update your Fedora machine
    sudo dnf update -y

Install SNMP services and utilities packages

    sudo dnf install net-snmp net-snmp-utils -y

Edit the SNMP daemon configuration

    sudo mv /etc/snmp/snmpd.conf /etc/snmp/snmpd.conf.orig
    sudo su
    echo "rocommunity public" > /etc/snmp/snmpd.conf
    exit

Add a SNMPv3 user using the net-snmp-create-v3-user command

    net-snmp-create-v3-user -a SHA -A password123 -x AES -X password123 <username>

Enable and start the SNMP daemon service

    sudo systemctl enable --now snmpd.service

Verify that SNMP is working

    snmpstatus -v 1 -c public localhost
    snmpstatus -v 3 -a SHA -A password123 -x AES -X password123 -l authPriv -u <username> localhost

## Install SNMP plugin to OPNSense

1. System -> Firmware -> Plugins
2. Install os-net-snmp plugin by clicking the button with the "+" icon
3. Refresh the webpage in your browser

## Configure SNMP on OPNsense

1. Services -> Net-SNMP
2. Enable the SNMP service by clicking the checkbox
3. Click "Save" to save your changes
4. Navigate to the "SNMPv3 Users" tab
5. Click the "+" icon to add a new user
6. Complete the form by adding a username, password and encryption key. Use your own name for the username and password123 for both the password and encryption key.
7. Click "Save" to add the user

## Verify SNMP on Fedora and OPNsense

From your Fedora machine, use the snmpstatus commands as shown below to verify SNMP on Fedora and OPNsense (remember to replace the username and IP address)

Using SNMP v1

    snmpstatus -v1 -c public <ip_address>

Using SNMP v3

    snmpstatus -v 3 -a SHA -A password123 -x AES -X password123 -l authPriv -u <username> <ip_address>

## Setup SNMP Exporter

Download and extract snmp_exporter

    wget https://github.com/prometheus/snmp_exporter/releases/download/v0.21.0/snmp_exporter-0.21.0.linux-amd64.tar.gz
    tar -zxvf snmp_exporter-0.21.0.linux-amd64.tar.gz 
    rm snmp_exporter-0.21.0.linux-amd64.tar.gz 

Start the snmp_exporter service

    cd snmp_exporter-0.21.0.linux-amd64/ 
    ./snmp_exporter > /dev/null 2>&1 &

## Add SNMP exporter to Prometheus scrape config

    cd prometheus-2.44.0-rc.1.linux-amd64
    vi prometheus.yml

Add the below lines to the prometheus.yml under "scrape_configs". See this link for additional clarification: https://github.com/prometheus/snmp_exporter.

  - job_name: 'snmp'
    static_configs:
      - targets:
        - 192.168.1.2  # REPLACE with IP address of OPNsense
    metrics_path: /snmp
    params:
      module: [if_mib]
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: 127.0.0.1:9116  # The SNMP exporter's real hostname:port.

## Start Prometheus and Grafana

    ./prometheus --config.file=prometheus.yml > /dev/null 2&>1 &
    cd ../grafana-9.5.1/
    ./grafana > /dev/null 2&>1 &

## Add Prometheus as a datasource to Grafana

1. Login to Grafana on port 3000 of your Fedora machine using credentials "admin:admin"
2. Navigate to Connections -> Your connections -> Data Sources
3. Click "Add Data Source"
4. Click on Prometheus from the list of data sources
5. Enter "http://localhost:9090" in the HTTP URL then click "Test and Save"
6. Navigate to Explore
7. In the query builder click "select label" and select "job" from the list
8. Click "select value" and select "snmp" from the list
9. Click "select metric" to browse the available SNMP metrics