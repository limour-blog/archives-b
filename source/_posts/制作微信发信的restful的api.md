---
title: 制作微信发信的RESTful的API
tags: []
id: '1974'
categories:
  - - uncategorized
date: 2022-08-13 16:45:58
---

境内微信使用太广泛了，然而微信这个倒霉玩意不像TG一样提供机器人接口，而学校核酸检测需要每天催没测的人去测核酸，太折磨了，因此先把微信发信的接口转成RESTful的API方便后续使用。主要思路是用**[ItChat-UOS](https://github.com/why2lyj/ItChat-UOS)**的接口，通过[Bottle](https://bottlepy.org/docs/dev/)暴露到互联网上，只要post对应的参数和消息就能通过**[ItChat-UOS](https://github.com/why2lyj/ItChat-UOS)**给指定群组发送通知。

## 解除系统限制

```cnofig
* soft nofile 100001
* hard nofile 100002
* soft nproc 65535
* hard nproc 65535
```

*   sudo -i
*   nano /etc/security/limits.conf
*   sysctl -w kernel.pid\_max=65535
*   reboot
*   ulimit -a

这个系统最大进程数限制简直太秀了，其他服务一切正常，ssh进不去，docker可视化面板不能停止docker，还好有这个可视化面板给出了正确的错误信息 [Resource temporarily unavailable](https://blog.csdn.net/qq_35963057/article/details/81489907)。一番所搜才知道是进程数量达到最大值了，我服了。。。

*   阿里云的机器默认的vm.swappiness居然是0，淦
*   nano /etc/sysctl.conf
*   sysctl -p #同时加到开机启动命令里
*   sysctl vm.swappiness=60
*   cat /proc/sys/vm/swappiness

## 安装依赖

*   sudo apt-get install python3-pip
*   pip3 install itchat-uos==1.5.0.dev0 -i https://pypi.tuna.tsinghua.edu.cn/simple
*   pip3 install bottle -i https://pypi.tuna.tsinghua.edu.cn/simple

## 测试itchat-uos

```python
def print_qr(fileDir):
    if config.OS == 'Darwin':
        subprocess.call(['open', fileDir])
    elif config.OS == 'Linux':
        # subprocess.call(['xdg-open', fileDir])
        print(os.getcwd())
        input(fileDir)
    else:
        os.startfile(fileDir)
```

*   nano /home/pi/.local/lib/python3.7/site-packages/itchat/utils.py
*   修改print\_qr的代码，改为输出QR图片的路径，自行下载扫码

*   python3
*   import itchat
*   \# itchat.auto\_login()
*   \# itchat.auto\_login(True, enableCmdQR=True)
*   itchat.auto\_login(True, enableCmdQR=2)
*   itchat.send('Hello, Limour', toUserName='filehelper')
*   itchat.logout() # 建议手机上点退出

## 获得好友和群组的UserName

*   import itchat
*   itchat.auto\_login(True, enableCmdQR=2)
*   usr = itchat.search\_friends(remarkName='Limour')
*   group = itchat.search\_chatrooms(name='相亲相爱一家人')
*   group = group\[0\]
*   group.send('测试一下')
*   group.userName 即可得到群组的userName

## 查看消息的结构

*   平时我们分享的小程序属于'Sharing'
*   一般的文字消息属于'Text'
*   让我们捕获一个'Sharing'试试

```python
import itchat
@itchat.msg_register(['Sharing'])
def mm_reply(msg):
    global msg_catch
    msg_catch = msg
    print(msg_catch)
    return u'收到分享' + msg['Text']
 
itchat.auto_login(True, enableCmdQR=2)
itchat.run()
```

*   很好，现在我们获得了一个Sharing样本
*   通过研究它，我们尝试构建自己的Sharing消息
*   先保存我们珍贵的样本

```python
import pickle
with open("myDictionary.pkl", "wb") as tf:
    pickle.dump(msg_catch,tf)
with open("myDictionary.pkl", "rb") as tf:
    new_msg = pickle.load(tf)
```

*   nano /home/pi/.local/lib/python3.7/site-packages/itchat/components/messages.py
*   nano /home/pi/.local/lib/python3.7/site-packages/itchat/\_\_init\_\_.py
*   forward\_appMsg = instance.forward\_appMsg

```python
def forward_appMsg(self, msg, toUserName):
    url = '%s/webwxsendmsg' % self.loginInfo['url']
    data = {
        'BaseRequest': self.loginInfo['BaseRequest'],
        'Msg': {
            'MsgType': msg['MsgType'],
            'Type': msg['Type'],
            'Content': msg['Content'],
            'AppMsgType': msg['AppMsgType'],
            'FileName': msg['FileName'],
            'Url': msg['Url'],
            'EncryFileName': msg['EncryFileName'],
            'Text': msg['Text'],
            'FromUserName': self.storageClass.userName,
            'ToUserName': (toUserName if toUserName else self.storageClass.userName),
            'LocalID': int(time.time() * 1e4),
            'ClientMsgId': int(time.time() * 1e4),
            },
        'Scene': 0, }
    headers = { 'ContentType': 'application/json; charset=UTF-8', 'User-Agent' : config.USER_AGENT }
    r = self.s.post(url, headers=headers,
        data=json.dumps(data, ensure_ascii=False).encode('utf8'))
    return ReturnValue(rawResponse=r)

def load_messages(core):
    core.send_raw_msg = send_raw_msg
    core.send_msg     = send_msg
    core.upload_file  = upload_file
    core.send_file    = send_file
    core.send_image   = send_image
    core.send_video   = send_video
    core.send         = send
    core.revoke       = revoke
    core.forward_appMsg = forward_appMsg
```

*   重启python，测试一下

```python
import pickle
with open("myDictionary.pkl", "rb") as tf:
    new_msg = pickle.load(tf)
 
import itchat
itchat.auto_login(True, enableCmdQR=2)
group = itchat.search_chatrooms(name='相亲相爱一家人')
group = group[0]
group.userName
 
itchat.forward_appMsg(msg=new_msg, toUserName=group.userName)
```

*   淦，失败了，不弄了，换成二维码图片算了，微信就是\*\*\*\*
*   测试一下图片功能：itchat.send\_image('./itchat/COVID.19.testing.jpg', group.userName)
*   算了，只能小程序卡片换成二维码图片了

## 构建API

```python
#!/usr/bin/python3
import itchat
itchat.auto_login(True, enableCmdQR=2)
 
import bottle
@bottle.route('/api/send_image', method='POST')
def api_send_image():
    token = bottle.request.json.get('token')
    print(token)
    print(bottle.request.json)
    if token != 'a123456':
        return dict(BaseResponse='invalid token!')
    fileDir = bottle.request.json.get('fileDir')
    toUserName = bottle.request.json.get('toUserName')
    return dict(itchat.send_image(fileDir, toUserName))
bottle.run(host='localhost', port=2001, debug=True)
```

### 测试API

```bash
curl -X POST \
-H "Content-Type: application/json" \
-d '{"fileDir": "./itchat/COVID.19.testing.jpg", "toUserName":"@@xxxxx", "token":"a123456"}' \
http://localhost:2001/api/send_image
```