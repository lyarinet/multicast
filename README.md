```sh 
#!/bin/sh
```
```sh
IPT="/sbin/iptables"
IPT6="/sbin/ip6tables"
```

## Flush old rules
```sh
$IPT --flush
$IPT --delete-chain
```

## By default, drop everything except outgoing traffic
```sh
$IPT -P INPUT DROP
$IPT -P FORWARD DROP
$IPT -P OUTPUT ACCEPT
$IPT6 -P INPUT DROP
$IPT6 -P OUTPUT DROP
$IPT6 -P FORWARD DROP
```

## Allow incoming and outgoing for loopback interfaces
```sh
$IPT -A INPUT -i lo -j ACCEPT
$IPT -A OUTPUT -o lo -j ACCEPT
```

## ICMP rules
```sh
$IPT -A INPUT -p icmp --icmp-type echo-reply -m state --state ESTABLISHED,RELATED -j ACCEPT
$IPT -A INPUT -p icmp --icmp-type echo-request -m limit --limit 5/s -m state --state NEW -j ACCEPT
$IPT -A INPUT -p icmp --icmp-type destination-unreachable -m state --state NEW -j ACCEPT
$IPT -A INPUT -p icmp --icmp-type time-exceeded -m state --state NEW -j ACCEPT
$IPT -A INPUT -p icmp --icmp-type timestamp-request -m state --state NEW -j ACCEPT
$IPT -A INPUT -p icmp --icmp-type timestamp-reply -m state --state ESTABLISHED,RELATED -j ACCEPT
```

## Dos/Scanners....
```sh
$IPT -A INPUT -p tcp --syn -j DROP
$IPT -A INPUT -m conntrack --ctstate NEW -p tcp --tcp-flags SYN,RST,ACK,FIN,URG,PSH SYN -j DROP
$IPT -A INPUT -m conntrack --ctstate NEW -p tcp --tcp-flags SYN,RST,ACK,FIN,URG,PSH FIN -j DROP
$IPT -A INPUT -m conntrack --ctstate NEW -p tcp --tcp-flags SYN,RST,ACK,FIN,URG,PSH ACK -j DROP
$IPT -A INPUT -m conntrack --ctstate INVALID -p tcp --tcp-flags ! SYN,RST,ACK,FIN,URG,PSH SYN,RST,ACK,FIN,URG,PSH -j DROP
$IPT -A INPUT -m conntrack --ctstate NEW -p tcp --tcp-flags SYN,RST,ACK,FIN,URG,PSH FIN,URG,PSH -j DROP
$IPT -A INPUT -p UDP -f -j DROP
$IPT -A INPUT -p TCP --syn -m iplimit --iplimit-above 9 -j DROP
$IPT -A INPUT -m pkttype --pkt-type broadcast -j DROP
$IPT -A INPUT -p ICMP --icmp-type echo-request -m pkttype --pkttype broadcast -j DROP
$IPT -A INPUT -p ICMP --icmp-type echo-request -m limit --limit 3/s -j ACCEPT
$IPT -A INPUT -p TCP --syn -m iplimit --iplimit-above 3 -j DROP
$IPT -A INPUT -p UDP -m pkttype --pkt-type broadcast -j DROP
$IPT -A INPUT -p UDP -m limit --limit 3/s -j ACCEPT
$IPT -A INPUT -p ICMP -f -j DROP
```
## Block new connections without SYN
```sh
$IPT -A INPUT -p tcp ! --syn -m state --state NEW -j DROP
```
## Allow established connections:
```sh
$IPT -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
```
## SSH
```sh
$IPT -A INPUT -p tcp --dport 44 -m state --state NEW -j ACCEPT
```

## Block fragments and Xmas tree as well as SYN,FIN and SYN,RST
```sh
$IPT -A INPUT -p ip -f -j DROP
$IPT -A INPUT -p tcp --tcp-flags ALL ACK,RST,SYN,FIN -j DROP
$IPT -A INPUT -p tcp --tcp-flags SYN,FIN SYN,FIN -j DROP
$IPT -A INPUT -p tcp --tcp-flags SYN,RST SYN,RST -j DROP
```

## Anti-spoofing rules
```sh
$IPT -A INPUT -s 200.200.200.200 -j DROP
$IPT -A INPUT -s 192.168.0.0/24 -j DROP
$IPT -A INPUT -s 127.0.0.0/8 -j DROP
```
## FTP
```sh
$IPT -A INPUT -p tcp --dport ftp -j ACCEPT
$IPT -A INPUT -p tcp --dport ftp-data -j ACCEPT
$IPT -A INPUT -p ALL -i eth0 -m state --state ESTABLISHED,RELATED -j ACCEPT
$IPT -A OUTPUT -o eth0 -p tcp --sport ftp -j ACCEPT
$IPT -A OUTPUT -o eth0 -p tcp --sport ftp-data -j ACCEPT
$IPT -A INPUT -p tcp --dport 50000:51000 -j ACCEPT
```
## Creation channel rejection flood udp 28
```sh
$IPT -N REJECT_FLOOD28
$IPT -A REJECT_FLOOD28 -j LOG --log-prefix 'IPTABLES-FLOOD LENGTH 28: ' --log-level info
$IPT -A REJECT_FLOOD28 -j DROP
```

## Creation channel rejection flood udp 46
```sh
$IPT -N REJECT_FLOOD46
$IPT -A REJECT_FLOOD46 -j LOG --log-prefix 'IPTABLES-FLOOD LENGTH 46: ' --log-level info
$IPT -A REJECT_FLOOD46 -j DROP
```
## Srcds Ports
```sh
$IPT -A INPUT -i eth0 -p udp --dport 27015 -m length --length 28 -j REJECT_FLOOD28
$IPT -A INPUT -i eth0 -p udp --dport 27016 -m length --length 28 -j REJECT_FLOOD28

$IPT -A INPUT -i eth0 -p udp --dport 27015 -m length --length 46 -j REJECT_FLOOD46
$IPT -A INPUT -i eth0 -p udp --dport 27016 -m length --length 46 -j REJECT_FLOOD46
```
## Steam Friends Service
```dh
$IPT -A INPUT -p udp --dport 1200 --jump ACCEPT
```
## Steam Main UDP
```sh
$IPT -A INPUT -p udp --dport 27000:27015 --jump ACCEPT
```
## Steam Main TCP
```sh
$IPT -A INPUT -p tcp --dport 27020:27039 --jump ACCEPT
```
## Steam Dedicated Server HLTV
```sh
$IPT -A INPUT -p udp --dport 27020 --jump ACCEPT
```
## TS3
```sh
$IPT -A INPUT -p TCP --dport 10011 --jump ACCEPT
$IPT -A INPUT -p UDP --dport 9987 --jump ACCEPT
$IPT -A INPUT -p TCP --dport 30033 --jump ACCEPT
```
## MC
```sh
$IPT -A INPUT -p TCP --dport 25565 --jump ACCEPT
$IPT -A INPUT -p TCP --dport 25566 --jump ACCEPT
```
## My SQL
```sh
$IPT -A INPUT -p TCP --dport 3306 --jump ACCEPT
```
