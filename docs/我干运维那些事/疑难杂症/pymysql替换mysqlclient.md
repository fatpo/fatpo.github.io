## 背景
MySQLdb只支持Python2.x，还不支持3.x

总会报错：
```text
      building 'MySQLdb._mysql' extension
      creating build/temp.linux-x86_64-3.8
      creating build/temp.linux-x86_64-3.8/MySQLdb
      x86_64-linux-gnu-gcc -pthread -Wno-unused-result -Wsign-compare -DNDEBUG -g -fwrapv -O2 -Wall -g -fstack-protector-strong -Wformat -Werror=format-security -g -fwrapv -O2 -I/snap/certbot/2772/usr/include/python3.8/ -Wdate-time -D_FORTIFY_SOURCE=2 -fPIC -Dversion_info=(2,1,1,'final',0) -D__version__=2.1.1 -I/usr/include/mysql -I/usr/include/python3.8 -c MySQLdb/_mysql.c -o build/temp.linux-x86_64-3.8/MySQLdb/_mysql.o -std=c99
      In file included from /snap/certbot/2772/usr/include/python3.8/Python.h:8:0,
                       from MySQLdb/_mysql.c:46:
      /snap/certbot/2772/usr/include/python3.8/pyconfig.h:3:12: fatal error: x86_64-linux-gnu/python3.8/pyconfig.h: No such file or directory
       #  include <x86_64-linux-gnu/python3.8/pyconfig.h>
                  ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
      compilation terminated.
      error: command 'x86_64-linux-gnu-gcc' failed with exit status 1
      [end of output]

  note: This error originates from a subprocess, and is likely not a problem with pip.
error: legacy-install-failure
```

## 解决方案
可以用PyMySQL代替。安装方法：pip install PyMySQL
 然后在需要的项目中，在__init__.py中添加两行：
```text
import pymysql
pymysql.install_as_MySQLdb()
```

就可以用 import MySQLdb了。其他的方法与MySQLdb一样。


