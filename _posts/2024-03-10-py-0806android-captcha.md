---
title: 0806手机验证码的自动化处理
date: 2024-03-10 21:00
categories: [python,网络爬虫开发实战]
tags: [python,网络爬虫开发实战,深度学,验证码] 
---

在android手机设备里用[SmsForwarder](https://github.com/pppscn/SmsForwarder)将接受到的短信，按照一定的规则转发到一个web接口。

## 发送规则

规则按照SmsForwarder的说明看看设置

## web接口

这个例子自己要做的也就是写个web服务。作者用Flask框架写了一个简单的将短信内容接收的web服务

```python
from flask 
import Flask, request, jsonifyfrom loguru 
import logger
app=Flask(_name)

@app.route('/sms', methods=['PosT'])
def receive():
    sms content = request.form.get('content')
    logger.debug(f'rece[sms content]')
    # 解析内容并将其保存到 db 或 mq
    return jsonify(status='success')

if __name__=='__main__':
    app.run(debu8-True)
```

## 暴露内网服务到公网

基本内网就可以搞定了，作者还介绍个[ngrok](https://ngrok.com/)（基本跟花生壳差不多的东西）来将内网服务暴露到公网。(打广告的嫌疑，算是开眼界了)

- 安装.这个东西得安装，还得注册设置自己的二级域名
- 工具命令

上面搞好了用命令:`ngrok http 5000`暴露 Flask的服务端口

## 批量接受短信验证的几种方式
- 1.机海战术

收集海量旧手机，插上sim卡。

- 2.硬件设备:卡池、猫池

- 3.接码平台
人家搞定设备、sim卡问题，你把验证码引流到解码平台，我给你api自取