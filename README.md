# ocserv-build



1、更新包列表&安装依赖环境
```
apt-get update

apt-get install build-essential libgnutls28-dev libpam0g-dev libwrap0-dev libreadline-dev libnl-nf-3-dev libseccomp-dev pkg-config
```

基本的步骤都通过 shell 脚本完成，程序仅仅在 Ubuntu Server 12.04/14.04 下简单测试过，Github 地址。不放心的朋友们可以自己一步步执行脚本里面的代码。所有程序都需要在 Root 用户下执行。


2、编译安装 OCServ
```
bash build-src.sh
```


安装路径在 /opt/ocserv 下，安装成功后，测试一下下面的命令
（如果可以正常打印帮助信息就说明安装成功了。）
```
 cd /opt/ocserv 
 LD_LIBRARY_PATH=/opt/local/lib ./sbin/ocserv -h
```

3、配置证书

执行脚本之前，需要配置的地方 (build-ca.sh文件)：

1. AUTHOR 填维护者或者公司的名字
2. VPN 填你给这个vpn的取名
3. DOMAIN 服务器的域名，如果没域名用 ip


最终证书都生成在 /opt/ocserv/ca 目录下，其中用户证书会要求你输入密码，记得一定要输入，并记住。


执行下面命令生成证书
```
bash build-ca.sh
```
4、配置 iptable/添加 upstart 脚本

运行下面的脚本开启转发：
```
iptables -t nat -A POSTROUTING -s 10.0.1.0/24 -o eth0 -j MASQUERADE
```

如果是 openvz 的主机则：
```
iptables -t nat -A POSTROUTING -s 10.0.1.0/24 -o venet0 -j MASQUERADE
```


设置完转发后，在 /etc/sysctl.conf 中添加 net.ipv4.ip_forward=1，并运行下面的脚本使之生效：
```
sysctl -p
```
4.1、iptables 持久化
```
iptables-save > /etc/iptables.rules
cat << EOF > /etc/network/if-pre-up.d/iptablesload
#!/bin/sh
iptables-restore < /etc/iptables.rules
exit 0
EOF

chmod +x /etc/network/if-pre-up.d/iptablesload
```

4.2、添加 upstart 脚本
```
mv config/ocserv.conf /etc/init/
```

5、测试运行

添加 ocserv 配置文件 + 测试连接

```
mkdir /opt/ocserv/etc/
mv config/config /opt/ocserv/etc/
LD_LIBRARY_PATH=/opt/local/lib ./sbin/ocserv -c etc/config -f -d 1
```

如果连接成功了，那么就开启服务
```
service ocserv start
```



