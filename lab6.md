# Lab 6 - SNMP Traps

This lab will include instructions for configuring the snmptrapd daemon to listen for and log SNMP traps.

## Grading

Submit screenshots (including your student number):
  - showing the contents of /etc/snmp/snmptrapd.conf
  - showing that snmptrapd is listening on port 162/udp
  - showing that snmptrapd is receiving trap messages
  - showing the SNMP Trap messages in Grafana
  - showing the Loki query for your SNMP trap messages

## Start Grafana, Loki and Promtail

The services can be run in a single terminal as shown below:

    ./loki-linux-amd64 -config.file=loki-local-config.yaml 
    ./promtail-linux-amd64 -config.file=promtail-remote-config.yaml 
    ./bin/grafana-server 

or each service can be run in it's own terminal as shown below. This method will print application logs to the terminal and is useful for troubleshooting errors.

    ./loki-linux-amd64 -config.file=loki-local-config.yaml
    ./promtail-linux-amd64 -config.file=promtail-remote-config.yaml
    ./bin/grafana-server

## Ensure that the net-snmp package is installed

    sudo dnf install net-snmp -y

## Configure snmptrapd

Edit the /etc/snmp/snmptrapd.conf and uncomment the line starting with "authCommunity". This will enable receiving traps with a matching community string such as "public".

Enable the snmptrapd service

    sudo systemctl enable --now snmptrapd

## Verify that snmptrapd is running

Verify that the snmptrapd daemon is running

    systemctl status snmptrapd

Verify that port 162/udp is listening

    ss -ulpn

## Verify SNMP traps are being received by snmptrapd

Send an snmptrap

    sudo snmptrap -v 1 -c public localhost '' '' 3 0 ''

and verify that it was received

    journalctl -u snmptrapd

## Verify SNMP trap messages are being received by Loki

1. Log into Grafana and navigate to the Explore page
2. Select Loki as the datasource
3. Use the query builder to build a query for only the SNMP trap messages
