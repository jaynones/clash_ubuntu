# clash_ubuntu20.04
集合了许多网友的教程汇集而成的保姆级教程



## 第一章 安装clash

```sh
1.使sudo用户可以免密登录 #在此文件内增加一行
cat >> /etc/sudoers << EOF
ubuntu ALL=(ALL)NOPASSWD: ALL
EOF

2.使root用户可以远程登录 #编辑此文件
vim /etc/ssh/sshd_config
sed -i '/PermitRootLogin /c PermitRootLogin yes' /etc/ssh/sshd_config

3.安装依赖包
apt-get update && apt-get -y install net-tools lrzsz vim wget 

4.新建文件夹
mkdir clash && cd clash

5.github项目地址，安装包在此下载
https://github.com/Dreamacro/clash/releases/
wget https://github.com/Dreamacro/clash/releases/download/v1.11.8/clash-linux-amd64-v1.11.8.gz

6.上传文件并解压
gzip -d clash-linux-amd64-v1.11.8.gz
mv clash-linux-amd64-v1.11.8.gz clash

7.将文件复制到可执行程序目录
cp clash /usr/local/bin/clash && chmod +x /usr/local/bin/clash && ls -l /usr/local/bin/clash

8.验证是否成功
root@clash:~/clash# clash -v
Clash v1.11.8 linux amd64 with go1.19.3 Fri Nov 25 12:43:25 UTC 2022

9.为 clash 添加绑定低位端口的权限，这样运行clash的时候无需root权限
sudo setcap cap_net_bind_service=+ep /usr/local/bin/clash
```



## 第二章 创建配置文件

- `clash` 运行需要依赖相应的 `YAML` 配置文件，默认读取 `$HOME/.config/clash/config.yaml`
- 当没有这份文件的时候，clash 会使用默认配置生成一份，所以我们可以直接运行一下 `clash` ，来获取模版，同时会自动下载 `Country.mmdb` 文件

```sh
1.创建配置文件目录
mkdir -p ~/.config/clash && cd ~/.config/clash

2.创建配置文件
此config.yaml文件是基于订阅的，可以访问对应的订阅商获取


3.第一次在终端执行 clash 的结果如下 此时已获得配置文件的模板，但这模板并没有什么用，yaml 文件的内容往往需要代理方提供，这就需要去购买 VPN
root@clash-1:~# clash
INFO[0000] Can't find config, create a initial config file 
INFO[0000] Can't find MMDB, start download

4.将从代理商处获取的config.yml复制到~/.config/clash下，并改后缀为yaml
cp config.yml /root/.config/clash/config.yaml

5.设置密码，在config.yaml文件内更改
# RESTful API 的口令
secret: '123456'

5.在配置完成后即可在终端执行 sudo clash 运行，若无报错则运行成功
root@clash:~# clash
INFO[0000] Start initial compatible provider Proxy      
INFO[0000] Start initial compatible provider Domestic   
INFO[0000] Start initial compatible provider GlobalTV   
INFO[0000] Start initial compatible provider Others     
INFO[0000] Start initial compatible provider AsianTV    
INFO[0000] RESTful API listening at: [::]:9090          

#指定配置文件执行
root@clash:~/clash# ./clash -f config.yaml
INFO[0000] Start initial compatible provider Proxy      
INFO[0000] Start initial compatible provider Domestic   
INFO[0000] Start initial compatible provider GlobalTV   
INFO[0000] Start initial compatible provider AsianTV    
INFO[0000] Start initial compatible provider Others

6.运行后可以浏览器访问 Clash控制台
http://clash.razord.top/#/proxies
```

![image](https://user-images.githubusercontent.com/73376764/209537529-fe2f8d23-d0bf-460e-9023-e3c881e3d0af.png)


- 如果访问失败，打开设置->网络->网络代理，切换为手动，设置下图参数后

```sh
#命令行版有需要安装GUI的话
apt-get -y install tasksel
tasksel install ubuntu-desktop
```

![image](https://user-images.githubusercontent.com/73376764/209537575-b0ea0539-be6b-45ce-9400-2128b7cf809a.png)

```sh
7.如果不是安装的GUI，没有打开代理，则需要手动开启代理
export https_proxy=http://127.0.0.1:7890 http_proxy=http://127.0.0.1:7890 all_proxy=socks5://127.0.0.1:7891

8.保持开机启动，修改~/.bashrc
echo "export https_proxy=http://127.0.0.1:7890 http_proxy=http://127.0.0.1:7890 all_proxy=socks5://127.0.0.1:7891" >> ~/.bashrc

#取消代理
unset  http_proxy  https_proxy  all_proxy
```

- 点击设置，检查***允许来自局域网的连接***是否开启

![image](https://user-images.githubusercontent.com/73376764/209537748-a4a4a80e-f2e4-4111-94b0-6ce17c5e7f44.png)


- 测试是否成功连接

```sh
curl -I https://www.google.com/
```

![image](https://user-images.githubusercontent.com/73376764/209537768-6e3e606f-129c-44ab-b974-1d5083214fab.png)


## 第三章 设置clash开机自启动

```sh
1.创建编辑service文件
cat > /etc/systemd/system/clash.service << EOF
[Unit]
Description=clash daemon

[Service]
Type=simple
User=root
ExecStart=/usr/local/bin/clash -f /root/.config/clash/config.yaml
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF

#ExecStart 的路径可以使用以下命令进行查询
ls $HOME/.config/clash/config.yaml

2.重新加载systemd模块
systemctl daemon-reload && systemctl start clash && systemctl enable clash --now && systemctl status clash

3.关闭防火墙
ufw disable
```



## 第四章 其他机器访问外网

```sh
1.手动开启代理，加载进环境变量
export https_proxy=http://192.168.26.90:7890 http_proxy=http://192.168.26.90:7890 all_proxy=socks5://192.168.26.90:7891

2.保持开机启动，修改~/.bashrc
echo "export https_proxy=http://127.0.0.1:7890 http_proxy=http://127.0.0.1:7890 all_proxy=socks5://127.0.0.1:7891" >> ~/.bashrc

3.配置docker使用http代理
sudo mkdir -p /etc/systemd/system/docker.service.d 
sudo touch /etc/systemd/system/docker.service.d/proxy.conf
sudo chmod 777 /etc/systemd/system/docker.service.d/proxy.conf
sudo echo '
[Service]
Environment="HTTP_PROXY=http://192.168.26.90:7890"
Environment="HTTPS_PROXY=http://192.168.26.90:7890"
' >> /etc/systemd/system/docker.service.d/proxy.conf
sudo systemctl daemon-reload
sudo systemctl restart docker
```
