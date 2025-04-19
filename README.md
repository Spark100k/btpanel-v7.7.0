# btpanel-v7.7.0
btpanel-v7.7.0-backup  官方原版v7.7.0版本面板备份

**Centos/Ubuntu/Debian安装命令 独立运行环境（py3.7）**

```Bash
curl -sSO https://raw.githubusercontent.com/Spark100k/btpanel-v7.7.0/main/install/install_panel.sh && bash install_panel.sh
```
或
```
curl -sSO https://raw.githubusercontent.com/8838/btpanel-v7.7.0/main/install/install_panel.sh && bash install_panel.sh
```

**备用安装链接，适用于不能访问GitHub的服务器。文件公开存放在[d.moe.ms](http://d.moe.ms/?btpanel-v7.7.0)**

```
curl -sSO http://d.moe.ms/AAAAA/btpanel-v7.7.0/install/install_panel.sh && bash install_panel.sh
```

# 手动破解：

1，屏蔽手机号

```
sed -i "s|bind_user == 'True'|bind_user == 'XXXX'|" /www/server/panel/BTPanel/static/js/index.js
```

2，删除强制绑定手机js文件

```
rm -f /www/server/panel/data/bind.pl
```

3，手动解锁宝塔所有付费插件为永不过期

文件路径：`/www/server/panel/data/plugin.json`

搜索字符串：`"endtime": -1`全部替换为`"endtime": 999999999999`

4，给plugin.json文件上锁防止自动修复为免费版

```
chattr +i /www/server/panel/data/plugin.json
```

============================

！！如需取消屏蔽手机号

```
sed -i "s|if (bind_user == 'REMOVED') {|if (bind_user == 'True') {|g" /www/server/panel/BTPanel/static/js/index.js
```

-------
# 注意：
推荐使用安装nginx1.24

宝塔nginx 反代的站点 计划任务SSL证书续期失败问题 

解决方法：停止反代，先初次申请证书启用后，开启反代，然后在站点修改-配置-找到
#一键申请SSL证书验证目录相关设置 添加 root /www/wwwroot/域名;   点保存即可在到期前自动续期。 修改后如下：
```
    #一键申请SSL证书验证目录相关设置
    location ~ \.well-known{
        root /www/wwwroot/域名;
        allow all;
    }
```

1，从宝塔面板7.7.0下载文件显示错误：

出错了，面板运行时发生错误！

TypeError: send_file() got an unexpected keyword argument 'add_etags'
![image](https://github.com/user-attachments/assets/6f45ff11-b531-4ece-b42b-350e9c0cc5b3)

原因：参数兼容性问题：在 Flask 2.x 版本中，send_file() 函数的参数更新，add_etags 参数不再使用。与较新 Flask 版本不兼容的代码中仍然使用了这个参数，导致函数调用失败。

解决宝塔面板7.7.0 Flask 版本不正确导致下载文件报错，降级 Flask 版本：

查看当前版本
```
btpip show flask
```
安装正确的版本（例如Flask 2.1.2）并重启面板
```
btpip install -U Flask==2.1.2 && bt 1
```

2，宝塔面板申请 SSL 证书时报错：Invalid version. The only valid version for X509Req is 0.

原因是X仅支持版本0，由于错误地设置了不适用的版本号，导致申请失败，服务器端的X509Req 版本只支持 0，而宝塔默认的版本为2。

解决方案：修改 /www/server/panel/class 中的 acme_v2.py 文件找到第973行，将 X509Req.set_version(2) 修改为：X509Req.set_version(0) 再重启面板。

或执行一键替换命令和重启面板：

备份当前的 acme_v2.py 文件
```
cp /www/server/panel/class/acme_v2.py /www/server/panel/class/acme_v2.py.bak
```
修改文件及重启面板
```
sed -i 's/X509Req.set_version(2)/X509Req.set_version(0)/g' /www/server/panel/class/acme_v2.py && bt 1
```

3，打开宝塔 SSH 终端显示 连接丢失,正在尝试重新连接!或光标一直闪烁。

问题原因：版本兼容性问题：较新版本的Flask和Werkzeug 改变了WebSocket 路由处理机制。由于路由机制改变，WebSocket 请求在默认情况下
被误识别为普通 HTTP 请求。路由标记问题：WebSocket 端点需要通过 websocket=True 标记来显式区分。缺少必要标记导致 WebSocket 请求处理错误。

解决方案：

修改 /www/server/panel/class 目录内的 flask_sockets.py 文件，将第78行
self.url_map.add(Rule(rule, endpoint=f))
修改为：
self.url_map.add(Rule(rule, endpoint=f, websocket=True))

或执行命令：
备份 flask_sockets.py 文件
```
cp /www/server/panel/class/flask_sockets.py /www/server/panel/class/flask_sockets.py.bak
```
修改文件重启面板
```
sed -i 's/self.url_map.add(Rule(rule, endpoint=f))/self.url_map.add(Rule(rule, endpoint=f, websocket=True))/g' /www/server/panel/class/flask_sockets.py && bt 1
```
