# DNS服务器搭建

## DNS服务器上的 Linux版本信息

```
Linux version 4.4.0-62-generic (buildd@lcy01-30) (gcc version 5.4.0 20160609 (Ubuntu 5.4.0-6ubuntu1~16.04.4) ) #83-Ubuntu SMP Wed Jan 18 14:10:15 UTC 2017
```

## 配置DNS具体步骤

1.下载bind9\(通过命令下载\)：

```
apt-get install bind9
```

2.编辑bind9的配置的文件（`/etc/bind/name.conf.default-zones`）在末尾添加

```
//正向解析
zone "testqyd.com" {
    type master;

    file "/etc/bind/db.ip2mygitlab.com";
};
//反向解析
zone "1.0.10.in-addr.arpa" {
    type master;

    file "/etc/bind/db.mygitlab2ip";
};
```

其中`testqyd.com`就是你需要设置的域名，`1.0.10.in-addr.arpa`中IP原本应该是\`\`\`10.0.1.106\`\`\`但这里只显示了\`\`\`10.0.1\`\`\`而\`\`\`106\`\`\`不在这里配置，

且这里IP是反写，若你的服务器ip是\`\`\`abc.def.ghi.jkl\`\`\`则在这要写成\`\`\`jkl.ghi.def\`\`\`，而主机号\`\`\`abc\`\`\`在下面步骤会说明用途

然后在\`\`\`/etc/bind\`\`\`中创建\`\`\`db.ip2mygitlab.com\`\`\`文件和\`\`\`db.mygitlab2ip\`\`\`，注意这两个文件名要和\`\`\`name.conf.default-zones \`\`\`中配置的file相同

3.配置\`\`\`db.ip2mygitlab.com\`\`\`文件

\`\`\`

```
;

; BIND data file for local loopback interface

;

$TTL    604800

@       IN      SOA     ns.testqyd.com. root.testqyd.com. \(

                              2         ; Serial

                         604800         ; Refresh

                          86400         ; Retry

                        2419200         ; Expire

                         604800 \)       ; Negative Cache TTL

;

@          IN      NS      ns.testqyd.com.

@          IN      A       10.0.1.106

ns         IN      A       10.0.1.106

wd         IN      A       10.0.1.106

www        IN      A       10.0.1.106
```

\`\`\`

其中你需要将\`\`\`testqyd.com\`\`\`改成你需要域名，\`\`\`10.0.1.106\`\`\`改成你的服务器IP，\`\`\`ns、wd、www\`\`\`

\`\`\`

序号\(Serial\)：这个序号代表的是这个资料库档案的新旧，序号越大代表越新。当slave要判断是否主动下载新的资料库时，就以序号是否比slave上的还要新来判断，若是则下载，若不是则不下载。 所以当你修订了资料库内容时，记得要将这个数值放大才行！ 为了方便使用者记忆，通常序号都会使用日期格式『YYYYMMDDNU』来记忆。不过，序号不可大于2的32次方，亦即必须小于4294967296才行喔。

更新频率\(Refresh\)：那么啥时slave会去向master要求资料更新的判断？就是这个数值定义的。每次slave去更新时，如果发现序号没有比较大，那就不会下载资料库档案。

失败重新尝试时间\(Retry\)：如果因为某些因素，导致slave无法对master达成连线，那么在多久的时间内，slave会尝试重新连线到master。在昆山科大的设定中，900秒会重新尝试一次。意思是说，每1800秒slave会主动向master连线，但如果该次连线没有成功，那接下来尝试连线的时间会变成900秒。若后来有成功，则又会恢复到1800秒才再一次连线。

失效时间\(Expire\)：如果一直失败尝试时间，持续连线到达这个设定值时限，那么slave将不再继续尝试连线，并且尝试删除这份下载的zone file资讯。昆山科大设定为604800秒。意思是说，当连线一直失败，每900秒尝试到达604800秒后，昆山科大的slave将不再更新，只能等待系统管理员的处理。

快取时间\(Minumum TTL\)：如果这个资料库zone file中，每笔RR记录都没有写到TTL快取时间的话，那么就以这个SOA的设定值为主。

\`\`\`

4.配置\`\`\`db.mygitlab2ip\`\`\`文件

\`\`\`

;

; BIND reverse data file for local loopback interface

;

$TTL    604800

@       IN      SOA     ns.testqyd.com. root.testqyd.com. \(

```
                          1         ; Serial

                     604800         ; Refresh

                      86400         ; Retry

                    2419200         ; Expire

                     604800 \)       ; Negative Cache TTL
```

;

@       IN      NS      testqyd.com.

106     IN      PTR     cn.testqyd.com.

106     IN      PTR     testqyd.com.

106     IN      PTR     www.testqyd.com.

106     IN      PTR     dns.testqyd.com.

106     IN      PTR     cn.testqyd.com.

\`\`\`

其中你需要将\`\`\`testqyd.com\`\`\`改成你需要域名，\`\`\`106\`\`\`就是在\`\`\`name.conf.default-zones\`\`\`配置反向解析的主机号，就是在这使用

5.配置\`\`\`/etc/resolv.conf\`\`\`文件,将nameserver配置服务器本机IP（坑）

\`\`\`

nameserver 127.0.0.1

\`\`\`

6.启动DNS服务器

\`\`\`

service bind9 start

\`\`\`

启动后查看\`\`\`cat /var/log/syslog\`\`\` 查看日志是否有Error。附bind9的相关命令：

\`\`\`

service bind9 start    //启动DNS服务器

service bind9 stop     //停止DNS服务器

service bind9 restart  //重启DNS服务器

service bind9 status   //查看DNS服务器状态

\`\`\`

7.测试域名\(nslookup\)

通过\`\`\`nslookup 你的域名\`\`\`查看是否成功

\`\`\`

```
hzdsj@nginx:~$ nslookup testqyd.com

Server:         127.0.0.1

Address:        127.0.0.1\#53



Name:   testqyd.com

Address: 10.0.1.106
```

\`\`\`

8.将原本在hosts中配置的域名注释掉，再将电脑的首选DNS服务器指向\`\`\`10.0.1.100\`\`\`，之后就可以通过\`\`\`testqyd.com\`\`\`来访问相关服务了

9.测试gitbook

