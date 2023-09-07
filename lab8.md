
# Lab 8 - Web Filtering

This lab will include instructions for configuring web filtering using OPNsense and ClamAV.

OPNsense has documentation on webfiltering, found here: https://docs.opnsense.org/manual/how-tos/proxywebfilter.html and here: https://docs.opnsense.org/manual/how-tos/proxyicapantivirusinternal.html.

## Grading

Submit screenshots (including your student number):
  - showing that you have enabled ClamAV and C-ICAP services
  - showing the test of the web filter using the eicar file
  - showing the request log in the web proxy access log

## Install the ClamAV and c-icap plugins to OPNsense

1. Login to OPNsense
2. Navigate to System > Firmware > Plugins
3. Search for os-clamav, then click the "+" to install it.
4. Search for os-c-icap, then click the "+" to install it.

Note: You may need to install OPNsense updates before you can install plugins.

## Enable the ClamAV Service

1. Refresh your browser
2. Navigate to Services > ClamAV > Configuration
3. Click the button for "Download signatures"
4. Check the boxes for "Enable clamav service" and "Enable freshclam service" then click "Save"

## Enable the C-ICAP Service

1. Navigate to Services > C-ICAP > Configuration
2. Check the box for "Enable c-icap service" then click "Save"
2. Click the tab for "Antivirus"
4. Check the box for "Enable ClamAV" then click "Save"

## Configure Web Proxy ICAP settings

1. Navigate to Services > Web Proxy > Administration
2. Click on the dropdown icon next to "Forward Proxy" then select "ICAP Settings" from the dropdown
3. Check the box for "Enable ICAP" then click "Apply"

## Test the Web Filtering

From your Fedora machine, run the below command to test the web filtering of your proxy.

    http_proxy=http://<opnsense_ip_address>:3128 curl http://www.csm-testcenter.org/cgi-bin/eicar.txt

If your webfiltering is working correctly, the above command will print the HTML of a web page warning you that you tried to download malware. Adding a grep for "MALWARE" as shown below should print out a line containing "<h1>MALWARE FOUND<h1>"

  http_proxy=http://<opnsense_ip_address>:3128 curl http://www.csm-testcenter.org/cgi-bin/eicar.txt | grep MALWARE

More information about eicar files can be found here: https://www.eicar.org/download-anti-malware-testfile/.
