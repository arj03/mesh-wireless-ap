# mesh-wireless-ap

1 extra wireless dongle: Edimax EW-7612UAn v2. Works with raspbian lite out of the box.

- wlan0 = internal raspberry pi wifi (access point)
- wlan1 = external wifi (connected to router)

## network

To make sure only wlan1 connects to internet router.

/etc/network/interfaces

```
allow-hotplug wlan0
iface wlan0 inet static
address 192.168.4.1
netmask 255.255.255.0
wireless-power off

allow-hotplug wlan1

iface wlan1 inet manual                
    wpa-roam /etc/wpa_supplicant/wpa_supplicant.conf

iface router inet dhcp          
```

/etc/wpa_supplicant/wpa_supplicant.conf

```
network={
        ssid="12345678"
        psk="XXXSECRETXXX"
        id_str="router"
}
```

## hostapd

Create a new file /etc/hostapd/hostapd.conf with:

```
interface=wlan0
ssid=mesh
#sets the mode of wifi, depends upon the devices you will be using. It can be a,b,g,n. Not all cards support 'n'.                                                                                                 
hw_mode=g
channel=7
wmm_enabled=0
macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid=0
```

## dns

dnsmasq for dhcp server on AP, /etc/dnsmasq.conf:

```
interface=wlan0
listen-address=192.168.4.1
bind-interfaces
server=1.1.1.1       # Cloudflare DNS
domain-needed        # Don't forward short names
bogus-priv           # Drop the non-routed address spaces
dhcp-range=192.168.4.50,192.168.4.150,12h # IP range and lease time
```

## nat

in /etc/sysctl.conf, comment out this line:

```
net.ipv4.ip_forward=1

```

Iptables:

```
sudo iptables -t nat -A POSTROUTING -o wlan1 -j MASQUERADE
sudo iptables -A FORWARD -i wlan1 -o wlan0 -m state --state RELATED,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -i wlan0 -o wlan1 -j ACCEPT
```

Save the iptables rule.

```
sudo sh -c "iptables-save > /etc/iptables.ipv4.nat"
```

Edit /etc/rc.local and add this just above "exit 0" to install these rules on boot.

```
iptables-restore < /etc/iptables.ipv4.nat
```
