---
title: 多线程模块threading的简单使用
tags:
  - threading
  - 选课
id: '154'
categories:
  - - Python练习
date: 2020-06-19 12:36:19
---

```
from aip import AipOcr
import requests, time, re, random, threading

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
    if s[0] not in ('{','['): print(s + '\n' + 'jsonfy 登录失效,请重新获取Cookie!')
    try:
        obj = eval(s, type('js', (dict,), dict(__getitem__=lambda s, n: n))())
        return obj
    except Exception:
        return {}
    

def getScLc(d):
    return (int(d['sc']), int(d['lc']))

def getXkList(r):
    a = r.text.find('[')
    b = r.text.find(r';/*sc 当前人数, lc 人数上限*/')
    c = r.text.find('{', b)
    print(f'getXkList {a} {b} {c}')
    if b==-1 or c==-1: print(r.text); time.sleep(0.5); return
    j = jsonfy(r.text[a:b])
    j2 = jsonfy(r.text[c:])
    return [(one['id'], one['no'], one['name'], getScLc(j2[str(one['id'])])) for one in j]

def getData(Form = None):
    if Form:
        r = requests.post('', data = Form, headers=headers)
    else:
        r = requests.get(r''
    print(f'getData status_code {r.status_code}')
    if '请不要过快点击' in r.text: time.sleep(random.randint(3,9)/3)
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

def Xk(CourseId, lock):
    print(f'Xk 怎么会死锁? {lock.locked()}')
    with lock:
        data = {
        "optype": "true",
        "operator0": f"{CourseId}:true:0",
        "captcha_response": getCaptcha()
        }
        r = requests.post(r'', data = data, headers=headers)
    print(f'Xk status_code {r.status_code}')
    return htag.sub(' ', space.sub('', r.text))

def Xk2S(CourseId, lock):
    i = 3
    ord = 1
    while i >= 0:
        st = Xk(CourseId, lock)
        if '选课成功' in st:
            print(f'Xk2S 第{ord}次选课成功!')
            return True
        else:
            print(f'Xk2S 第{ord}次选课:')
            print(st)
            assert '选课失败:公选人数已满' not in st, 'Xk2S 公选人数已满 无法继续'
            assert '选课失败:你已经选过' not in st, 'Xk2S 选课成功 无需继续'
            if '操作失败:验证码错误' not in st: i -= 1
        ord += 1
    else:
        return False
#PHPM110047.01

from headers import headers

def getCourse(Form):
    Courses = getXkList(getData(Form))
    while not Courses:
        print(f'getCourse 想必是要死循环了 {Courses}')
        time.sleep(random.randint(5,16)/2)
        Courses = getXkList(getData(Form))
        
    print (Courses)
    for Course in Courses:
        if Course[1] in Form['lessonNo']:
            print (Course)
            return Course
    assert False, 'getCourse 出现未知错误,请重试!'

def isSuccessful(Form):
    time.sleep(5)
    Courses = getXkList(getData())
    print (Courses)
    for Course in Courses:
        if Course[1] == Form['lessonNo']:
            print (Course)
            return True
    return False

#Form['lessonNo'] = 'PEDU110091.19'

def start(lessonNo, lock, event):
    Form = {
    "lessonNo": lessonNo,
    "courseCode": "",
    "courseName": ""
    }
    time.sleep(random.randint(0,50)/10)
    print(f'start {lessonNo} {id(lock)} {id(event)}')
    while True:
        try :
            Course = getCourse(Form)
            if Course[3][0] < Course[3][1]:
                if Xk2S(Course[0], lock) and isSuccessful(Form):
                    print(f'主循环 选课成功 {time.ctime(time.time())}')
                    break
                else:
                    print(f'主循环 选课失败 继续尝试 {time.ctime(time.time())}')
            else:
                print(f'主循环 人数已达上限 继续等待 {time.ctime(time.time())}')
            event.set()
            time.sleep(random.randint(20,50)/5)
        except requests.ConnectionError:
            print('网络超时, 延时等待1分钟')
            for i in range(6):
                event.set()
                time.sleep(10)

printf = print
def wrapprint(*args, **kw):
    printf(threading.current_thread().name, ":",end='')
    printf(*args, **kw)
print = wrapprint





```

```
import threading, time, ctypes, inspect

def _async_raise(tid, exctype):
    """raises the exception, performs cleanup if needed"""
    tid = ctypes.c_long(tid)
    if not inspect.isclass(exctype):
        exctype = type(exctype)
    res = ctypes.pythonapi.PyThreadState_SetAsyncExc(tid, ctypes.py_object(exctype))
    if res == 0:
        #raise ValueError("invalid thread id")
        return "ValueError: invalid thread id"
    elif res != 1:
        # """if it returns a number greater than one, you're in trouble,
        # and you should call it again with exc=NULL to revert the effect"""
        ctypes.pythonapi.PyThreadState_SetAsyncExc(tid, None)
        #raise SystemError("PyThreadState_SetAsyncExc failed")
        return "SystemError: PyThreadState_SetAsyncExc failed"
    return "success!"
def startThread(tasks, timeout=35):
    lock = threading.Lock()
    tasks = [(target, args, threading.Event()) for target,args in tasks]
    #for task in tasks: task['args'] = task.get('args',()) + (lock, watchdog)
    threads = [[threading.Thread(target=target, args=args+(lock, event)), event] for target,args,event in tasks]
    #if threads: timeout = timeout // len(threads) + 1
    for t,event in threads: t.setDaemon(True)
    for t,event in threads: t.start()
    print('startThread 守护线程开始执行!')
    counter = 1
    while any(t.is_alive() for t,event in threads):
        print(f'startThread 守护线程第{counter}轮 开始 {time.ctime(time.time())}')
        time.sleep(timeout)
        for t,event in threads:
            if not event.isSet() and t.is_alive(): break
            time.sleep(0.1)
        else: print(f'startThread 守护线程第{counter}轮 正常 {time.ctime(time.time())}'); counter += 1;continue
        print(f'startThread 守护线程第{counter}轮 发生异常,准备重启... {time.ctime(time.time())}')
        lock = threading.Lock()
        restart = []
        for i,one in enumerate(threads):
            t, event = one
            if not t.is_alive(): print(f'startThread 线程自然退出,忽略 {t.name}'); continue
            if event.isSet(): event.clear()
            else:
                print(f'startThread 正在结束线程 {t.name} {_async_raise(t.ident, SystemExit)}')
                target,args,event = tasks[i]
                t = threads[i][0] = threading.Thread(target=target, args=args+(lock, event))
                t.setDaemon(True)
                restart.append(t) 
        for t in restart: t.start(); print(f'startThread 启动替代线程 {t.name}')
        del restart
    print('startThread 守护线程准备退出中...')
    for t,event in threads:
        if t.is_alive(): print(f'startThread 正在结束线程 {t.name} {_async_raise(t.ident, SystemExit)}')
    print('startThread 守护线程已退出!')

from FDUxk1 import start

cl = [] 
#cl.append('PEDU110091.19') #乒乓球
cl.append('PHPM110047.01') #医院经营管理
#cl.append('TOUR110001.01') #旅游与经济管理
cl.append('PTSS110016.01') #当代世界经济与政治
cl.append('PHIL119056.01') #哲学视野中的人工智能

```