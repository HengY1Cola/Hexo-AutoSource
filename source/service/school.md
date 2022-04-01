---
title: 学校教务服务
date: 2022-04-02 00:01:33
top_img: /img/service.webp
aside: false
comments: false
---

## 接口一：查询课表
接口路径：/school/schedule
请求方式：POST
参数请求：{ "account"="xxx" , "password"="xxx" }
接口示例：https://api.hengy1.top/school/schedule
```json
{
    "errno": 0,
    "errmsg": "",
    "data": {
        "total": 数量,
        "scheduleDetail": [
            {
                "From": "学院",
                "Teacher": "老师",
                "Long": "那些周上课",
                "Week": "星期几",
                "Which": "多少节",
                "Where": "在哪里上课"
            }
        ],
        "tips": ""
    },
    "trace_id": "",
    "stack": null
}
```

## 错误接口
```json
{
    "errno": 错误代码,
    "errmsg": "错误消息",
    "data": "",
    "trace_id": "",
    "stack": ""
}
```