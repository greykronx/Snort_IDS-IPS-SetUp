# INSTALL_STEPS.md
Step-by-step commands for installing Snort (build from source) and basic configuration.

> **Important**: Run these commands as a user with `sudo` privileges. Always verify package names and availability on your target distribution. Replace `enp0s3`/`eth0` with your actual network interface. See README.md for notes and references. :contentReference[oaicite:10]{index=10}

---

## 0 — Quick checks & update
(Detect distro and update packages)

# detect distro (quick)
cat /etc/os-release

# Debian/Ubuntu/Kali:
sudo apt update && sudo apt -y upgrade

# CentOS/RedHat (yum) or RHEL7:
sudo yum -y update

# For RHEL8+/CentOS8+ you may use dnf:
# sudo dnf -y update


1 — Install build dependencies and packet libraries
(Install compilers, make, pcre, libnet, libpcap, tcpdump, etc.)

# Debian/Ubuntu:
sudo apt -y install build-essential autotools-dev libdumbnet-dev libluajit-5.1-dev \
libpcap-dev libpcre3-dev libdnet libdnet-dev zlib1g-dev pkg-config \
libcap-ng-dev libcap-ng-utils bison flex libnet1 libnet1-dev tcpdump wget

# Kali already has a snort package; to build from source you still need deps:
# sudo apt -y install gcc make libpcap-dev libpcre3-dev libdumbnet-dev bison flex

# CentOS / RHEL (example)
sudo yum -y groupinstall "Development Tools"
sudo yum -y install pcre-devel libpcap-devel libdnet-devel zlib-devel \
libnet-devel bison flex wget tcpdump libnghttp2

2 — Create a workspace for source files
mkdir -p ~/snort_src && cd ~/snort_src

3 — Download & install DAQ (Data Acquisition library)
# Download (example: daq-2.0.7). Verify the current recommended version on snort.org first.
wget https://www.snort.org/downloads/snort/daq-2.0.7.tar.gz
tar -xvzf daq-2.0.7.tar.gz
cd daq-2.0.7/
./configure
make
sudo make install
# run ldconfig after new libs installed
sudo ldconfig
cd ~/snort_src

4 — Download & build Snort from source
# Example (replace version with your chosen release)
# You can find official downloads at https://www.snort.org/downloads
wget https://www.snort.org/downloads/snort/snort-2.9.12.tar.gz
tar -xvzf snort-2.9.12.tar.gz
cd snort-2.9.12/

# Configure, compile and install
./configure --enable-sourcefire
make
sudo make install

# verify snort binary
snort -V
# Output should show version information

Note: If you prefer to install from distro packages, e.g. on Debian/Ubuntu:
sudo apt install snort


5 — Create user/group, directories and set permissions
# create snort group and snort user (system user, no login)
sudo groupadd --system snort
sudo useradd --system -g snort -s /sbin/nologin -c SNORT_IDS snort

# create config / log / dynamic rules directories
sudo mkdir -p /etc/snort/rules
sudo mkdir -p /etc/snort/preproc_rules
sudo mkdir -p /var/log/snort
sudo mkdir -p /usr/local/lib/snort_dynamicrules

# set ownership and permissions
sudo chown -R snort:snort /etc/snort /var/log/snort /usr/local/lib/snort_dynamicrules
sudo chmod -R 5775 /etc/snort /var/log/snort /usr/local/lib/snort_dynamicrules

6 — Copy default configuration files into /etc/snort
# Example path if you built from ~/snort_src/snort-2.9.12
sudo cp ~/snort_src/snort-2.9.12/etc/*.conf /etc/snort/
sudo cp ~/snort_src/snort-2.9.12/etc/*.map /etc/snort/
sudo touch /etc/snort/rules/white_list.rules
sudo touch /etc/snort/rules/black_list.rules
sudo touch /etc/snort/rules/local.rules

7 — Download community rules and extract
# download the community rules archive (name may change—check snort.org)
wget https://www.snort.org/rules/community -O ~/snort_src/community.tar.gz

# extract and copy rules into /etc/snort/rules
tar -xvf ~/snort_src/community.tar.gz -C ~/
# The extracted directory might be 'community-rules' or 'snort3-community-rules' etc.
# Inspect the extracted directory and copy rules:
sudo cp ~/community-rules/* /etc/snort/rules/ || sudo cp ~/snort3-community-rules/* /etc/snort/rules/

8 — Edit /etc/snort/snort.conf
# open with your favourite editor
sudo vim /etc/snort/snort.conf

Make these edits (search inside file):
Network variables — set HOME_NET to your protected network (example):
ipvar HOME_NET 192.168.1.0/24

Rule paths:
var RULE_PATH /etc/snort/rules
var SO_RULE_PATH /etc/snort/so_rules
var PREPROC_RULE_PATH /etc/snort/preproc_rules
var WHITE_LIST_PATH /etc/snort/rules
var BLACK_LIST_PATH /etc/snort/rules

Output plugin (unified2) — example:
output unified2: filename snort.log, limit 128

Include rule files (uncomment/add):
include $RULE_PATH/local.rules
include $RULE_PATH/community.rules

Save and exit.
Pro tip: Use sudo snort -T -c /etc/snort/snort.conf to test the snort.conf for config errors. (See next step.)

9 — Test Snort configuration
sudo snort -T -c /etc/snort/snort.conf

10 — Run Snort in console alert mode (quick live test)
# Example: start snort capturing on interface enp0s3
sudo snort -A console -i enp0s3 -u snort -g snort -c /etc/snort/snort.conf

In another terminal generate test traffic:
# from another host or same host (to your IP)
ping -c 4 <your_server_ip_or_interface_ip>
You should see ICMP alerts in the Snort console if corresponding rules exist.
To stop Snort: Ctrl+C.

11 — Reading logged alerts
If you used unified2 output, parsing is usually done with barnyard2 or other tools. For a quick read of pcap logs:
# If using binary pcap logs (packet logger mode)
snort -r /var/log/snort/snort.log

12 — System services, timezone, NTP & swap (optional items from your notes)
Timezone & NTP
# list timezones
timedatectl list-timezones
# set timezone (example)
sudo timedatectl set-timezone Asia/Kolkata
# install and enable NTP (package name varies)
sudo apt install -y ntp || sudo yum install -y ntp
sudo systemctl start ntpd
sudo systemctl enable ntpd

Swap file (1GB example)
sudo fallocate -l 1G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
# persist across reboots
sudo sh -c 'echo "/swapfile none swap sw 0 0" >> /etc/fstab'

13 — Firewall & OpenVPN (optional)
Your notes include OpenVPN install steps and firewall setup for both CentOS and Debian. Use distribution-specific instructions (e.g., ufw on Debian/Ubuntu, firewalld on CentOS). Example UFW enable:
# Debian/Ubuntu:
sudo apt update
sudo apt install -y ufw
sudo ufw enable
sudo ufw status verbose

CentOS example:
sudo yum install -y firewalld
sudo systemctl start firewalld
sudo systemctl enable firewalld
sudo firewall-cmd --list-all

Extra troubleshooting & tips

If a rule include line is commented out in snort.conf (for example include $RULE_PATH), uncomment only the rules you need.
If community rules extraction names differ (e.g., snort3-community-rules), inspect the extracted folder and copy the contained rules/ directory files into /etc/snort/rules/.
When building from source, missing headers will show up during ./configure or make — install the corresponding -dev packages and retry.
Always test snort -T -c /etc/snort/snort.conf after changing snort.conf.

References
Snort official downloads & docs. 
Snort community rules and MD5 sums. 
Practical install guides (Ubuntu / CentOS examples). 
