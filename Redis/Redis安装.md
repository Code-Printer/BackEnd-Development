## 在本地windows安装redis  
1. Redis官方并不支持在windows上安装redis，但github上有windows系统的redis安装包，根据自己系统的位数选择相应位数的安装包   
```
https://github.com/MSOpenTech/redis/releases
```
2. 解压压缩文件到C盘，使用cmd切换到该目录下，使用如下命令启动redis，保持窗口打开。   
```
redis-server redis.windows.conf
```
3. 移动到该目录下，使用如下命令将Redis加入到系统服务   
```
redis-server.exe --service-install redis.windows.conf --loglevel verbose
```
4. 此时使用如下命令就可以直接启动、停止和卸载redis服务   
```
#启动服务
redis-server --service-start
#停止服务
redis-server –service-stop
#卸载服务
redis-server --service-uninstall
```
## 在阿里云服务器的Ubuntu系统上安装Redis  
1、首先去redis.io网站下载合适的Redis版本  
2、上传下载包到阿里云服务器的/opt目录下，解压使用如下命令安装  
```
tar -zxvf redis.tar.gz
```
3、安装基本的软件包  
```
sudo apt update
sudo apt install build-essentia
```
4、在对应的redis文件下，使用make命令配置需要的文件  
```
make
make install
```
5、redis的默认安装路径在/usr/local/bin目录下  
6、在/usr/local/bin下创建myconfig目录，存放redis.config文件，使用这个配置文件启动，源配置文件当做备份，出错好恢复  
```
mkdir myconfig
cp /opt/redis-5.0.14/redis.conf myconfig
```
7、redis默认不是后台启动的，需要到配置文件中修改配置文件  
```
daemonize yes
```
8、启动redis服务  
切换到/usr/local/bin目录下  
启动redis服务：redis-server myconfig/redis.conf  
![](https://mrggz.oss-cn-hangzhou.aliyuncs.com/img/202205201140243.png?x-oss-process=style/null)  
9、使用redis-cli客户端测试连接  
redis-cli -p 6379  
![](https://mrggz.oss-cn-hangzhou.aliyuncs.com/img/202205201142395.png?x-oss-process=style/null)  
10、关闭redis服务  
方式一：在redis-cli中输入命令shutdown  
方式二：查看redis服务进程号，使用kill命令杀死该进程  
```
ps -ef|grep redis
kill -9 进程号
```
11、在服务器的安全组中添加6379的安全规则端口  
![](https://mrggz.oss-cn-hangzhou.aliyuncs.com/img/202205201200526.png?x-oss-process=style/null)   
12、为了服务器的安全，设置redis客户端连接密码  
1. 进入到解压的redis目录的src文件夹下：cd /opt/redis-5.0.14/src  
2. 启动客户端：./redis-cli  
3. get name //获取名字  
4. config get requirepass //查询密码
“requirepass”

   ""  //出现这种情况就是自己的redis没有密码，自己需要设置  
5. config set requirepass '你的密码'   

13、设置完密码后需要再次用密码登录  
1. 进入到解压的redis目录的src文件夹下：cd /opt/redis-5.0.14/src  
2. 启动客户端：./redis-cli  
3. config get requirepass
会报这个错误(error) NOAUTH Authentication required.  
4. 输入auth 设置登录密码  
再操作就没有问题了  
5. 在usr/local/bin/中使用redis-cli登录时需要完整输入以下命令  
```
redis-cli -p 6379 -a 登陆密码
```




