---
title: 安装 shadowsocks client
date: 2019-12-02 22:00:21
categories:
- 配置
tags:
- shadowsocks
---
安装 shadowsocks 本地客户端记录

## 安装步骤

查看是否已经安装 python 和 pip, 这里用的是 python3

```bash
python3 -v
pip3 -v

# 如果没有安装运行
sudo apt-get install python3
sudo apt-get install python3-pip
```

配置 douban 源加速

```bash
# 跳转到 $HOME 目录下
cd
# 创建 .pip 目录
mkdir .pip
# 创建 config 文件
vim pip.conf
# 将如下内容写进 config 文件，保存退出
[global]
timeout = 60
index-url = http://pypi.douban.com/simple
trusted-host = pypi.douban.com
```

安装 python shadowsocks 包

```bash
sudo pip3 install shadowsocks

# 如果嫌 sudo 累赘，也可以用
pip3 install --user shadowsocks
```

安装完毕，配置本地 client 端，创建文件 `ssclient.json`（名字可以自选，不一定要这个），写入内容

```bash
{
"server":"xxx.xxx.xxx.xxx",
"server_port":8989,
"local_address": "127.0.0.1",
"local_port":1080,
"password":"xxxx",
"timeout":300,
"method":"aes-256-cfb",
"fast_open": false,
"workers": 1
}
```

端口信息根据实际情况修改，完毕后运行 `sslocal -c /path/to/ssclient.json`，报错

```bash
jack@DESKTOP-9TGTFK1:~/ss$ sslocal  -c ssclient.json
INFO: loading config from shadowsocks.json
2019-12-02 21:02:09 INFO     loading libcrypto from libcrypto.so.1.1
Traceback (most recent call last):
  File "/usr/local/bin/sslocal", line 11, in <module>
    load_entry_point('shadowsocks==2.8.2', 'console_scripts', 'sslocal')()
  File "/usr/local/lib/python3.6/dist-packages/shadowsocks/local.py", line 39, in main
    config = shell.get_config(True)
  File "/usr/local/lib/python3.6/dist-packages/shadowsocks/shell.py", line 262, in get_config
    check_config(config, is_local)
  File "/usr/local/lib/python3.6/dist-packages/shadowsocks/shell.py", line 124, in check_config
    encrypt.try_cipher(config['password'], config['method'])
  File "/usr/local/lib/python3.6/dist-packages/shadowsocks/encrypt.py", line 44, in try_cipher
    Encryptor(key, method)
  File "/usr/local/lib/python3.6/dist-packages/shadowsocks/encrypt.py", line 83, in __init__
    random_string(self._method_info[1]))
  File "/usr/local/lib/python3.6/dist-packages/shadowsocks/encrypt.py", line 109, in get_cipher
    return m[2](method, key, iv, op)
  File "/usr/local/lib/python3.6/dist-packages/shadowsocks/crypto/openssl.py", line 76, in __init__
    load_openssl()
  File "/usr/local/lib/python3.6/dist-packages/shadowsocks/crypto/openssl.py", line 52, in load_openssl
    libcrypto.EVP_CIPHER_CTX_cleanup.argtypes = (c_void_p,)
  File "/usr/lib/python3.6/ctypes/__init__.py", line 361, in __getattr__
    func = self.__getitem__(name)
  File "/usr/lib/python3.6/ctypes/__init__.py", line 366, in __getitem__
    func = self._FuncPtr((name_or_ordinal, self))
AttributeError: /usr/lib/x86_64-linux-gnu/libcrypto.so.1.1: undefined symbol: EVP_CIPHER_CTX_cleanup
jack@DESKTOP-9TGTFK1:~/ss$
```

这是因为在openssl 1.1.0中废弃了 EVP_CIPHER_CTX_cleanup() 函数而引入了 EVE_CIPHER_CTX_reset() 函数，具体可以查看[官方文档](https://www.openssl.org/docs/man1.1.0/man3/EVP_CIPHER_CTX_reset.html), 修复如下

```bash
# 在错误日志中找到 openssl.py 文件路径, 通过 vim 修改
sudo vim /usr/local/lib/python3.6/dist-packages/shadowsocks/crypto/openssl.py
# 替换关键自
:%s/cleanup/reset/
# 保存推出
:x
```

再运行 sslocal，成功

- MacOS 升级到 10.15 Catalina 之后就跑不起来了

据说是应为升级之后，一些包比如 openssl, dyid 什么的不兼容了导致的，重新安装一下就行了

```bash
brew update && brew upgrade

brew uninstall --ignore-dependencies openssl; brew install https://github.com/tebelorg/Tump/releases/download/v1.0.0/openssl.rb

# 如果 pip install 不好使了，可以试试重装一下
brew reinstall python
```

然后在 `.zshrc` 里面添加配置 `export DYLD_LIBRARY_PATH=/usr/local/opt/openssl/lib:$DYLD_LIBRARY_PATH`

### 终端 Git 下载加速

```config
# config your `~/.gitconfig` file
[http]
proxy = socks5://127.0.0.1:1080
sslVerify = false

[https]
proxy = socks5://127.0.0.1:1080
  
# or you can config it by typing terminal
git config --global http.proxy socks5://127.0.0.1:1080
git config --global https.proxy socks5://127.0.0.1:1080
git config --global http.sslVerify false

# turn off proxy, 开启后 git commit 会受影响
git config --global --unset http.proxy
git config --global --unset https.proxy
```

慢的话肯定是vps不给力，之前用 Vultr 的时候也是龟速，用了 google cloud, 芜湖，起飞！！！

## MacOS 安装 SS 客户端

Git 上有一个客户端，用了下还挺香的 [ShadowsocksX-NG](https://github.com/shadowsocks/ShadowsocksX-NG)。README 上有下载地址，直接下载后，解压将安装文件拖至 Application 文件夹下就行了。

配置注意点：

1. Servers -> Server Preference 添加自己的 SS 节点
2. Preferences 里面可以看到 proxy 设置，需要注意的是它为 Socks5 和 HTTP 设置了不同端口，Sock5 是 1086，HTTP 是 1087
3. 安装了这个应用之后貌似就不需要单独配置终端 proxy 了， 可以通用，或者使用 global mode