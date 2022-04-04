# 在linux（centos）上使用VPN翻墙进校内

### 通过将运行在 docker 的 深信服（Sangfor）非自由的代理软件 EasyConnect 作为网关 并添加路由表 实现资源访问

！！！只有学校使用的是深信服（Sangfor）的代理软件 EasyConnect 才可以

##### step1：首先自行下载安装好docker 并进行简单验证（如运行docker run hello-world 等

可以通过以下链接根据官方教程进行安装

https://docs.docker.com/engine/install/centos/

##### step2：拉取docker-easyconnect镜像在doker内进行vpn连接

```shell
docker run --device /dev/net/tun --cap-add NET_ADMIN -ti -p 127.0.0.1:1080:1080 -p 127.0.0.1:8888:8888 -e EC_VER=7.6.3 -e CLI_OPTS="-d XXXXXX  -u XXXXXX -p XXXXXX" hagb/docker-easyconnect:cli
```

以上命令中XXXX部分自行替换   其中第一个XXXX是学校vpn网址  第二个用户名第三个密码

初次使用命令 会找不到镜像 等待一会doker就会从dokerhub拉取镜像

如下图出现successfully表示成功

![image-20220404162441244](http://106.14.104.157/images/image-20220404162441244.png)

（这时没有提示符让你输命令了，千万不要按ctrl+c  但是可以直接关掉这个session窗口/终端  然后另外开一个 session窗口/终端就行了）

！这时候vpn运行在doker内  并且把1080 8888 端口暴露出来分别用于socket 及 http代理

！！！但是这还需要进行额外的代理配置才能使用代理  具体配置可以参考后面给出的参考链接 （如git的配置等

！！！！而且现在ping学校内网是ping不通的

![image-20220404163431392](http://106.14.104.157/images/image-20220404163431392.png)

##### step3：将docker作为网关配置路由表

因为EasyConnect-docker提供作为网关和/或提供 socks5、http 代理服务

所以只需要进行路由表配置就ok了

1. 首先查看容器名

   ```shell
   docker ps
   ```

   ![image-20220404164116569](http://106.14.104.157/images/image-20220404164116569.png)

   运行

   ```shell
   docker exec  xxxxx  cat /sys/class/net/tun0/mtu
   ```

   xxxx替换为容器名  我这里是gifted_blackburn

   如果输出数字则正常

2. 设置MTU为上面查看到的  XXXX自行替换

   ```shell
   MTU=$(docker exec XXXXX  cat /sys/class/net/tun0/mtu)
   ```

3. 

   ```shell
   docker inspect  容器名
   ```

   ![image-20220404165156659](http://106.14.104.157/images/image-20220404165156659.png)

![image-20220404165223329](http://106.14.104.157/images/image-20220404165223329.png)

找到这里 记为ip1

4.

```shell
ip route add ip2替换/16 via ip1替换 mtu $MTU table 3
```

ip2是要访问的学校内网网段 

如：ip route add 172.29.0.0/16  via 172.17.0.2 mtu $MTU table 3

 5.

```
ip rule add iif lo table 3
```

再ping一下  成功了\\^o^/

![image-20220404165719566](http://106.14.104.157/images/image-20220404165719566.png)





参考链接：

- docker：https://docs.docker.com/engine/install/centos/
- docker-easyconnect：https://github.com/Hagb/docker-easyconnect
- git代理设置：https://www.cnblogs.com/hookjc/p/13179004.html
- Linux 设置代理：https://blog.csdn.net/jim_LoveQ/article/details/95313989
- [docker查看容器IP地址](https://www.cnblogs.com/kofsony/p/11101356.html)

