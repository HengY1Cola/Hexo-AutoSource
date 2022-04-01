---
title: 宿舍水费预警系统
date: 2022-02-24 10:07:33
top_img: /img/2407565.webp
aside: false
comments: false
---

##  价格表
|   描述   |    价格     |
| :------: | :---------: |
| 电费单价 | 0.6259元/度 |
| 冷水单价 |  2.7元/吨   |
| 热水单价 |   25元/吨   |
| 电费补贴 |    40度/月     |
| 冷水补贴 |    10吨/月     |
| 热水补贴 |    10吨/月     |

[^注意]: 该价格表更新于2022-02


## 接口一：查询余额等信息
接口路径：/IBSJnuWeb/balance
请求方式：GET
参数请求：dormitory
接口示例：https://api.hengy1.top/IBSJnuWeb/balance?dormitory=t20222
```json
{
    "errno": 0,
    "errmsg": "",
    "data": {
        "dormitory": "宿舍号",
        "balance": 余额,
        "notice": "距离上次充值时间",
        "electricitySubsidy": "电费剩余补贴",
        "coldSubsidy": "冷水剩余补贴",
        "hotSubsidy": "热水剩余补贴"
    },
    "trace_id": "无用",
    "stack": null
}
```

## 接口二：查询消费记录信息
接口路径：/IBSJnuWeb/stat
请求方式：GET
参数请求：dormitory
接口示例：https://api.hengy1.top/IBSJnuWeb/stat?dormitory=t20222
```json
{
    "errno": 0,
    "errmsg": "",
    "data": {
        "Electric": {
            "electricInfo": [电费信息],
            "electricDate": [电费日期]
        },
        "ColdWater": {
            "coldWaterInfo": [冷水信息],
            "coldWaterDate": [冷水日期]
        },
        "HotWater": {
            "hotWaterInfo": [热水信息],
            "hotWaterDate": [热水日期]
        }
    },
    "trace_id": "",
    "stack": null
}
```

## 接口三：注册邮箱/取消邮箱
接口路径：/IBSJnuWeb/email
请求方式：GET
参数请求：email&dormitory
接口示例：https://api.hengy1.top/IBSJnuWeb/email?email=xxxx@xxx&dormitory=t20222
```json
{
    "errno":0,
    "errmsg":"",
    "data":"消息",
    "trace_id":"",
    "stack":null
}
```

## 接口四：验证邮箱
接口路径：/IBSJnuWeb/verify
请求方式：GET
参数请求：token
接口示例：https://api.hengy1.top/IBSJnuWeb/verify?token=xxx

## 接口五：余额充值
接口路径：/IBSJnuWeb/charge
请求方式：GET
参数请求：dormitory&room&money
接口示例：https://api.hengy1.top/IBSJnuWeb/charge?dormitory=t11&room=0222&money=1
> Success消息请Base64解密然后生成二维码即可使用

```json
{
    "errno":0,
    "errmsg":"",
    "data":
        {
            "Success":"base64加密",
            "Warning":""
        },
    "trace_id":"",
    "stack":null
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
