# Cuckoo Debian Install  
# How to Build a Debian Cuckoo Sandbox Malware Analysis System
## Dynamic Malware Analysis Sandbox
> https://github.com/cuckoosandbox/cuckoo

A complete Cuckoo install guide on Debian host.
  * Debian 10.7
  * Virtualbox 6.0
  * optional win7 guest ova
    * Box in a Box
  * building Volatility from source
  * building Gaucamole 1.2 from source
  * using venv for cuckoo
  
# Debian Host setup  
  * Download Debian 10.7 iso
    * https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/debian-10.6.0-amd64-netinst.iso 
  * Create a VM or use on phyiscal machine
    * I used ESXI and VMware workstation Pro for host hypervisor 
    * Optional settings  
      I used these settings for this guide.
    - 60GB storage Minimum 
      * Recommend 80-100GB
    - 8Gb RAM/Memory
    - 1CPU w/ 4cores
    - Guided partitioning w/ seperate /home /var and /tmp partitions.
    - Software
      * ssh server
      * kde desktop
      * standard system utilities   
     - ESXI and Vmware need "virtualize intel vt-x" enabled
        * we will be running vbox inside host
      
> following steps as Root   
# Installing Python libraries
` sudo apt-get install python python-pip python-dev libffi-dev libssl-dev`   
` sudo apt-get install python-virtualenv python-setuptools`   
` sudo apt-get install libjpeg-dev zlib1g-dev swig`   

# Install MongoDB
` wget -qO - https://www.mongodb.org/static/pgp/server-4.4.asc | sudo apt-key add -`   

` echo "deb http://repo.mongodb.org/apt/debian buster/mongodb-org/4.4 main" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.4.list`   

` sudo apt-get update`   
` sudo apt-get install -y mongodb-org`   
` sudo systemctl start mongod`  
` sudo systemctl enable mongod`  

#  Install PostgreSQL
` sudo apt-get install postgresql libpq-dev`   

# Install PyDeep
` sudo apt-get install libfuzzy-dev`  
` wget https://github.com/kbandla/pydeep/archive/master.zip -O pydeep-master.zip`   
` unzip pydeep-master.zip`   
` cd pydeep-master/`  
` chmod +x -R *`   
` python ./setup.py build`   
` python ./setup.py test`   
` python ./setup.py install`   
` cd ~`   

# Install VirtualBox 6.0
 ` echo "deb https://download.virtualbox.org/virtualbox/debian  buster contrib" | sudo tee /etc/apt/sources.list.d/virtualbox.list`    

` wget -q https://www.virtualbox.org/download/oracle_vbox_2016.asc -O- | sudo apt-key add -`     
` wget -q https://www.virtualbox.org/download/oracle_vbox.asc -O- | apt-key add -`  
` sudo apt update`   
` sudo apt install virtualbox-6.0` 
  ### Install 6.0 extension pack
  * https://download.virtualbox.org/virtualbox/6.0.24/Oracle_VM_VirtualBox_Extension_Pack-6.0.24.vbox-extpack
    * Downloaded via Browser and doubled clicked. Nothing fancy.
    * good way verify virtualbox launches correctly
    * verify in vbox gui that you have a vboxnet0 network. If not, create it.
      * 192.168.56.1/24
      
# Create a cuckoo user
` sudo adduser cuckoo`   
` sudo usermod -a -G vboxusers cuckoo`  

# Installing TCPdump
` sudo apt-get install tcpdump apparmor-utils`   
` sudo aa-disable /usr/sbin/tcpdump`   
  > Debian 10.6 aa-disable will show a error /usr/sbin/tcpdump not found. Just ignore.

` sudo groupadd pcap`   
` sudo usermod -a -G pcap cuckoo`   
` sudo chgrp pcap /usr/sbin/tcpdump`  
` sudo setcap cap_net_raw,cap_net_admin=eip /usr/sbin/tcpdump`   
#### You can verify the results of the last command with:   
` getcap /usr/sbin/tcpdump`   
> /usr/sbin/tcpdump = cap_net_admin,cap_net_raw+eip

# Install Volatility
> https://github.com/volatilityfoundation/volatility   

` wget https://github.com/volatilityfoundation/volatility/archive/master.zip -O volatility-master.zip`   
` unzip volatility-master.zip`   
` cd volatility-master/`   
` chmod +x -R *`   
` python ./setup.py build`   
` python ./setup.py test`   
` python ./setup.py install`   
` cd ~`   
` sudo pip install distorm3==3.4.4`
 
# Installing M2Crypto
` sudo apt install python-m2crypto`    

# Install Guacamole
` sudo apt install build-essential libcairo2-dev libjpeg62-turbo-dev libpng-dev libtool-bin libossp-uuid-dev libvncserver-dev freerdp2-dev libssh2-1-dev libtelnet-dev libwebsockets-dev libpulse-dev libvorbis-dev libwebp-dev libssl-dev libpango1.0-dev libswscale-dev libavcodec-dev libavutil-dev libavformat-dev`   
> watch for 404 errors when installing serveral packages and keep trying till they resolve   

`mkdir /tmp/guac-build && cd /tmp/guac-build`   
`wget https://downloads.apache.org/guacamole/1.2.0/source/guacamole-server-1.2.0.tar.gz`   
`tar -zxvf guacamole-server-1.2.0.tar.gz`   
`cd guacamole-server-1.2.0`   
`./configure --with-init-dir=/etc/init.d`   
`make && sudo make install && cd ..`   
`sudo ldconfig`   
`sudo /etc/init.d/guacd start`   
` cd ~`   
`systemctl enable guacd`

> Root tasks are finished. reboot and switch to cuckoo user

# Install Cuckoo 
  * as cuckoo user   
  ` virtualenv venv`   
  ` . venv/bin/activate`   
  (venv)` pip install -U pip setuptools`   
  (venv)` pip install -U cuckoo`   
  * for cuckoo agent to work   
  (venv)` pip install pip==9.0.3`     
  * first run    
  (venv)` cuckoo`   
  * install cuckoo community rules   
  (venv)` cuckoo community`   
  > hopefully cuckoo does intialize and shows welcome message. This will create the initial conf files.    
  ### Volatility for venv   
  > as root    
  * copy vol to venv   
  `sudo cp -r /usr/local/lib/python2.7/dist-packages/volatility-2.6.1-py2.7.egg/volatility/ /home/cuckoo/venv/lib/python2.7/`  
  
  > as cuckoo   
  * confirm distorm3 is installed in venv for vol   
  (venv) `pip install distorm3==3.4.4`   
    
  * Don't start cuckoo yet. Will still have a bit more to do. 

# Import ova Cuckoo Guest machine with VirtualBox   
  ### I created a Windows 7 guest if you wanted the easy way.   
  > cuckoo1.ova   
  > https://mega.nz/file/9AB1XQJb#MZbHNuh3ofmqXQfpQwOg_YDv0lIuWywvAVSZGcgCvuY   
  > Hash: SHA256 E:\cuckoo1.ova          
  > D927696A69B095D82B0E26D20B8786A8BB823E57200A95F3FDAB0B834AD0CEFD           
   * logged in as a cuckoo user
     * Import it to Vbox
   * may prompt for reboot for finishing IE install
   * Verify machine is on Host-Only Network Adapter
   * Start once, wait for the desktop to load
   * cmd as admin, use `slmgr -rearm` command to restart evaluation period. reboot.
   *  wait for the desktop to load
   * Create a snapshot while running named "Snapshot1"   
      * cuckoo requires running snapshot
   * About this ova:
     * Windows 7 non-licensed. Office 2010 non-licensed. Adobe. IE11. Python2.7. Enough windows updates to make it stable. 
     * Default static IP 192.168.56.101 
       > which is the default IP in cuckoo conf
     * It's pre-configured as needed by cuckoo
     * auto-login
     * Cuckoo agent installed, and agent configured to start automatically
     * Pillow installed for screenshots
     * 1CPU/1core, 2Gb RAM, 32Gb storage/16GB used
     * Volatility Suggested Profile(s) : Win7SP1x64

# OR Create Cuckoo Guest
  * best instructions I found.   
    >  https://gist.github.com/braimee/bf570a62f53f71bad1906c6e072ce993#build-your-base-windows-vm   
  * I would recommend using 192.168.56.101 as static IP. Unless you like rabbit holes. 

  
# Create iptables pre-cuckoo-start script
  * The idea is that the guest machine is on a Host-Only network adapter (no internet). Although, we need the machine to have network access. 
  * We create a simple bash script to run as root. This will forward and allow traffic to pass from guest Through Host(cuckoo) eventually to the internet. 
  * purpose to capture all the packets and get a good analysis.  
  * copy and paste the following as `iptables-cuckoo-start.sh`     
    `iptables -t nat -A POSTROUTING -o ens33 -s 192.168.56.0/24 -j MASQUERADE`    
    `# Default drop.`   
    `iptables -P FORWARD DROP`   
    `# Existing connections.`   
    `iptables -A FORWARD -m state --state RELATED,ESTABLISHED -j ACCEPT`   
    `# Accept connections from vboxnet to the whole internet.`   
    `iptables -A FORWARD -s 192.168.56.0/24 -j ACCEPT`   
    `# Internal traffic.`   
    `iptables -A FORWARD -s 192.168.56.0/24 -d 192.168.56.0/24 -j ACCEPT`   
    `# Log stuff that reaches this point (could be noisy).`   
    `iptables -A FORWARD -j LOG`   
    `iptables -A FORWARD -o ens33 -i vboxnet0 -s 192.168.56.0/24 -m conntrack --ctstate NEW -j ACCEPT`   
    `iptables -A FORWARD -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT`   
    `iptables -A POSTROUTING -t nat -j MASQUERADE`   
    `sysctl -w net.ipv4.ip_forward=1`   
  * Verify Network adapter alias(ens33) with `ip a` change as needed. 
  * Will need to run the script as root after every reboot. There are ways to make it permanent and the iptables rules are not perfect but will work. 
  >  A really good test to perform is to start the guest VM. It should show network connectivity after you run the script.  

# Edit Cuckoo conf's
  * Must enable MongoDB before run
    * EDIT: `/home/cuckoo/.cuckoo/conf/reporting.conf`
      * Find line: `[mongodb]`
      * below set `enabled = yes`
  * Must enable remote control if you plan to use Guacamole
    * EDIT: `/home/cuckoo/.cuckoo/conf/cuckoo.conf`
      * Find Line: `[remotecontrol]` (Near bottom)
      * below set `enabled = yes`
  * Must enable memory dump if you plan to use volatility
    * EDIT: `/home/cuckoo/.cuckoo/conf/cuckoo.conf`
      * Find line: `memory_dump = no`
      * set `memory_dump = yes`
    * EDIT: `/home/cuckoo/.cuckoo/conf/processing.conf`
      * Find Line: `[memory]`
      * below set `enabled = yes`
  * Must set volatility image profile
    > if you used my ova win7 image 
    * EDIT: `/home/cuckoo/.cuckoo/conf/memory.conf`
      * Find Line: `[basic]`
      * below set `guest_profile = Win7SP1x64`
      
# Fix Gaucamole Cuckoo remote console
* Guacamole shows a conneciton error during testing. This would be a temp fix till someone finds a better solution. 
> https://github.com/cuckoosandbox/cuckoo/issues/2787#issuecomment-630522999
  * we used venv. so, the path is a bit different. 
  * EDIT: `/home/cuckoo/venv/local/lib/python2.7/site-packages/cuckoo/web/analysis/urls.py`
  * below line 7 add+: `from django.views.decorators.csrf import csrf_exempt`
  * Comment now line 27 with `#` at the beginning
  * below line 27 add+: `url(r"^(?P<task_id>\d+)/control/tunnel/.*", csrf_exempt(ControlApi.tunnel), name="analysis/control/tunnel"),`
  * Comment now line 52 with `#` at the beginning
  * below line 52 add+: `url(r"^api/tasks/info/$", csrf_exempt(AnalysisApi.tasks_info)),`
  * save and close
  * you should see some simulaitries with the lines. 

# Activate venv and start Cuckoo
  * as cuckoo user, open terminal and activate venv and start cuckoo with debug enabled.   
    `. venv/bin/activate`   
    (venv)` cuckoo -d`   
    > it should start with cuckoo news and no errors.  
    
  * open another terminal and start cuckoo webserver  
    `. venv/bin/activate`   
    (venv)` cuckoo web runserver 0.0.0.0:8000`   
    
  * open another terminal and start cuckoo api   
    `. venv/bin/activate`   
    (venv)` cuckoo api --host 0.0.0.0 --port 8090`   
  * open browser on local machine(host) and go to `http://127.0.0.1:8000`   
  * Should display Cuckoo news and Guest VM stats   
  
# Testing Cuckoo
  * malware samples at https://github.com/fabrimagic72/malware-samples to test the new cuckoo server
  * Password on zip files is 'infected' becareful make snapshot of host.
  * download and upload to cuckoo using host make snaphot of host before

# Final thoughts
  * **After every reboot of host**
    * run `iptables-cuckoo-start.sh` as root
    * you need to run virtualbox and guest at least once after every startup
       * cuckoo needs vboxnet0 to be up or will fail and shows errors on starting cuckoo.
        * Make sure you can ping virtualbox net gateway. 192.168.56.1
       > never did find a service you can just start for vboxnet0
       * launch win7 guest and verify it has internet access
        * from host make sure you can ping guest
          * 192.168.56.101
       * you need to activate venv before starting Cuckoo
        * before starting Cuckoo, Cuckoo Webserver, and Cuckoo API
  * you may want to use anything but your own work or home Network connection. 
      * use DMZ, VPS, or VPN connection
  * I would imagine some malware would create problems
      * This malware would probably require static analysis
      * Also be a good idea to submit and share the samples with the community 
      
 # More information 
   https://cuckoo.readthedocs.io/en/latest/installation/   
   https://cuckoo.sh/docs/installation/host/installation.html   
   https://gist.github.com/braimee/bf570a62f53f71bad1906c6e072ce993#file-mostly_painless_cuckoo_sandbox_install-md   
   https://blog.nviso.eu/2018/04/12/painless-cuckoo-sandbox-installation/   
   
