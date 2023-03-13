
# 国内本地调式chatGp接口


1.如果你的服务器可以直接访问就不用看了，
2.本地电脑安装了socks代理就不用看了

首先这里只针对国内服务器无法使用chatGpt接口,解决本地调式chatGp接口需要使用socks代理，你需要一个上网的tz，或者能上外网的vps
我使用的v2ray客户端，加bwg的梯子，废话不多说，我们开始，

下载v2ray的安装包。可以在v2ray官网的下载页面中找到最新版本的安装包。选择适用于您的系统的版本并下载。以Linux x86_64为例，您可以使用以下命令下载：
~~~ bash
wget https://github.com/v2fly/v2ray-core/releases/latest/download/v2ray-linux-64.zip
~~~
解压缩安装包。您可以使用以下命令将其解压缩到当前目录：
~~~ bash
unzip v2ray-linux-64.zip -d v2ray
~~~
进入v2ray目录并运行v2ray程序。例如，可以使用以下命令启动v2ray：
~~~ bash
cd v2ray
./v2ray
~~~
运行成功后，v2ray将会在后台运行，并输出日志信息。您可以通过在命令行中运行`tail -f /var/log/syslog | grep v2ray`命令来查看v2ray的日志信息

配置订阅地址。要配置订阅地址，您需要编辑v2ray的配置文件`config.json`。在该文件中，找到`inbounds`字段和`outbounds`字段，分别表示入站和出站连接。您可以使用以下配置示例作为参考：
~~~ json
{
  "inbounds": [
    {
      "port": 1080,
	  "listen": "0.0.0.0",
      "protocol": "socks",
      "settings": {
        "auth": "noauth",
        "udp": true
      },
      "streamSettings": {
        "network": "tcp"
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "vmess",
      "settings": {
        "vnext": [
          {
            "address": "your.server.address",
            "port": 443,
            "users": [
              {
                "id": "your_vmess_user_id",
                "alterId": 64
              }
            ]
          }
        ]
      },
      "streamSettings": {
        "network": "tcp",
        "security": "tls"
      }
    }
  ]
}
~~~
在此配置示例中，入站连接使用SOCKS协议，监听在本地端口`1080`上，不需要身份验证。出站连接使用VMess协议，并连接到指定的服务器地址和端口，需要提供VMess用户ID和AlterID。请将`address`和`id`字段替换为您的服务器地址和用户ID。

inbounds是入站配置可以参考上门的，outbounds是出站配置，将outbounds中的port替换服务器端口，请将`address`和`id`字段替换为您的服务器地址和用户ID。
配置文档可以参考 https://www.v2fly.org/config/routing.html 可能需要梯子tz

启动v2ray并使用客户端订阅配置。启动v2ray后，您可以使用v2ray客户端来订阅配置。例如，您可以使用v2rayN客户端，在其配置中添加v2ray订阅链接，并连接到服务器
服务器配置信息可以在后台查看

在Linux上可以使用curl命令来测试socks代理是否成功。具体步骤如下：
1.  打开终端，输入以下命令安装curl：
~~~ code
sudo apt-get install curl
~~~
2.  设置socks代理，其中127.0.0.1:1080是你的socks代理服务器地址和端口。例如：
~~~
export all_proxy=socks5://127.0.0.1:1080
~~~
3.  使用curl命令测试socks代理是否成功。例如，可以使用以下命令访问一个网站：
~~~
curl https://www.example.com/
~~~
如果代理设置正确，则可以看到网站内容；否则，将返回连接错误。

如果curl命令无法通过代理访问网站，则说明socks代理配置错误或代理服务器无法连接。可以检查代理服务器地址和端口是否正确，以及代理服务器是否正常运行

如果觉得麻烦那就简单
~~~
curl -x socks5://proxy-server-address:proxy-server-port http://example.com
~~~
将 `proxy-server-address` 和 `proxy-server-port` 替换为实际的代理服务器地址和端口，将 `http://example.com` 替换为需要访问的 URL。



现在我们还有重要一步让v2ray在linux后台运行

1.  创建一个服务文件
~~~
sudo nano /etc/systemd/system/v2ray.service
~~~
2.添加以下内容：
~~~
[Unit]
Description=V2Ray Service
After=network.target
[Service]
User=root
Group=root
ExecStart=/usr/local/bin/v2ray -config /etc/v2ray/config.json
Restart=always
LimitNOFILE=1000000
[Install]
WantedBy=multi-user.target
~~~

其中，`ExecStart` 是指定 `v2ray` 的启动命令和配置文件路径，这里默认配置文件路径为 `/etc/v2ray/config.json`。

3.  启动 V2Ray 服务
~~~
//启动
sudo systemctl start v2ray
//检查服务状态：
sudo systemctl status v2ray
//停止服务：
sudo systemctl stop v2ray
//启用服务开机自启
sudo systemctl enable v2ray
//禁用服务开机自启
sudo systemctl disable v2ray
~~~

ok 这样就可以让 V2Ray 在后台运行，并且在服务器启动时自动启动。
运行 
~~~
sudo systemctl start v2ray
~~~
然后就可能出现报错
The unit file, source configuration file or drop-ins of v2ray.service changed on disk. Run 'systemctl daemon-reload' to reload units.

这个错误提示是因为 v2ray.service 文件在磁盘上被修改，但是系统并没有加载最新的配置文件。可以通过执行 `systemctl daemon-reload` 命令来重新加载 systemd 的配置文件，使得 systemd 能够读取最新的配置并应用到服务中。

具体操作步骤如下：

 执行以下命令重新加载 systemd 的配置文件：
~~~
sudo systemctl daemon-reload
~~~
然后重启v2ray
~~~
systemctl restart v2ray
~~~
如果 v2ray 服务已经在运行，则应该会看到类似下面的输出：
~~~
● v2ray.service - V2Ray Service
   Loaded: loaded (/etc/systemd/system/v2ray.service; enabled; vendor preset: disabled)
   Active: active (running) since Mon 2022-03-21 10:21:22 UTC; 15s ago
 Main PID: 12345 (v2ray)
    Tasks: 8 (limit: 4915)
   Memory: 18.6M
   CGroup: /system.slice/v2ray.service
           ├─12345 /usr/bin/v2ray/v2ray -config /etc/v2ray/config.json
           └─12346 /usr/bin/v2ray/v2ray -config /etc/v2ray/config.json

Mar 21 10:21:22 example.com systemd[1]: Started V2Ray Service.
~~~
如果看到类似于 `loaded active running` 的信息，表示服务已经在后台运行。

现在代理已经配置好了，我们开始调试代码
1，下载laravel扩展包

![解决本地调式chatGp接口](https://cdn.learnku.com/uploads/images/202303/08/85912/AZhppInXUe.png!large)

2.配置laravel代理.
在.env文件中加入
~~~
HTTP_PROXY=socks5://你的服务器地址:10808
HTTPS_PROXY=socks5://你的服务器地址:10808
~~~

这里要注意端口要和你配置服务器一致的端口一致

3.本地调试.
~~~
$response = OpenAI::chat()->create([
            'model'    => 'gpt-3.5-turbo',
            'messages' => [
                ['role'      => 'user', 'content'   => '你好，现在你叫小张，今年十八岁，是个男生，生活在深圳，是一名程序员'],
                ['role'      => 'assistant', 'content'   => 'Hello, I am currently called Xiao Zhang. I am 18 years old and I am a male living in Shenzhen. I am a programmer'],
                ['role'      => 'user', 'content'   => '请用中文和我交流'],
                ['role'      => 'assistant', 'content'   => '好的，你有什么需要我帮忙的吗？'],
                ['role'      => 'user', 'content'   => '你叫什么名字？'],
                ['role'      => 'assistant', 'content'   => '您可以随意称呼我小张。'],
                ['role'      => 'user', 'content'   => '你今年多大了？'],
                ['role'      => 'assistant', 'content'   => '我今年的年龄是18岁'],
                ['role'      => 'user', 'content'   => '上一个问题是什么？'],
            ],
        ]);

        dd($response);
~~~

具体的其他使用方法和上下文对接使用还在研究





