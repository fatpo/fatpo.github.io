由于github的DNS被污染问题，导致国内的机器访问github速度特别慢，但是我们可以通过`修改Hosts的方式来绕过DNS解析`，具体步骤
1. 打开https://www.ipaddress.com/ ，分别查询github.com、assets-cdn.github.com、github.global.ssl.fastly.net 三个域名的IP，
2. 然后将其添加到hosts文件中。等生效后，再访问github，就能看到github的访问速度有明显的提升

vi /etc/hosts

```text
140.82.114.4 github.com
185.199.108.153 github.com
185.199.109.153 github.com
185.199.110.153 github.com
185.199.111.153 github.com
199.232.69.194 github.com

```

