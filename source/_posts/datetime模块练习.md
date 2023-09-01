---
title: datetime模块练习
tags:
  - datetime
  - MotherDay
  - Python
id: '120'
categories:
  - - Python练习
date: 2020-05-06 16:40:04
---

```
import datetime

oneday = datetime.timedelta(days = 1)

def sunday(date):
    while date.weekday() != 6:
        date += oneday
    return date

def MotherDay(year):
    mday = datetime.datetime(year, 5, 1)
    if mday.weekday() != 6:
        mday = sunday(mday + oneday)
    return  sunday(mday + oneday)

def IntervalstoMday(today):
    mday = MotherDay(today.year)
    if today.strftime('%Y%m%d') == mday.strftime('%Y%m%d'):
        print("Today is Mother's Day")
    elif today < mday:
        Intervals = mday - today
        print(f"Intervals to Mother's Day is {Intervals.total_seconds()/3600:.2f}h")
    else:
        Intervals = MotherDay(today.year + 1) - today
        print(f"Intervals to Mother's Day is {Intervals.total_seconds()/3600:.2f}h")

```