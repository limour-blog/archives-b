---
title: AWS的Lambda使用尝试
tags: []
id: '1995'
categories:
  - - uncategorized
date: 2022-08-23 12:18:06
---

之前[给博客尝试用了CloudFront](https://occdn.limour.top/2187.html)，今天来试试Lambda。

## 先给虚拟机装个conda

*   wget https://mirrors.tuna.tsinghua.edu.cn/anaconda/miniconda/Miniconda3-py39\_4.12.0-Linux-x86\_64.sh -O conda\_install.sh
*   chmod +x conda\_install.sh
*   ./conda\_install.sh
*   source ~/.bashrc
*   nano -K .condarc # [清华镜像源](https://mirrors.tuna.tsinghua.edu.cn/help/anaconda/)
*   conda clean -i

## 安装python3.9运行时

*   conda create -n aws\_py39 -c conda-forge python=3.9.13 -y
*   conda activate aws\_py39
*   conda install -c conda-forge zip -y

## 添加第三方python依赖

*   mkdir python
*   pip3 install --target ./python -i https://pypi.tuna.tsinghua.edu.cn/simple requests
*   zip -r py-requests.zip python
*   下载 py-requests.zip

![](https://img.limour.top/archives_2023/2022/08/23/6304b9102f98d.webp)

[创建层](https://aws.amazon.com/cn/blogs/china/use-aws-lambda-layer-function/)

## 创建函数

![](https://img.limour.top/archives_2023/2022/08/23/6304b9edf26b2.webp)

## 添加依赖层

![](https://img.limour.top/archives_2023/2022/08/23/6304bb0d72894.webp)

## 编写函数

```python
import json
import requests,base64
 
def getsend(wecom_cid, wecom_aid, wecom_secret):
    get_token_url = f"https://qyapi.weixin.qq.com/cgi-bin/gettoken?corpid={wecom_cid}&corpsecret={wecom_secret}"
    response = requests.get(get_token_url).content
    access_token = json.loads(response).get('access_token')
    if access_token and len(access_token) > 0:
        send_msg_url = f'https://qyapi.weixin.qq.com/cgi-bin/message/send?access_token={access_token}'
        def send(text, wecom_touid='@all'):
            data = {
                "touser":wecom_touid,
                "agentid":wecom_aid,
                "msgtype":"text",
                "text":{
                    "content":text
                },
                "duplicate_check_interval":600
            }
            response = requests.post(send_msg_url,data=json.dumps(data)).content
            return response
    else:
        def send(text, wecom_touid='@all'):
            print('get_token Failed')
    return send
 
### 以下请按实际情况自行配置，以上勿动 ###
 
'''
WECOM_CID企业微信公司ID
WECOM_AID企业微信应用ID
WECOM_SECRET企业微信应用Secret
wecom_touid消息发送对象的UID
'''
info = getsend( # 与Server酱中的参数一致
    wecom_cid = '***',
    wecom_aid = '***',
    wecom_secret = '***'
    )
wecom_touid = 'limour'
### 以上请按实际情况自行配置，以下勿动 ###
 
def lambda_handler(event, context):
    # TODO implement
    args = event.get('body')
    args = json.loads(args)
    if args.get('token') != 'a123456':
        return {
            'statusCode': 404,
            'body': json.dumps('invalid token!')
        }
    response = info(
        text = args.get('msg'),
        wecom_touid = args.get('touid')
        )
    body = {}
    body["event"] = event.get('body')
    return {
        'statusCode': 200,
        "isBase64Encoded": False,
        "headers": {"Content-Type" : "application/json"},
        'body': json.dumps(body)
    }
```

### 测试

```sh
/usr/bin/curl -X POST \
-H "Content-Type: application/json" \
-d '{"msg": "test msg", "touid":"limour", "token":"a123456"}' \
https://***.lambda-url.us-east-1.on.aws/
```