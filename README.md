# mesh-wireless-ap

1 extra wireless dongle: Edimax EW-7612UAn v2. Works with raspbian lite out of the box.

- wlan0 = internal raspberry pi wifi (access point)
- wlan1 = external wifi (connected to router)

## network

To make sure only wlan1 connects to internet router.

/etc/network/interfaces

```
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

Static ip for wlan0 using dhcp client software, /etc/dhcpcd.conf:

```
interface wlan0
static ip_address=192.168.4.1/24
nohook wpa_supplicant
```

dnsmasq for dhcp server on AP, /etc/dnsmasq.conf:

```
interface=wlan0
listen-address=192.168.4.1
bind-interfaces
domain-needed        # Don't forward short names
bogus-priv           # Drop the non-routed address spaces
dhcp-range=192.168.4.50,192.168.4.150,12h # IP range and lease time
```

## other

If udev changes the network interfaces names you can include a file in /etc/udev/rules.d/ called 10-network-device.rules:

```
SUBSYSTEM=="net", ACTION=="add", ATTR{address}=="22:bb:cc:33:44:dd", NAME="wlan0"
SUBSYSTEM=="net", ACTION=="add", ATTR{address}=="22:bb:cc:33:44:d1", NAME="wlan1"
```
