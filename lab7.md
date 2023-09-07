
# Lab 7 - Web Proxy

This lab will include instructions for configuring a caching web proxy using OPNsense. We will also create an reverse web proxy using Nginx.

OPNsense has documentation on setting up a caching proxy, found here: https://docs.opnsense.org/manual/how-tos/cachingproxy.html#enable-disable.

## Grading

Submit screenshots (including your student number):
  - showing your command testing your web proxy
  - showing the request log in the web proxy access log
  - showing your Nginx configuration file
  - showing your Nginx reverse proxy in your web browser

## Enable the Web Proxy Service

1. Login to OPNsense
2. Navigate to Services > Web Proxy > Administration
3. Check the box for "Enable Proxy" then click "Save"
4. Click on the dropdown icon next to "General Proxy Settings" then select "Local Cache Settings" from the dropdown
5. Check the box for "Enable local cache" then click "Apply"
6. Restart the web proxy service using the restart icon in the top-right of the web page

## Test the Web Proxy Service

From your Fedora machine, run the below command to test your proxy. If your proxy is working, the command will print your public IP address.

    http_proxy=http://<opnsense_ip_address>:3128 curl icanhazip.com

You can further verify that the proxy is working by navigating within OPNsense to Services > Web Proxy > Access Log and checking that there is a log message for your request.

## Configure Nginx Reverse Proxy

1. Install Nginx on your Fedora machine using the below command

    sudo dnf install nginx -y

2. Create the reverse proxy configuration file under the /etc/nginx/conf.d/ directory

    sudo vi /etc/nginx/default.d/reverse_proxy.conf

Then add the below contents to the configuration file

    location / {
        proxy_set_header Host $http_host;
        proxy_pass http://localhost:3000;
    }

3. Enable the Nginx Service

    sudo systemctl enable --now nginx

4.  Configure SELinux to allow Nginx to connect to the network

    setsebool -P httpd_can_network_connect 1

## Test Nginx Reverse Proxy

Start your Grafana service then, from your Fedora machine open Firefox and navigate to http://localhost/. If your Nginx reverse proxy is working properly, you will be see your Grafana webpage.
