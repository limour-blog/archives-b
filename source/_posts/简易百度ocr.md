---
title: 简易百度OCR
tags:
  - OCR
  - Python
id: '150'
categories:
  - - Python练习
date: 2020-06-18 16:42:30
---

```
pip install baidu-aip #安装SDK
111.202.114.49 console.bce.baidu.com #改善宿舍的联通网络体验,hosts定向到最近的联通服务器
123.125.114.17 aip.baidubce.com
```

```
from aip import AipOcr

""" 你的 APPID AK SK """
APP_ID = '你的 App ID'
API_KEY = '你的 Api Key'
SECRET_KEY = '你的 Secret Key'

client = AipOcr(APP_ID, API_KEY, SECRET_KEY)
options = {}
options["language_type"] = "CHN_ENG"
options["detect_direction"] = "true"
options["detect_language"] = "true"

```

```
一个简单的应用, 去掉了关键信息,想使用请自行补全
from aip import AipOcr
import requests, time, re, random

space = re.compile(r'\s+')
htag = re.compile(r'<[^>]+>')

""" 你的 APPID AK SK """
APP_ID = ''
API_KEY = ''
SECRET_KEY = ''

client = AipOcr(APP_ID, API_KEY, SECRET_KEY)
options = {}
options["language_type"] = "CHN_ENG"
options["detect_direction"] = "true"
options["detect_language"] = "true"

def BaiduOCR(image, options=options):
    if isinstance(image, str):
        results = client.basicGeneralUrl(image, options)
    else:
        results = client.basicGeneral(image, options)
    if 'error_code' in results:
        print (f'BaiduOCR error {results["error_code"]} {results["error_msg"]}')
        return ''
    return ''.join(line['words'] for line in results['words_result'] if 'words' in line)
    
def jsonfy(s:str)->object:
    #此函数将不带双引号的json的key标准化
    assert s[0] in ('{','['), print(s) or 'jsonfy 登录失效,请重新获取Cookie!'
    obj = eval(s, type('js', (dict,), dict(__getitem__=lambda s, n: n))())
    return obj

def getScLc(d):
    return (int(d['sc']), int(d['lc']))

def getXkList(r):
    a = r.text.find('[')
    b = r.text.find(r';/*sc 当前人数, lc 人数上限*/')
    c = r.text.find('{', b)
    j = jsonfy(r.text[a:b])
    j2 = jsonfy(r.text[c:])
    return [(one['id'], one['no'], one['name'], getScLc(j2[str(one['id'])])) for one in j]

def getData(Form = None):
    if Form:
        r = requests.post('', data = Form, headers=headers)
    else:
        r = requests.get(r'', headers=headers)

    print(f'getData status_code {r.status_code}')
    return r

def getTimestamp(size = 1000):
    t = time.time()
    return int(round(t * size))

def getCaptcha():
    r = requests.get(f'', headers=headers)
    print(f'getCaptcha status_code {r.status_code}')
    img = r.content
    text = BaiduOCR(img)
    print(f'getCaptcha results {text}')
    return text.replace(' ','')

def Xk(CourseId):
    data = {
    "optype": "true",
    "operator0": "",
    "captcha_response": ""
    }
    r = requests.post(r'', data = data, headers=headers)
    print(f'Xk status_code {r.status_code}')
    return htag.sub(' ', space.sub('', r.text))

def Xk2S(CourseId):
    for i in range(3):
        st = Xk(CourseId)
        if '选课成功' in st:
            print(f'Xk2S 第{i+1}次选课成功!')
            return True
        else:
            print(f'Xk2S 第{i+1}次选课:')
            print(st)
            assert '选课失败:公选人数已满' not in st, 'Xk2S 公选人数已满 无法继续'
            assert '选课失败:你已经选过' not in st, 'Xk2S 选课成功 无需继续'
        time.sleep(random.randint(2,5))
    else:
        return False

Form = {
    "lessonNo": "",
    "courseCode": "",
    "courseName": ""
}

headers = {
    "Accept": "image/webp,image/apng,image/*,*/*;q=0.8",
    "Accept-Encoding": "gzip, deflate",
    "Accept-Language": "zh-CN,zh;q=0.9,en;q=0.8",
    "Connection": "keep-alive",
    "Cookie": "",
    "Host": "",
    "Referer": "",
    "User-Agent": ""
}

def getCourse():
    Courses = getXkList(getData(Form))
    print (Courses)
    for Course in Courses:
        if Course[1] == Form['lessonNo']:
            print (Course)
            return Course
    print('getCourse 出现未知错误,请重试!')

def isSuccessful():
    Courses = getXkList(getData())
    print (Courses)
    for Course in Courses:
        if Course[1] == Form['lessonNo']:
            print (Course)
            return True
    return False

```