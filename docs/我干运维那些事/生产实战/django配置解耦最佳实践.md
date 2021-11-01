# 1、背景
众所周知，django的配置文件都写在 `settings.py`，随着项目配置越来越多，文件越来越臃肿，也很难找到配置和管理。

# 2、django-split-settings 帮你解耦
版本号：
```dtd
# pip3 freeze | grep split
django-split-settings==1.1.0
```

默认的 `settings.py`：
```dtd
from split_settings.tools import include

LOCAL_DEV = False

include(
    'settings_base.py',
    'settings_security.py',
    'settings_logger.py',
    'settings_mail.py',
    'settings_biz.py',
    'settings_apple.py',
    'settings_google.py',
    'settings_huawei.py',
    'settings_cache.py',
    'settings_database.py',
)
```

然后在和 `settings.py`同一个目录下，新增上面几个配置文件，比如：`settings_logger.py`:
```dtd
import os

# Build paths inside the project like this: os.path.join(BASE_DIR, ...)
BASE_DIR = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))

##--------------日志模块-------------
LOG_DIR = os.path.join(BASE_DIR, "logs")
if not os.path.exists(LOG_DIR):
    os.mkdir(LOG_DIR)

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
            'formatter': 'standard'
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
            'class': 'MyApp.mail.MyEmailHandler',
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
其他的代码都不需要更改，用起来无感知，解耦成功。

# 3、参考
* [GITHUB: django-split-settings](https://github.com/sobolevn/django-split-settings)
* [DJANGO十大不能忍的事](https://www.toptal.com/django/django-top-10-mistakes)