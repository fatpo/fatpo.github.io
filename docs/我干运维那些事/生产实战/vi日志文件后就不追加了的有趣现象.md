# 1、介绍背景
阴差阳错，对`/var/log/nginx/access.log` 日志进行删除，删除我的两个测试请求。

结果，下午看了下，发现这个文件已经有3小时没更新了，我擦嘞？ 最后的更新时间刚好是我vi后。

我反复确认了，理论上这3小时内有不少请求进来的，不应该是0，那么问题就出在了这个vi上面。

# 2、做个测试
写个python文件:
```python
import time

if __name__ == '__main__':
    with open("1.txt", "w") as f:
        for i in range(300):
            f.write(f"{i}\n")
            f.flush()
            time.sleep(1)

```

这边等脚本跑10来秒后，对 `1.txt`进行vi，随机删除两条，此时tail的窗口依旧能看到日志在不断地追加。

但是如果我关闭tail命令，直接cat操作：
```
root@fatpo: # cat 1.txt
0
1
2
3
4
8
9
10
11
12
13
14
15
16
```
理论上应该要到100的，在16这里戛然而止了。

好吧，上午的场景复现了。

# 3、为什么
尝试好了好多搜索：
```
vim lost new log
why my new log lost after i vim
why my log lost after i vi
vi change fd
```
其实这个不好搜索，稍微看到一个答案，来自参考1：
```
If you accidentally write to a log file you may lose any new log file entries made since you opened it 
(although vim may in some cases warn you of this).
```
但还是没说明原理。

到此为止吧，大概率就是fd被替换了，新的日志都去了那个被vi弄出来的新fd。

不要太在意细节，写这篇只是想和大家分享一个有趣的vim现象，仅此而已。

# 4、参考
* [Is using vi to view log files in PROD servers not recommended?](https://askubuntu.com/questions/321751/is-using-vi-to-view-log-files-in-prod-servers-not-recommended)