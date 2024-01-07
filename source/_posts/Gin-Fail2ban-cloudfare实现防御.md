---
title: Gin+Fail2ban+Cloudfare实现防爆破扫描
categories: 捣鼓分享
keywords: 实现防爆破
top_img: /img/783705.png
cover: /img/783710.png
abbrlink: a637161
date: 2022-02-25 10:00:58
---

##  前言：

最近受某学长刺激，授权爆破他的网站，短时间内就被他的网站防住了～

好滴～ 最近又开始捣鼓服务器了，我也想搞～ 所以我就简单说一下我的实现。

{% note green 'fas fa-bullhorn' flat %}
以下我简单说下我的思路来**实现防爆破与CC等攻击**：👇

1. 通过Cloudfare获取到原始IP
2. Fail2ban读取日志
3. 回调通知Cloudfare进行防御

要用到的有：`Gin框架`与` nginx服务器`与`cloudfare免费版`与`Fail2ban`工具
{% endnote %}


##  获取到原始IP

其实有个思路是已经**有模块与nginx进行二进制编译后**

`access日志`就已经是原始IP了，但是刚好我是后端接口怼的CF

那么我为什么不写个中间件然后模拟Nginx日志结构，同样实现日志。

说干就干，下面直接贴代码： （**实现了高并发下的读写**）

```go
package middleware

import (
	"GoBackEnd/common" // 这里自己改下
	"fmt"
	"github.com/gin-gonic/gin"
	"io"
	"io/ioutil"
	"os"
	"sync"
	"time"
)

// NginxLogMiddleware 为记录CF原始IP
func NginxLogMiddleware() gin.HandlerFunc {
	return func(c *gin.Context) {
		var bodySize int
		// todo 拿到目录路径
		nginxPath := common.Path("access.log") // 自己写个函数获取哈～
		// todo 拿到将要合成的信息
		ip := c.ClientIP()
		utcTimeStr := time.Now().Format("02/Jan/2006:15:04:05")
		url := c.Request.URL
		method := c.Request.Method
		status := c.Writer.Status()
		body, err := ioutil.ReadAll(c.Request.Body)
		http := c.Request.Proto
		if err != nil {
			bodySize = -1
		}
		bodySize = len(body)
		header := c.Request.UserAgent()
		// todo 写入标准格式
		res := fmt.Sprintf("%v - - [%v +8000] \"%v %v %v\" %v %v \"-\" \"%v\"", ip, utcTimeStr, method, url, http, status, bodySize, header) // 因为是东八区我就写死了
		// todo 写入文件
		file, _ := os.OpenFile(nginxPath, os.O_WRONLY|os.O_APPEND, 0666)
		defer file.Close()
		wr := &SyncWriter{sync.Mutex{}, file} // 添加上锁
		wg := sync.WaitGroup{}
		wg.Add(1)
		go func(content string) { // fork子协程去写
			fmt.Fprintln(wr, content)
			wg.Done()
		}(res)
		wg.Wait()
		c.Next()
	}
}

type SyncWriter struct {
	m      sync.Mutex
	Writer io.Writer
}

func (w *SyncWriter) Write(b []byte) (n int, err error) {
	w.m.Lock()
	defer w.m.Unlock()
	return w.Writer.Write(b)
}
```

<img src="/img/mics/image-20220225101411647.webp" alt="" style="zoom:67%;" />

然后直接放到默认路由里面，所有流量都要走～

<img src="/img/mics/image-20220225101614873.webp" alt="" style="zoom:67%;" />

<img src="/img/mics/image-20220225101648740.webp" alt="" style="zoom:67%;" />

##  Fail2ban的安装与设置

```bash
$ sudo apt install fail2ban
$ systemctl stop fail2ban
$ cd /etc/fail2ban
```

这里说下Fail2ban的加载文件规则：依次进行

- `/etc/fail2ban/jail.conf`
- `/etc/fail2ban/jail.d/*.conf`
- `/etc/fail2ban/jail.local`
- `/etc/fail2ban/jail.d/*.local`

我们需要做的就是: 分别写入匹配规则与拉黑设置

```bash
$ cd /etc/fail2ban/jail.d
$ vim nginxcc.conf # 开始写入
##############################
[nginxcc]                                                                                                             
enabled = true                                                                                                        
port = http,https  # 禁止80 443                                                                                                   
filter = nginxcc      # 使用的是哪个                                                                                                
logpath = /home/ubuntu/HengY1Service/access.log # 框架里面配置的日志                                                       
bantime = 1200    # 被禁止多久                                                                                   
findtime = 30     # 扫描多长时间的                                                                                   
maxretry = 10     # 单位时间内次数                                                                                      
action = %(action_mwl)s   # 调用cloudfare内置脚本                                                                                      
         custom-cloudflare   # custom-cloudflare是后面要自己写的
         
:wq
##############################
$ cd /etc/fail2ban/filter.d
$ vim nginxcc.conf
##############################
[Definition]
failregex =(?i)^<HOST> .* "(GET|POST|HEAD).*HTTP.*" (404|503) .*$
ignoreregex =.*(robots.txt|favicon.ico|jpg|png|webp|js|css)

:wq
##############################
```

然后验证下自己写对没：access.log最好有点东西

```bash
$ fail2ban-regex /home/ubuntu/HengY1Service/access.log /etc/fail2ban/filter.d/nginxcc.conf
$ systemctl restart fail2ban
$ fail2ban-client status # 看下nginxCC开启没
$ fail2ban-client status nginxcc
$ fail2ban-client set nginxcc unbanip 1.1.1.1 # 解封你要的IP
```

<img src="/img/mics/image-20220225102817422.webp" alt="" style="zoom:80%;" />

##  通知Cloudfare进行防御

到这里的话还是不行的，因为你虽然拿到了源IP，但是是过CDN过来的IP你没有封

但是Fail2ban已经考虑到了，提供了脚本。这就是为什么要设置`action`

```bash
$ cd /etc/fail2ban/action.d
$ cp cloudflare.conf custom-cloudflare.conf
$ vim custom-cloudflare.conf # 翻看到最下面
# 填写 cftoken 与 cfuser cfuser就是你注册的邮件 cftoken 是GlobalToken
$ systemctl restart fail2ban
```

<img src="/img/mics/image-20220225103713641.webp" alt="" style="zoom:67%;" />

然后可以自己试试效果了～  （我的是：api.hengy1.top） 可以来试试效果

<img src="/img/mics/image-20220225103819706.webp" alt="" style="zoom:67%;" />

<img src="/img/mics/image-20220225104523559.webp" alt="" style="zoom:67%;" />

##  总结：

逼逼两句：

1. 构造日志参数与高并发读写要注意
2. 正则匹配必须有\<HOST\>，正则测试https://regex101.com/
3. 一定要CF来防御，思路得先串通
4. Fail2ban有点反应延迟