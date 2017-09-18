# ipvsadm

ip转法的的配置命令;

```
ipvsadm -A -t 192.168.143.78:31555 -s rr -p 600
ipvsadm -a -t 192.168.143.78:31555 -r 10.244.3.5:8080 -m
ipvsadm -lc
```

