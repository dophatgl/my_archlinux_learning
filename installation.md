# arch_setup

##disable ipv6, when you can't `ping -c 3 google.com`
```
sudo sysctl -w net.ipv6.conf.all.disable_ipv6=1
```


