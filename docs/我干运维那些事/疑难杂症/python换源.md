## 各种源合集
* 阿里云 https://mirrors.aliyun.com/pypi/simple/ 
* 中国科技大学 https://pypi.mirrors.ustc.edu.cn/simple/ 
* 豆瓣(douban) https://pypi.douban.com/simple/ 
* 清华大学 https://pypi.tuna.tsinghua.edu.cn/simple/ 
* 中国科学技术大学 https://pypi.mirrors.ustc.edu.cn/simple/
* 官方源 https://pypi.python.org/simple

## 临时使用方法

```text
pip install scrapy -i  http://mirrors.aliyun.com/pypi/simple/ 
``` 

##  linux 永久换源方法
修改 ~/.pip/pip.conf (没有就创建一个)， 内容如下：
```text
[global]
index-url =  http://mirrors.aliyun.com/pypi/simple/ 
```