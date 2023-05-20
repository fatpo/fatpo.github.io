## install 3.8

```text
apt install python3.8
update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.6 1
update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.8 2
update-alternatives --config python3
cd /usr/lib/python3/dist-packages/
sudo cp apt_pkg.cpython-36m-x86_64-linux-gnu.so apt_pkg.so
```

## install pip
```text
apt-get -y install python3-pip
apt-get install -y python3-setuptools
pip3 install firebase_admin  --ignore-installed httplib2
```


## reinstall pip if need

pip error:

```text
Pip is not working: ImportError: No module named 'pip._internal'
```

```text
curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
python3 get-pip.py --force-reinstall
```

pip3安装和卸载以及常用命令:

```text
卸载PIP: python3 -m pip uninstall pip
升级PIP已验证: pip3 install --upgrade pip
升级PIP未验证: python -m pip install --upgrade pip
```

手动下载安装指定版本:

```text
# 下载指定版本
wget https://pypi.python.org/packages/source/p/pip/pip-18.1.tar.gz
# 解压
tar -zxvf pip-18.1.tar.gz 
# 安装
cd pip-18.1
python3 setup.py build
python3 setup.py install
# 添加到软连接
ln -s /usr/local/python3/bin/pip3 /usr/bin/pip3
# 查看软连接
ll  /usr/bin/pip*
```