# 1、lsof -Pn -i4
```
lsof -Pn -i4
COMMAND     PID   USER   FD   TYPE             DEVICE SIZE/OFF NODE NAME
loginwind   137 ouyang    7u  IPv4 0x9cef83178bd2c065      0t0  UDP *:*
rapportd    366 ouyang    5u  IPv4 0x9cef83178db23935      0t0  UDP *:*
rapportd    366 ouyang    6u  IPv4 0x9cef83178db23c25      0t0  UDP *:*
rapportd    366 ouyang    9u  IPv4 0x9cef83178bd2b795      0t0  UDP *:*
Google      715 ouyang   37u  IPv4 0x9cef8317977c12a5      0t0  TCP 127.0.0.1:62722->127.0.0.1:4780 (CLOSE_WAIT)
Google      715 ouyang   38u  IPv4 0x9cef83179460156d      0t0  TCP 127.0.0.1:62728->127.0.0.1:4780 (CLOSE_WAIT)
Google      715 ouyang   39u  IPv4 0x9cef831797833ccd      0t0  TCP 127.0.0.1:62729->127.0.0.1:4780 (CLOSE_WAIT)
Google      715 ouyang   43u  IPv4 0x9cef831794580e55      0t0  TCP 127.0.0.1:62737->127.0.0.1:4780 (CLOSE_WAIT)
Google      715 ouyang   48u  IPv4 0x9cef8317978a19bd      0t0  TCP 127.0.0.1:62743->127.0.0.1:4780 (CLOSE_WAIT)
Google      715 ouyang   49u  IPv4 0x9cef8317945faf95      0t0  TCP 127.0.0.1:62748->127.0.0.1:4780 (CLOSE_WAIT)
```
其中:
```
-i4 means only show ipv4 address and ports -P and -n fast output
```

只查看监听端口：
```
➜  ~ lsof -Pn -i4 | grep LISTEN
clashr-da   469 ouyang   12u  IPv4 0x9cef83178f07187d      0t0  TCP 127.0.0.1:4780 (LISTEN)
clashr-da   469 ouyang   13u  IPv4 0x9cef83178f0736f5      0t0  TCP 127.0.0.1:4781 (LISTEN)
clashr-da   469 ouyang   14u  IPv4 0x9cef83178f07411d      0t0  TCP 127.0.0.1:4788 (LISTEN)
Python    24811 ouyang    5u  IPv4 0x9cef83179457a6f5      0t0  TCP 127.0.0.1:8000 (LISTEN)
```

国哥喜欢这个命令，容易打，而且能看ESTABLISHED状态.

# 2、lsof -PiTCP -sTCP:LISTEN
和上面的差不多，参数些许变化而已：查出ipv4和ipv6的监听。

当然，上面的命令`lsof -Pn -i6 | grep LISTEN`也能查出ipv6。

```
➜  ~ lsof -PiTCP -sTCP:LISTEN
COMMAND     PID   USER   FD   TYPE             DEVICE SIZE/OFF NODE NAME
idea        369 ouyang   55u  IPv6 0x9cef83179b831415      0t0  TCP localhost:50807 (LISTEN)
idea        369 ouyang  274u  IPv6 0x9cef831793ea6415      0t0  TCP localhost:6942 (LISTEN)
idea        369 ouyang  481u  IPv6 0x9cef831793ea5755      0t0  TCP localhost:63342 (LISTEN)
pycharm     384 ouyang  233u  IPv6 0x9cef831793ea5db5      0t0  TCP localhost:6943 (LISTEN)
pycharm     384 ouyang  346u  IPv6 0x9cef831793ea50f5      0t0  TCP localhost:63343 (LISTEN)
clashr-da   469 ouyang   12u  IPv4 0x9cef83178f07187d      0t0  TCP localhost:4780 (LISTEN)
clashr-da   469 ouyang   13u  IPv4 0x9cef83178f0736f5      0t0  TCP localhost:4781 (LISTEN)
clashr-da   469 ouyang   14u  IPv4 0x9cef83178f07411d      0t0  TCP localhost:4788 (LISTEN)
com.docke  1125 ouyang  105u  IPv6 0x9cef83179bde10f5      0t0  TCP *:6379 (LISTEN)
com.docke  1125 ouyang  106u  IPv6 0x9cef83179bde1db5      0t0  TCP *:3306 (LISTEN)
Python    24811 ouyang    5u  IPv4 0x9cef83179457a6f5      0t0  TCP localhost:8000 (LISTEN)
node      33784 ouyang   13u  IPv6 0x9cef83179b8300f5      0t0  TCP *:3000 (LISTEN)
node      33784 ouyang   14u  IPv6 0x9cef83179b82fa95      0t0  TCP *:35729 (LISTEN)
```

# 3、netstat -anvp tcp | awk 'NR<3 || /LISTEN/'
```
➜  ~ netstat -anvp tcp | awk 'NR<3 || /LISTEN/'
Active Internet connections (including servers)
Proto Recv-Q Send-Q  Local Address          Foreign Address        (state)     rhiwat shiwat    pid   epid  state    options
tcp46      0      0  *.35729                *.*                    LISTEN      131072 131072  33784      0 0x0100 0x00000106
tcp46      0      0  *.3000                 *.*                    LISTEN      131072 131072  33784      0 0x0100 0x00000106
tcp4       0      0  127.0.0.1.8000         *.*                    LISTEN      131072 131072  24811      0 0x0000 0x00000006
tcp46      0      0  *.3306                 *.*                    LISTEN      131072 131072   1125      0 0x0100 0x00000006
tcp46      0      0  *.6379                 *.*                    LISTEN      131072 131072   1125      0 0x0100 0x00000006
tcp4       0      0  127.0.0.1.50807        *.*                    LISTEN      131072 131072    369      0 0x0100 0x00000006
tcp4       0      0  127.0.0.1.63343        *.*                    LISTEN      131072 131072    384      0 0x0100 0x00000006
tcp4       0      0  127.0.0.1.63342        *.*                    LISTEN      131072 131072    369      0 0x0100 0x00000006
tcp4       0      0  127.0.0.1.6943         *.*                    LISTEN      131072 131072    384      0 0x0100 0x00000006
tcp4       0      0  127.0.0.1.6942         *.*                    LISTEN      131072 131072    369      0 0x0100 0x00000006
tcp4       0      0  127.0.0.1.4788         *.*                    LISTEN      131072 131072    469      0 0x0100 0x00000006
tcp4       0      0  127.0.0.1.4781         *.*                    LISTEN      131072 131072    469      0 0x0100 0x00000006
tcp4       0      0  127.0.0.1.4780         *.*                    LISTEN      131072 131072    469      0 0x0100 0x00000006
```
小知识：
```
awk NR < 3，是为了打印列头。 
NR = NUMBER OF ROW = 行数。
NR < 3 是为了打印前三行，也就能把列头打印出来。
```

# 4、参考
[How can I list my open network ports with netstat?](https://apple.stackexchange.com/questions/117644/how-can-i-list-my-open-network-ports-with-netstat)