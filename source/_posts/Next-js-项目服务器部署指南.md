---
title: Next.js 项目服务器部署指南
date: 2021-10-29 12:19:18
categories: 
- [Node.js]
tags: [Next.js, Ubuntu, Server Deployment, MongoDB, SSH]
---





| 序号 | 程序                    | 版本 or 作用                       |
| ---- | ----------------------- | ---------------------------------- |
| 1    | Trojan                  | Trojan for linux                   |
| 2    | proxychains             | shell /命令行 代理                 |
| 3    | mongodb                 | MongoDB 4.4.4 Community Edition    |
| 4    | mongodump、mongorestore | 数据备份工具，本地安装或服务器安装 |
| 5    | nodejs                  | JavaScript 服务器端运行环境        |
| 6    | pm2                     | 进程守护、自动化部署项目           |
| 7    | ssh                     | 通过ssh协议远程连接                |
| 8    | nginx                   | 80端口反向代理到3000端口           |

<!--more-->

#### 1.Trojan客户端

##### 1.1 下载客户端

```shell
~$ cd /usr/src && wget https://github.com/trojan-gfw/trojan/releases/download/v1.15.1/trojan-1.15.1-linux-amd64.tar.xz
```

##### 1.2 解压

```shell
tar xvf trojan-1.15.1-linux-amd64.tar.xz
```

##### 1.3 配置代理

打开配置文件：

```shell
sudo vim /usr/src/trojan/config.json
```

修改如下：

```json
	"run_type": "client", # 修改为 client
    "local_addr": "0.0.0.0",
    "local_port": 1080, # 修改为 1080
    "remote_addr": "some.domain", # Trojan代理的域名
    "remote_port": 443, # 修改为 443
    "password": ["password"], # Trojan代理的密码
    “ssl": {
    	"verify": false, # 修改为 false （如果配置文件中没有，则添加这个配置）
        "verify_hostname": false, # 修改为 false （如果配置文件中没有，则添加这个配置）
        "cert": "", # 改成空的
    }
    
    
```

##### 1.4 新增 trojan.service

```shell
cat > /etc/systemd/system/trojan.service <<-EOF
[Unit]
Description=trojan
After=network.target

[Service]
Type=simple
PIDFile=/usr/src/trojan/trojan.pid
ExecStart=/usr/src/trojan/trojan -c /usr/src/trojan/config.json -l /usr/src/trojan/trojan.log
ExecReload=/bin/kill -HUP \$MAINPID
Restart=on-failure
RestartSec=1s

[Install]
WantedBy=multi-user.target

EOF
```

##### 1.5 启动 trojan

```shell
sudo systemctl start trojan
```

##### 1.6 查看是否启动成功

###### 1.6.1 方法1—systemctl

```shell
sudo systemctl status trojan
```

###### 1.6.2 方法2—ps 

```shell
ps aux | grep trojan | grep -v grep
```

###### 1.6.3 方法3—curl

```shell
curl ip.sb --socks5 127.0.0.1:1080
```

看是否为代理服务器的 IP

##### 1.7 设置开机启动

```shell
sudo systemctl enable trojan
```



#### 2. proxychains

##### 2.1 安装

```shell
sudo apt-get install proxychains
```

##### 2.2 配置

编辑配置文件，

```shell
sudo vim /etc/proxychains.conf
```

修改如下：

```shell
# socks4 127.0.0.1 9050 # 注释掉
socks5 127.0.0.1 1080 # 加入Trojan的代理设置
```

##### 2.3 测试是否成功安装

###### 2.3.1 测试本地 IP

```shell
curl -4 ip.sb
```

将显示本地的IP；

###### 2.3.2 测试代理 IP

```shell
proxychains curl -4 ip.sb
```

将显示Trojan代理的 IP。

##### 2.4 shell 中使用代理

shell 中需要代理时，只需要在前面加上 `proxychains` 即可，如：

```shell
proxychains npm install
```



#### 3.MongDB

##### 3.1 Ubuntu安装

MongoDB 版本：4.4.4

Ubuntu 版本：20.04 LTS ("Focal")

查看 Ubuntu 版本命令：`lsb_release -dc`

###### 3.1.1 添加 MongoDB public GPG Key

```shell
wget -qO - https://www.mongodb.org/static/pgp/server-4.4.asc | sudo apt-key add -
```

###### 3.1.2 创建MongoDB安装源列表

```shell
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/4.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.4.list
```

###### 3.1.3 重新加载包数据库

```shell
sudo apt-get update
```

###### 3.1.4 安装 MongoDB 包

```shell
sudo apt-get install -y mongodb-org=4.4.4 mongodb-org-server=4.4.4 mongodb-org-shell=4.4.4 mongodb-org-mongos=4.4.4 mongodb-org-tools=4.4.4
```

###### 3.1.5 固定版本信息

避免自动更新，执行：

```shell
echo "mongodb-org hold" | sudo dpkg --set-selections
echo "mongodb-org-server hold" | sudo dpkg --set-selections
echo "mongodb-org-shell hold" | sudo dpkg --set-selections
echo "mongodb-org-mongos hold" | sudo dpkg --set-selections
echo "mongodb-org-tools hold" | sudo dpkg --set-selections
```

##### 3.2 常用命令

###### 3.2.1 启动、停止、状态、重启

```shell
sudo systemctl start mongodsudo systemctl stop mongodsudo systemctl status mongodsudo systemctl restart mongod
```

###### 3.2.2 设置开机启动

```shell
sudo systemctl enable mongod
```

###### 3.2.3 查看日志

```shell
sudo vim /var/log/mongodb/mongod.log
```

###### 3.2.4 开启一个 `mongo shell`连接以使用 MongoDB

```shell
mongo "mongodb://domain:port" -u user -p password
```

##### 3.3 添加超级管理员

###### 3.3.1 添加超级管理员

```mysql
use admin  
db.createUser({
  user: 'admin',  // 用户名
  pwd: '123456',  // 密码
  roles:[{
    role: 'root',  // 角色
    db: 'admin'  // 数据库
  }]
})
```

设置完成，可以输入 `show users` 查看是否设置成功

###### 3.3.2 开启验证

找到 mongod 配置文件：`/etc/mongod.conf`，设置 `security`如下：

```mysql
security:  authorization: enabled
```

###### 3.3.3 重启 MongoDB

```shell
sudo systemctl restart mongod
```



#### 4.MongoDB数据从本地备份至服务器

##### 4.1 备份工具 `mongodump` 与 `mongorestore`

安装 [MongoDB Database Tools](https://docs.mongodb.com/database-tools/) 

##### 4.2 备份代码——从本地备份至服务器

```python
# 只能在 命令行 中执行，因为sys.stdout在vscode ipyhon交互模式中与再命令行中是不同的，前者没有fileno属性
import subprocess
import sys

# 保存备份数据的文件夹，默认为当前目录下的 dump 文件夹
output = 'e:\\backup_data\\data\\' 

# 要备份的数据库
dbs = ['strategy', 'stock', 'minute_stock']
# 远程数据库地址
remote = 'mongodb://IP:PORT' 

# python 执行 shell 命令的函数
def run_shell(shell):
    cmd = subprocess.Popen(shell, stdin=subprocess.PIPE, stderr=sys.stderr, close_fds=True,
                           stdout=sys.stdout, universal_newlines=True, shell=True, bufsize=1)
    cmd.communicate() 
    return cmd.returncode

# 从数据库提取数据的函数
def dump(dbs):
    for db in dbs:
        # mongodump 命令
        shell = ['mongodump', '--gzip', '-d', db, '-o', output]
        run_shell(shell)
# 备份函数
def backup(dbs):
    print('\nstart dumping...\n')
    dump(dbs)
    print('\nstart restore...\n')
    # mongorestore 命令:将数据推送至远程服务器
    shell = ['mongorestore', '--gzip', '--uri', remote, '-u', 'MongoDB user', '-p', 'password', '--dir', output]
    run_shell(shell)
    print('\nDone.')

backup(dbs)
```

**注意事项：** 

1. 远程服务器 mongod的bindIP要修改为`0.0.0.0` 才能允许外部访问，需在腾讯云控制台开放指定端口，且不能为27017(腾讯限制)。
2. 本地与服务器端的MongoDB版本应一致
3. 服务器MongoDB启动错误处理 [Failed To Unlink Socket File Error When Start Mongo DB、code=exited, status=14]： 删除 `mongodb-27017.sock`文件。



#### 5.Node.js

##### 5.1 安装

服务器与本地版本应保持一致，Ubuntu 服务器安装代码：

```shell
# Using Ubuntu
curl -fsSL https://deb.nodesource.com/setup_14.x | sudo -E bash -sudo apt-get install -y nodejs
```

其他版本的[Node.js安装说明与地址](https://github.com/nodesource/distributions/blob/master/README.md#debinstall)



#### 6.PM2

##### 6.1 安装

```javascript
npm install pm2 -g
```

##### 6.2 配置文件

运行：

```shell
pm2 ecosystem
```

生成模板文件 `ecosystem.config.js`，配置文件参考如下：

```javascript
module.exports = {
  apps : [{
    name: 'app name',
    script: 'server.js', // 入口文件
    autorestart: true,
    watch: false,
    max_memory_restart: '1G',
    env: {
      NODE_ENV: 'development'
    },
    env_production: {
      NODE_ENV: 'production'
    }
  }],

  deploy : {
    production : {
      user : 'User', // 服务器用户名
      host : 'IP', // 服务器地址
      ref  : 'origin/main',   // 仓库 branch
      repo : 'git@github.com:user/project.git', // 仓库地址
      path : '/path/to/project', // 项目在服务器上的根目录
      'post-deploy' : 'npm install && npm run build && pm2 reload ecosystem.config.js --env production',
        // pm2 reload 是重启命令，此命令必须加上配置文件名，env参数指定运行环境为生产环境
    }
  }
};

```

其中，通过 `server.js` 作为入口文件，而非采用 `pm2 start npm --name "app name" -- start` 命令，因后者在Windows上无法顺利运行。`server.js`内容如下：

```javascript
var exec = require('child_process').exec;
exec('npm run start', {windowsHide: true})
```

##### 6.3 部署

部署前要求：(1) 服务器能访问GitHub; (2)本地能免密访问服务器

(Windows通过 `git bash` 执行部署命令)

###### 6.3.1 初次部署

```shell
pm2 deploy production setup
```

将代码从仓库克隆至服务器

###### 6.3.2 部署

```shell
pm2 deploy production
```

初次部署后及后续每次更新仓库后，均通过此命令更新服务器代码并重启在线项目。

**注意**：每次更新服务器线上项目前，须将本地开发所做修改通过：

```shell
git add . 
git commit -m "commit message"
```

使GitHub仓库为修改后的最新状态，之后才进行`pm2 deploy production`。

###### 6.3.3 部署之后

运行：

```shell
pm2 startup
```

使 PM2 能开机重启，运行：

```shell
pm2 save
```

使当前运行列表的中项目能开机重启。

##### 6.4 常用命令

```shell
pm2 list # 查看当前项目列表
pm2 logs --lines 20 # 查看指定行数日志
pm2 restart app_name # 重启项目
pm2 reload app_name # 重启项目
pm2 start app_name # 启动项目
pm2 stop app_name # 停止项目
pm2 delete app_name # 从pm2列表中删除项目

# 查看项目详细信息，如执行目录、日志文件等
pm2 show app_name

# 查看当前环境变量
pm2 env app_id
```



#### 7.SSH

所谓"公钥登录"，就是用户将自己的公钥储存在远程主机上。登录的时候，远程主机会向用户发送一段随机字符串，用户用自己的私钥加密后，再发回来。远程主机用事先储存的公钥进行解密，如果成功，就证明用户是可信的，直接允许登录shell，不再要求密码。[阮一峰：SSH原理与运用（一）：远程登录](https://www.ruanyifeng.com/blog/2011/12/ssh_remote_login.html)

##### 7.1 生成公钥和私钥

在 windows 或 Ubuntu 的 home 目录下，执行:

 ```shell
ssh-keygen
 ```

生成 `id_rsa` 和 `id_rsa.pub` 密钥文件

##### 7.2 拷贝公钥至 GitHub

执行：

```shell
cat ~/.ssh/id_rsa.pub
```

将结果复制粘贴至仓库SSH设置中

##### 7.3 设置本地无密码访问服务器

###### 7.3.1 首次访问

`ssh user@host` , 第一次访问需输入密码，会将服务器的公钥保存到本地 `~/.ssh/known_hosts`里。

###### 7.3.2 将本地公钥提交至服务器

```shell
ssh-copy-id user@host
```

此命令将本地公钥提交至服务器，保存至服务器 `/home/user/.ssh/authorized_keys` 文件里，如此，便可实现无密码访问服务器。



#### 8.Nginx

##### 8.1 安装

```shell
apt-get install nginx
```

[基本概念](https://nginx.org/en/docs/beginners_guide.html)

##### 8.2 配置

###### 8.2.1 新建配置文件

```shell
sudo vim /etc/nginx/sites-available/sample.domain
```

###### 8.2.2 添加硬链接

通过绝对路径添加硬链接至 `/etc/nginx/sites-enabled/`目录：

```shell
sudo ln -s /etc/nginx/sites-available/linyibai.space /etc/nginx/sites-enabled/
```

使配置文件在两个目录下保持同步更新，之后只更新 `sites-available`下的文件即可，注意使用绝对路径，否则会生成空文件

###### 8.2.3 每次更新 nginx 配置文件后执行：

1. 测试配置文件是否有误：

   ```shell
   sudo nginx -t
   ```

2. 重启 nginx ：

   ```shell
   sudo service nginx restart
   ```

##### 8.3 配置文件内容

###### 8.3.1 添加域名前，只通过IP访问时

```nginx
server {
  listen 80;
  listen [::]:80;

  server_name IP address or Domain name;

  location / {
    proxy_pass http://localhost:3000;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_cache_bypass $http_upgrade;
  }
}
```

