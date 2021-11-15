# 1、介绍背景
django版本：
```
Django==3.2
```
搞了个项目，但是不能人一直盯着日志嘛，所以想着出了错误日志发邮件给我。

有感而发，写于 `2021年9月2日22:37:33`。


# 2、效果
我使用qq邮箱作为邮件服务器，所以要先去qq邮箱设置下允许SMTP服务。

![](.django发送错误日志邮件_images/fdaf23de.png)

收到错误邮件效果图：

![](.django发送错误日志邮件_images/a1154cc8.png)

# 3、django实操
settings.py增加邮箱配置：
```py
# 管理员邮箱-收件人
ADMINS = (
    ('fatpo', '1@qq.com'),
    ('ch', '2@sina.com'),
    ('jie', '3@qq.com'),
)
EMAIL_BACKEND = 'django.core.mail.backends.smtp.EmailBackend'
EMAIL_HOST = 'smtp.qq.com'  # QQ邮箱SMTP服务器(邮箱需要开通SMTP服务)
EMAIL_PORT = 25  # QQ邮箱SMTP服务端口
EMAIL_HOST_USER = '发送者@qq.com'  # 我的邮箱帐号
EMAIL_HOST_PASSWORD = '！！qq邮箱的授权码！！'  # 授权码  不是你的QQ密码
EMAIL_SUBJECT_PREFIX = '[django]'  # 为邮件标题的前缀,默认是'[django]'
EMAIL_USE_TLS = True  # 开启安全链接
DEFAULT_FROM_EMAIL = SERVER_EMAIL = EMAIL_HOST_USER  # 设置发件人
```

日志级别增加相关配置：
```py

import logging

LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'formatters': {
        'standard': {
            'format': '[%(levelname)s][%(asctime)s][%(filename)s][%(funcName)s][%(lineno)d] > %(message)s'
        },
        'simple': {
            'format': '[%(levelname)s]> %(message)s',
            'datefmt': '%Y-%m-%d %H:%M:%S'
        },
    },
    'handlers': {
        'console': {
            'level': 'DEBUG',
            'filters': None,
            'class': 'logging.StreamHandler',
            'formatter': 'simple'
        },
        'file_handler': {
            'level': 'INFO',
            'class': 'logging.handlers.TimedRotatingFileHandler',
            'filename': '%s/django.log' % LOG_DIR,
            'formatter': 'standard',
            'encoding': 'utf-8',
            'when': 'midnight',
            'interval': 1,
            'backupCount': 30,
        },  # 用于文件输出
        'mail_admins_handler': {
            'level': 'ERROR',
            'class': 'myapp.mail.MyEmailHandler',
            'formatter': 'standard',
            'include_html': True,
        },
    },
    'loggers': {
        'mdjango': {
            # 一个记录器中可以使用多个处理器
            'handlers': ['console', 'file_handler', 'mail_admins_handler'],
            'level': 'DEBUG',
            'propagate': True,
        },
        'django.request': {
            'handlers': ['mail_admins_handler'],
            'level': 'ERROR',
            'propagate': False,
        },
    }
}

logger = logging.getLogger("mdjango")
```
主要注意这个handler：
```py
'mail_admins_handler': {
    'level': 'ERROR',
    'class': 'myapp.mail.MyEmailHandler',
    'formatter': 'standard',
    'include_html': True,
},
```
把它应用在logger中：
```py
'mdjango': {
    # 一个记录器中可以使用多个处理器
    'handlers': ['console', 'file_handler', 'mail_admins_handler'],
    'level': 'DEBUG',
    'propagate': True,
},
```
当然，因为我们`mail_admins_handler`中选择了自定义的`myapp.mail.MyEmailHandler`，那么要给一个class：
路径：
```
/webRoot/myapp/mail.py
```
代码：
```py
from django.conf import settings
from django.core.mail.message import EmailMultiAlternatives
from django.utils.log import AdminEmailHandler


class MyEmailHandler(AdminEmailHandler):

    def emit(self, record):
        if not getattr(settings, "ADMINS", None):
            return
        if not getattr(record, "message", None):
            return

        subject = self.format_subject(record.getMessage())
        message = getattr(record, "email_body", record.getMessage())
        mail = EmailMultiAlternatives(u'%s%s' % (settings.EMAIL_SUBJECT_PREFIX, subject),
                    message, settings.SERVER_EMAIL, [a[1] for a in settings.ADMINS],)
        mail.send(fail_silently=False)
```

# 4、发送邮件避坑指南
## 4.1、坑1：打开了debug
```dtd
记得debug=False, 否则不会发
```

## 4.2、坑2：没有开smtp
```dtd
去百度怎么开，记得拿到授权码
```

## 4.3、坑3：使用默认的日志处理handler
```py
'mail_admins': {  # 发送邮件通知管理员
    'level': 'ERROR',
    'class': 'django.utils.log.AdminEmailHandler',
    'filters': ['require_debug_false'],  # 仅当 DEBUG = False 时才发送邮件
    'include_html': True,
},
```
这里不建议用默认的，因为默认的会把整个`settings.py`给带过去，非常长，而且暴露了很多重要配置，这是不对。

而且还不能定制，不好。比如默认的`django.utils.log.AdminEmailHandler`会触发两条ERROR错误，导致收到2次邮件。

我定制的代码，去掉其中一个没有message的邮件：
```py
if not getattr(record, "message", None):
    return
```

## 4.4、坑4：vultr没有开启25端口
这个我们不能自己开，要去vultr提工单。
![](./imgs/django发送错误日志邮件-1630593362395.png)

这里还有个小插曲，就是vultr说是给我开了25端口，但需要我自己重启。

我重启后，整个服务器所有的端口都无法telnet，只能另外开工单去反馈这件事，前前后后和vultr的工程师对线了2小时。
![](./imgs/django发送错误日志邮件-1630593406500.png)

哎。
