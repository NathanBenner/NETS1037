
# Lab 9 - RADIUS AAA (Authentication, Access and Accounting) 

This lab will include instructions for deploying a RADIUS server and configuring SSH to authenticate using the RADIUS server.

OPNsense has documentation on FreeRadius found here: https://docs.opnsense.org/manual/how-tos/freeradius.html.

## Grading

Submit screenshots (including your student number) of the following commands and their output after completing the lab:
- radtest testuser radiuspassword localhost 1 testing123
- radtest testuser badpass localhost 1 testing123
- radtest testuser password localhost 1 badsecret
- ssh testuser@localhost
- sudo tail -n 20 /var/log/secure

## Install the FreeRADIUS plugin to OPNsense

1. Login to OPNsense
2. Navigate to System > Firmware > Plugins
3. Search for os-freeradius, then click the "+" to install it.

Note: You may need to install OPNsense updates before you can install plugins.

## Enable the FreeRadius Service

1. Refresh your browser
2. Navigate to Services > FreeRADIUS > General
3. Check the boxes for "Enable" and "Log Authentication Request" then click "Save"

## Add a user to RADIUS

1. Navigate to Services > FreeRADIUS > Users
2. Click on the "+" to add a new user
3. Give the new user the username "testuser" and password "password" then click "Save"

## Add a client to RADIUS

1. Navigate to Services > FreeRADIUS > Clients
2. Click on the "+" to add a new client
3. Give the new client the name "LAN", secret "testing123" and your LAN network CIDR (e.g. 192.168.1.1/24) then click "Save"
4. Click "Apply"

## Test the RADIUS Authentication

1. Install the freeradius-utils package on Fedora

        sudo dnf install -y freeradius-utils

2. Test the RADIUS authentication by running the below command

        radtest testuser password <opnsense_ip_address> 1 testing123

3. Navigate to Services > FreeRADIUS > Log File. Check the log for the authentication request
4. Use radtest to try to authenticate using the wrong password. What is recorded in the log file?

## Configure SSHD on Fedora to use RADIUS authentication

1. Install the pam_radius package on Fedora

    sudo dnf install -y pam_radius

2. Edit /etc/pam_radius.conf to configure PAM to use the FreeRadius on OPNsense. Scroll to the bottom of the file, delete or comment out the existing lines then add your own line with the IP address of OPNsense and the radius shared secret (testing123) as shown below.

        # server[:port]                 shared_secret       timeout (s)     source_ip       vrf
        <opnsense_ip_address>           testing123          3
        #127.0.0.1                      secret3             3
        #other-server...
        #[2001:0db8:85a3::4]:1812...
        #other-other-server...

3. Edit /etc/pam.d/sshd to configure SSHD to use pam_radius. Do this by adding the below line at the top of the file. The top 4 lines of your file should look like below.

        #%PAM-1.0
        auth    sufficient      pam_radius_auth.so
        auth    substack        password-auth
        auth    include         postlogin
        ...

4. Add a user named testuser to the Fedora Machine and give it the password "password123"

        sudo useradd testuser
        sudo passwd testuser

## Test the effect of using RADIUS with SSH on other methods of login (console)

1. Login on the console of Fedora as user testuser
- Try the password in radius (password)
- Try the linux password (password123)
- Try an invalid password
2. Which password(s) was/were accepted?
3. What was logged to /var/log/secure?

## Restrict SSHD to use only RADIUS authentication

1. Disable standard UNIX authentication by editing /etc/pam.d/sshd and commenting out the line "auth substack password-auth". The top 4 lines of your file should look like below.

        #%PAM-1.0
        auth    sufficient      pam_radius_auth.so
        #auth    substack        password-auth
        auth    include         postlogin
        ...

2. Use ssh to connect to Fedora as user testuser ("ssh testuser@localhost")
- Try the password in radius (password)
- Try the linux password (password123)
- Try an invalid password
3. Which password(s) was/were accepted?
4. What was logged to the /var/log/freeradius/radius.log and /var/log/auth.log files?
