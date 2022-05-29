# Lab-9

## Step1:
根據要求把 netplan 都設定好

gw server:
/etc/netplan/00-installer-config.yaml
```yaml
# This is the network config written by 'subiquity'
network:
  ethernets:
    ens33:
      dhcp4: true
      addresses: [192.168.229.144/24]
    lan:
            dhcp4: false
            addresses: [192.168.3.254/24]
            match:
                    macaddress: "00:0c:29:44:33:a2"
            set-name: lan
  version: 2
```
---
srv server:
netplan
```yaml
# This is the network config written by 'subiquity'
network:
  ethernets:
    eth0:
            addresses: [ 192.168.3.1/24 ]
            match:
                    macaddress: "00:0c:29:fb:2a:5e"
            set-name: eth0
            nameservers:
                    addresses: [ 192.168.3.254 ]
            routes:
                    - to: default
                      via: 192.168.3.254
  version: 2
```
---

clt:
netplan
```yaml
# This is the network config written by 'subiquity'
network:
  ethernets:
    eth0:
      dhcp4: true
      match:
            macaddress: "00:0c:29:28:f4:b7"
      set-name: eth0
  version: 2
```


## Step2 架設 dhcp server

/etc/default/isc-dhcp-server
```bash
# On what interfaces should the DHCP server (dhcpd) serve DHCP requests?
#	Separate multiple interfaces with spaces, e.g. "eth0 eth1".
INTERFACESv4="lan"
INTERFACESv6=""
```

/etc/dhcp/dhcpd.conf

```conf
default-lease-time 600;
max-lease-time 7200;

subnet 192.168.3.0 netmask 255.255.255.0 {
        range 192.168.3.100 192.168.3.200;
        option routers 192.168.3.254;
        option domain-name-servers 8.8.8.8, 1.1.1.1;
        option domain-name "p76091543.nasa.";
}
authoritative;
```


isc-dhcp config command:
```bash
systemctl stop isc-dhcp-server
systemctl restart isc-dhcp-server
systemctl status isc-dhcp-server
```


## DNS server
### Install bind9
```bash
sudo apt install bind9
sudo apt install dnsutils
```

---

### config setup

/etc/bind/named.conf.options
設定 forwarder，位址要轉到 wan ip
把 dnssec-validation auto; 關掉，改成 no
```conf
options {
        directory "/var/cache/bind";

        // If there is a firewall between you and nameservers you want
        // to talk to, you may need to fix the firewall to allow multiple
        // ports to talk.  See http://www.kb.cert.org/vuls/id/800113

        // If your ISP provided one or more IP addresses for stable
        // nameservers, you probably want to use them as forwarders.
        // Uncomment the following block, and insert the addresses replacing
        // the all-0's placeholder.

        forwarders {
                10.100.100.254;
        };

        //========================================================================
        // If BIND logs error messages about the root key being expired,
        // you will need to update your keys.  See https://www.isc.org/bind-keys
        //========================================================================
        dnssec-validation no;

        listen-on-v6 { any; };
};
```

---
/etc/bind/named.conf.default-zones
```conf
// prime the server with knowledge of the root servers
//zone "." {
//      type hint;
//      file "/usr/share/dns/root.hints";
//};

// be authoritative for the localhost forward and reverse zones, and for
// broadcast zones as per RFC 1912
view "default-zones" {
        match-clients { any; };
        zone "localhost" {
                type master;
                file "/etc/bind/db.local";
        };

        zone "127.in-addr.arpa" {
                type master;
                file "/etc/bind/db.127";
        };

        zone "0.in-addr.arpa" {
                type master;
                file "/etc/bind/db.0";
        };

        zone "255.in-addr.arpa" {
                type master;
                file "/etc/bind/db.255";
        };
};
```
---
/etc/bind//named.conf.options
```
options {
        directory "/var/cache/bind";

        // If there is a firewall between you and nameservers you want
        // to talk to, you may need to fix the firewall to allow multiple
        // ports to talk.  See http://www.kb.cert.org/vuls/id/800113

        // If your ISP provided one or more IP addresses for stable
        // nameservers, you probably want to use them as forwarders.
        // Uncomment the following block, and insert the addresses replacing
        // the all-0's placeholder.

        forwarders {
                10.100.100.254;
        };

        //========================================================================
        // If BIND logs error messages about the root key being expired,
        // you will need to update your keys.  See https://www.isc.org/bind-keys
        //========================================================================
        dnssec-validation no;

        listen-on-v6 { any; };
        allow-query { any; };
};
```

---
/etc/bind/named.conf.local
```conf
// Consider adding the 1918 zones here, if they are not used in your
// organization
//include "/etc/bind/zones.rfc1918";

view "lan" {
        match-clients { 192.168.3.0/24; };
        zone "p76091543.nasa" {
                type master;
                file "/etc/bind/zones/p76091543.nasa.fwd";
                allow-query { any; };
                allow-update { none; };
        };
        zone "168.192.in-addr.arpa" {
                type master;
                file "/etc/bind/zones/p76091543.nasa.bac";
                allow-query { any; };
                allow-update { none; };
        };
};

view "wan" {
        match-clients { any; };
        zone "p76091543.nasa" {
                type master;
                file "/etc/bind/zones/p76091543.nasa.fwd.net";
                allow-query { any; };
                allow-update { none; };
        };
        zone "168.192.in-addr.arpa" {
                type master;
                file "/etc/bind/zones/p76091543.nasa.bac.net";
                allow-query { any; };
                allow-update { none; };
        };
};

```
---

/etc/bind/zones/p76091543.nasa.bac
反解好像是錯的
```conf
$TTL    604800
$ORIGIN 168.192.in-addr.arpa.
;SOA
@       IN      SOA     dns.p76091543.nasa root.p76091543.nasa. (
                        2       ;Serial
                        604800  ;Refresh
                        86400   ;Retry
                        2419200 ;Expire
                        604800 )        ;Default TTL

;NS
@       IN      NS      dns.p76091543.nasa.
254.3	IN	PTR	dns.p76091543.nasa.

;web server
1.3.168.192.in-addr.arpa.       IN      PTR     www.p76091543.nasa.
```
---
/etc/bind/zones/p76091543.nasa.bac.net
反解好像是錯的
```conf
$TTL	604800
$ORIGIN 100.10.in-addr.arpa.
;SOA
@	IN	SOA	dns.p76091543.nasa root.p76091543.nasa. (
			2	;Serial
			604800	;Refresh
			86400	;Retry
			2419200	;Expire
			604800 )	;Default TTL

;NS
@	IN	NS	dns.p76091543.nasa.
49.100.100.10.in-addr.arpa.	IN	PTR	dns.p76091543.nasa.

;PTR
49.100.100.10.in-addr.arpa.	IN	PTR	www.p76091543.nasa.
```

---
正解
/etc/bind/zones/p76091543.nasa.fwd
```conf
$TTL	604800
$ORIGIN	p76091543.nasa.
;SOA
@	IN	SOA	dns.p76091543.nasa	root.p76091543.nasa. (
			2	;Serial
			604800	;Refresh
			86400	;Retry
			2419200	;Expire
			604800 )	;Default TTL

;NS
@	IN	NS	dns.p76091543.nasa.
dns	IN	A	192.168.3.254

;A
;@	IN	A	192.168.3.254 ;@ is domain name
www	IN	A	192.168.3.1 ;@ is domain name
```
---
正解
/etc/bind/zones/p76091543.nasa.fwd.net
```conf
$TTL    604800
$ORIGIN p76091543.nasa.
;SOA
@       IN      SOA     dns.p76091543.nasa  root.p76091543.nasa. (
                        2       ;Serial
                        604800  ;Refresh
                        86400   ;Retry
                        2419200 ;Expire
                        604800 )        ;Default TTL

;NS
@       IN      NS      dns.p76091543.nasa.
;dns	IN	A	10.100.100.49
dns	IN	A	192.168.3.254

;A
;@	IN      A       10.100.100.49 ;@ is domain name
www     IN      A       10.100.100.49 ;@ is domain name
```



## iptables
iptables
```yaml
# Generated by iptables-save v1.8.4 on Wed May  4 15:05:39 2022
# this tables got 60 scores
*nat
:PREROUTING ACCEPT [49:4488]
:INPUT ACCEPT [0:0]
:OUTPUT ACCEPT [8:1610]
:POSTROUTING ACCEPT [9:1938]
-A POSTROUTING -o wan -j MASQUERADE
-A PREROUTING -i wan -p tcp --dport 80 -j DNAT --to-destination 192.168.3.1
-A PREROUTING -p udp --dport 53 -j DNAT --to-destination 192.168.3.254
COMMIT

# Completed on Wed May  4 15:05:39 2022
# Generated by iptables-save v1.8.4 on Wed May  4 15:05:39 2022
*filter
:INPUT ACCEPT [0:0]
:FORWARD ACCEPT [4:1312]
:OUTPUT ACCEPT [832:94094]
-A INPUT -p icmp -j ACCEPT
-A INPUT -i lo -j ACCEPT
-A INPUT -p tcp -m tcp -m multiport --dports 22 -j ACCEPT
-A INPUT -p udp -j ACCEPT
-A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
-A INPUT -j DROP
COMMIT
# Completed on Wed May  4 15:05:39 2022
# Generated by iptables-save v1.8.4 on Wed May  4 15:05:39 2022
*mangle
:PREROUTING ACCEPT [2208:181162]
:INPUT ACCEPT [2200:179658]
:FORWARD ACCEPT [4:1312]
:OUTPUT ACCEPT [1367:157622]
:POSTROUTING ACCEPT [1370:158578]
-A PREROUTING -p udp -m comment --comment "wg-quick(8) rule for wan" -j CONNMARK --restore-mark --nfmask 0xffffffff --ctmask 0xffffffff
-A FORWARD -p tcp -m tcp --tcp-flags SYN,RST SYN -j TCPMSS --clamp-mss-to-pmtu
-A POSTROUTING -p udp -m mark --mark 0xca6c -m comment --comment "wg-quick(8) rule for wan" -j CONNMARK --save-mark --nfmask 0xffffffff --ctmask 0xffffffff
COMMIT
# Completed on Wed May  4 15:05:39 2022
# Generated by iptables-save v1.8.4 on Wed May  4 15:05:39 2022
*raw
:PREROUTING ACCEPT [2208:181162]
:OUTPUT ACCEPT [1367:157622]
COMMIT
# Completed on Wed May  4 15:05:39 2022
```