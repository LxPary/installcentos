# Setup easy DNS server using dnsmasq on CentOS 7
## 1 Install dnsmasq

```
$ yum install dnsmasq -y
```

## 2 Put in upstream dns server in /etc/resolv.conf. In this case, I want to use opendns as my upstream dns server.

```
$ cat >> /etc/resolv.conf <<EOF
nameserver 208.67.222.222
EOF

```

## 3 For dns records, just use /etc/hosts

```
$ cat >> /etc/hosts <<EOF
192.168.1.xxx mydns.local
192.168.1.xxx myportal.local
192.168.1.xxx myworkspace.local
EOF
```

## 4 With just these 2 settings, you are good to go. Start dnsmasq, and your dns server should be able to resolve those 3 domains.

```
$ systemctl start dnsmasq
```

## 5 Allow on firewall

```
$ firewall-cmd --add-service dns
$ firewall-cmd --add-service dns --permanent
```

## 6 Test with dig

```
$ dig +short @localhost myportal.local
192.168.0.xxx
```

## 7 Test from other machine

```
$ dig +short @192.168.1.xx myworkspace.local
192.168.1.xxx

```

## 8 It can even forward to upstream DNS

```
$ dig +short @192.168.0.xx www.google.com
216.58.xxx.xx

```