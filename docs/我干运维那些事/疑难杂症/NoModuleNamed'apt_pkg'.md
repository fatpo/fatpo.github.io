# 现象
ubuntu18.04中，敲一些不存在的命令时，比如 mqqq, 会报错：
```text
root@VM-0-10-ubuntu:~/www/# mqqq
Traceback (most recent call last):
  File "/usr/lib/command-not-found", line 28, in <module>
    from CommandNotFound import CommandNotFound
  File "/usr/lib/python3/dist-packages/CommandNotFound/CommandNotFound.py", line 19, in <module>
    from CommandNotFound.db.db import SqliteDatabase
  File "/usr/lib/python3/dist-packages/CommandNotFound/db/db.py", line 5, in <module>
    import apt_pkg
ModuleNotFoundError: No module named 'apt_pkg'
```
虽然不影响使用，但是我只是敲错一个命令，不至于给我这么一大段吧。

# 解决
```text
dpkg -L python3-apt | grep apt_pkg
```
根据出来的结果：
```text
ln -s /usr/lib/python3/dist-packages/apt_pkg.cpython-36m-x86_64-linux-gnu.so /usr/lib/python3/dist-packages/apt_pkg.so
```