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
                "Long": "周数",
                "Week": "星期",
                "Which": "节数",
                "Where": "地点"
            }
        ],
        "tips": ""
    },
    "trace_id": "",
    "stack": null
}
```

## 接口二：查询考试安排
接口路径：/school/exam
请求方式：POST
参数请求：{ "account"="xxx" , "password"="xxx" }
接口示例：https://api.hengy1.top/school/exam
```json
{
    "errno": 0,
    "errmsg": "",
    "data": {
        "total": 6,
        "examDetail": [
            {
                "What": "科目",
                "Teacher": "老师",
                "Where": "考试地点",
                "When": "考试时间",
                "Sit": "座位"
            }
        ],
        "tips": ""
    },
    "trace_id": "",
    "stack": null
}
```

## 接口三：查询我的成绩
接口路径：/school/score
请求方式：POST
参数请求：{ "account"="xxx" , "password"="xxx" }
接口示例：https://api.hengy1.top/school/score
```json
{
    "errno": 0,
    "errmsg": "",
    "data": {
        "gradeDetail": {
        "grade": 总平均绩点,
        "score": [
            {
            "name": "科目",
            "pass": "是否通过",
            "score": 学分,
            "value": 成绩,
            "when": "考试时间"
            }
        ]
        },
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