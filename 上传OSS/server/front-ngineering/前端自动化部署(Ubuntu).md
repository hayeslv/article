大致流程如下：

<img src="https://dylan-static.oss-cn-beijing.aliyuncs.com/markdown/images/githook.png" alt="" />



### 安装服务(我这里是Ubuntu系统)

安装Node环境

```shell
# 在 /root 下新建nodejs文件夹（这里可以按照自己配置的路径）
mkdir nodejs

# 官网下载
wget https://nodejs.org/dist/v12.10.0/node-v12.10.0-linux-x64.tar.xz

# 如果官网下载报错：wget:unable to resolv host address
# 可以尝试进行下列操作
vi /etc/resolv.conf
# 添加下列代码
nameserver 8.8.8.8 #google https://nodejs.org


# 解压
tar xf node-v12.10.0-linux-x64.tar.xz

# 输入以下命令（注意，/root/tool/nodejs  换成你自己官网下载node的解压路径，我这里是在 /root/tool/nodejs 解压的）
ln -s /root/tool/nodejs/node-v12.10.0-linux-x64/bin/node /usr/local/bin/node
ln -s /root/tool/nodejs/node-v12.10.0-linux-x64/bin/npm /usr/local/bin/npm


# 测试
node -v
输出： v12.10.0
npm -v
输出： 6.10.3
代表nodejs安装成功

# 安装 cnpm
npm install -g cnpm -registry=https://registry.npm.taobao.org
# /root/tool/nodejs/node-v12.10.0-linux-x64/bin/cnpm -> /root/tool/nodejs/node-v12.10.0-linux-x64/lib/node_modules/cnpm/bin/cnpm
# 配置环境
ln -s /root/tool/nodejs/node-v12.10.0-linux-x64/bin/cnpm /usr/local/bin/cnpm
```



#### 服务器安装vue-cli

安装cnpm：`sudo npm install -g cnpm --registry=https://registry.npm.taobao.org`

执行命令：`npm install -g @vue/cli`

安装完成后输出：

```shell
/root/tool/nodejs/node-v12.10.0-linux-x64/bin/vue -> /root/tool/nodejs/node-v12.10.0-linux-x64/lib/node_modules/@vue/cli/bin/vue.js
```

配置环境变量（软连接包）：

```shell
ln -s /root/tool/nodejs/node-v12.10.0-linux-x64/bin/vue /usr/local/bin/
```

执行 vue -V 生效！



#### 安装docker-compose

```shell
apt install docker-compose

# 试验一下是否安装成功
mkdir helloworld

vi docker-compose.yml

# 将下面代码弄进去
version: '3.1'
services:
  hello-world:
    image: hello-world

# 启动
docker-compose up
# 如果能打印出日志，说明没有问题，安装完毕
```

简介：Compose项目是Docker官方的开源项目，负责实现对Docker容器集群的快速编排。

一句话概括：如果好几个Docker想一起工作，就用 Compose



### 代码配置

> 首先，需要去github上配置githook：git工程 => settings => Webhooks



#### 配置VSCode（上传代码使用）

```json
// .vscode/sftp.json
{
    "name": "AliyunServer",
    "host": "xx.xx.xx.xx",
    "port": 22,     
    "username": "root",
    "password": "xxxxxxxxxxx", 
    "protocol": "sftp", 
    "passive": false,
    "interactiveAuth": false,
    "remotePath": "/root/for-study-test/git-for-study",    
    "uploadOnSave": false, 
    "syncMode": "update",
    "ignore": [            
        "**/.vscode/**",
        "**/node_modules/**",
        "**/.DS_Store"
    ]
}
```



在项目根目录创建autobuild文件夹

安装 github-webhook-handler 和 pm2

```shell
npm i github-webhook-handler pm2 -g
```



#### **编写webhook服务**

```js
//  autobuild/webhook.js
const http = require('http');
// github-webhook-handler 的绝对路径
var createHandler = require('/root/tool/nodejs/node-v12.10.0-linux-x64/lib/node_modules/github-webhook-handler')
// secret 保持和 GitHub 后台设置保持一致
var handler = createHandler({ path: '/', secret: 'dylanlv2021' })

function run_cmd(cmd, args, callback) {
  var spawn = require('child_process').spawn;
  var child = spawn(cmd, args);
  var resp = "";

  child.stdout.on('data', function(buffer) { resp += buffer.toString(); });
  child.stdout.on('end', function() { callback (resp) });
}

http.createServer(function (req, res) {
  handler(req, res, function (err) {
    res.statusCode = 404
    res.end('no such location')
  })
}).listen(7777) // 启动服务的端口，需要开放安全组

handler.on('push', function (event) {
  console.log('Received a push event for %s to %s',
    event.payload.repository.name,
    event.payload.ref);
    run_cmd('sh', ['./webhooks.sh',event.payload.repository.name], function(text){ console.log(text) });
})
```



#### **编写shell脚本**

autobuild/webhook.sh

```shell
#!/bin/bash
WEB_PATH='/root/for-study-test/git-for-study/deploy_server/pm2-test/server'

echo "开始执行shell"
cd $WEB_PATH
echo "pulling source code..."
git pull &&
echo "changing permissions..."
#chown -R $WEB_USER:$WEB_USERGROUP $WEB_PATH
echo " git pull 完成. 开始 build"
npm run build &&
echo "build 完成"
echo "开始重启docker-compose"
docker-compose down &&
docker-compose up -d
echo "docker-compose 重启完毕"
```



#### **启动pm2服务**

```shell
# 在此文件夹下：autobuild/ 
pm2 start webhooks.js -o ./webhooks.log
```



至此githook的功能就全部实现了！！！

下面是配置docker-compose的代码：

```yaml
// autobuild/docker-compose.yml
version: '3.1'
services:
  nginx:
    restart: always
    image: nginx
    ports:
      - 81:81
    volumes:
      - ./conf.d/:/etc/nginx/conf.d
      - /root/for-study-test/git-for-study/deploy_server/pm2-test/dist:/usr/share/nginx/test/
```

```js
// autobuild/conf.d/docker.conf
server {
  listen  81;
  location / {
    root    /usr/share/nginx/test;
    index   index.html index.htm;
    try_files $uri $uri/ /index.html;
  }
}
```













