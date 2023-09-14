
# Lab 1 - Setup

This lab will include instructions for creating a lab environment for this course. This lab requires that you have a hypervisor such as VirtualBox, VMware or HyperV installed for creating virtual machines. The lab environment will consist of the following components:
- OPNsense Firewall
- Fedora Workstation
- Windows 10

The Fedora machine will be our monitoring host for monitoring the Windows 10 and OPNsense machines, as well as itself. On the Fedora machine we will deploy our monitoring tools (collectively called the Grafana Stack), consisting of:
- Grafana (monitoring front-end)
- Prometheus (monitoring system)
- Loki (log aggregation system) *** Deployed in lab 3 ***

## Grading

Submit screenshots (including your student number) proving that you have successfully deployed OPNsense, Fedora, Windows 10, Grafana and Prometheus. One screenshot each is sufficient.

# OPNsense

Download OPNsense using this link: https://opnsense.org/download/

Select the below options then click 'Download'.

- Architecture: amd64
- Image Type: dvd
- Mirror Location: United States > LeaseWeb (East Coast)
The downloaded ISO file will be compressed using bzip. You will need to unzip the file using a program such as 7zip which can be downloaded here: https://www.7-zip.org/download.html.

Create a VM for a router with two ethernets

1. Connect the first interface to the VMWare NAT network (automatic setting can be used) - this will be used as the wan interface
2. Connect the second one to the private network you created in the previous steps to be used as the LAN interface

Install OPNsense

1. Start your VM in the VMWare virtual machine library window
2. Select to install using the Opnsense ISO downloaded from opnsense.org
3. Configure the router wan connection as a DHCP client
4. On the console after boot, configure the router LAN to have a static IP using host number 2 on whatever network ip address block VMWare assigned to your private LAN and have the router provide DHCP service to the LAN
5. On the console, login as installer using username "installer" and password "opnsense"
6. Continue through the installation wizard
7. Connect to your Opnsense router’s LAN ip using a web browser on your host laptop
8. Login as root with password opnsense
9. Click through the initial setup wizard. Set the domain to home.arpa
10. Run the updates when offered, and let it reboot

Install Updates

1. In the web console, navigate to System -> Fireware -> Status, then click "Check for Update".
2. Close the popups and then scroll to the bottom of the page and click "Update".

# Prometheus

Download node_exporter to your Fedora machine

    wget https://github.com/prometheus/node_exporter/releases/download/v1.5.0/node_exporter-1.5.0.linux-amd64.tar.gz
    tar -zxvf node_exporter-1.5.0.linux-amd64.tar.gz
    rm node_exporter-1.5.0.linux-amd64.tar.gz

Then start the node_exporter service

    cd node_exporter-1.5.0.linux-amd64/
    ./node_exporter

When you run node_exporter locally, navigate to http://localhost:9100/metrics to check that it is exporting metrics.

Download Prometheus to your Fedora machine

    wget https://github.com/prometheus/prometheus/releases/download/v2.44.0-rc.1/prometheus-2.44.0-rc.1.linux-amd64.tar.gz
    tar -zxvf prometheus-2.44.0-rc.1.linux-amd64.tar.gz
    rm prometheus-2.44.0-rc.1.linux-amd64.tar.gz

Start the Prometheus service

    cd prometheus-2.44.0-rc.1.linux.amd64
    ./prometheus --config.file=./prometheus.yml 

Confirm that Prometheus is running by navigating to http://localhost:9090.

# Grafana

Download and extract Grafana to your Fedora machine

    wget https://dl.grafana.com/oss/release/grafana-9.5.1.linux-amd64.tar.gz
    tar -zxvf grafana-9.5.1.linux-amd64.tar.gz
    rm grafana-9.5.1.linux-amd64.tar.gz

Then start Grafana Server in the background

    cd grafana-9.5.1
    ./bin/grafana-server 
