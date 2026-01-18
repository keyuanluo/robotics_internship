# 使用SSH协议设置电脑远程操控 123456
## 环境配置
### 服务端：
**Ubuntu22.04**
### 客户端：
**Ubuntu22.04**
## 操作流程
### 1. 安装 XRDP 及依赖
**服务端：** <br>
终端执行指令
```bash
# 更新软件源
sudo apt update
# 安装xrdp远程桌面服务
sudo apt install xrdp -y
```
### 2. 启动SSH服务
根据是否需要设置开机自启在以下操作二选一
#### 一、设置开机自启
**服务端：** <br>
终端执行指令
```bash
# 启动 SSH 服务，并设置开机自启
sudo systemctl enable --now sshd
# 验证服务是否运行成功
sudo systemctl status sshd
```
输出中出现 `active (running)` 则表示服务正常。
#### 二、不设置开机自启
**服务端：** <br>
终端执行指令
```bash
# 手动启动 SSH 服务
sudo systemctl start sshd
# 验证服务是否运行成功
sudo systemctl status sshd
```
输出中出现 `active (running)` 则表示服务正常。
### 3. 配置防火墙放行 SSH 端口
Ubuntu 22.04 默认启用 ufw 防火墙，需放行 SSH 默认的 22 端口：
#### 一、放行端口
**服务端：** <br>
终端执行指令
```bash
sudo ufw allow ssh
```
或指定端口
```bash
sudo ufw allow 22/tcp
```
#### 二、验证防火墙规则
**服务端：** <br>
终端执行指令
```bash
sudo ufw status
```
出现 `22/tcp ALLOW Anywhere` 则表示生效
### 4. 获取服务器内网 IP
获取内网IP有两种指令，以下操作二选一执行
#### 一、列出本机所有网络接口的详细参数
**服务端：** <br>
终端执行指令
```bash
ip addr show
```
查找 `eno2` 或 `wlo1` 对应的 `inet` 字段，如 `192.168.1.100`<br>
#### 二、快速查看 `eno2` 或 `wlo1` 的内网 IP<br>
**服务端：** <br>
终端执行指令
```bash
# 查看eno2的IPv4地址
ip addr show eno2 | grep 'inet ' | awk '{print $2}' | cut -d '/' -f1
```
该输出为有线内网 IP
```bash
# 查看wlo1的IPv4地址
ip addr show wlo1 | grep 'inet ' | awk '{print $2}' | cut -d '/' -f1
```
该输出为无线内网 IP
> *注：`eno2` 和 `wlo1` 是Ubuntu系统采用可预测网络接口命名规则后生成的网卡设备名，取代了传统的 `eth0/wlan0` 命名方式，命名规则由设备的物理位置或固件信息决定，稳定性更强（重启后不会随机变更名称）。*
### 5. 客户端连接
**客户端：** <br>
终端执行指令
```bash
ssh `服务端用户名` @ `服务器内网IP`
# 示例：ssh ubuntu@192.168.1.100
```
#### 首次连接
当你的主机首次连接该服务器，将会触发提示：
```plaintext
The authenticity of host '服务器内网IP' can't be established.
ED25519 key fingerprint is ......
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```
输入 `yes` 使这台服务器的密钥信息永久保存到本地的「可信主机列表」里
此时系统会提示 `Warning: Permanently added '服务器IP地址' (ED25519) to the list of known hosts.`
#### 输入密码
当你的主机连接服务器时，将会触发提示：
```plaintext
`服务端用户名` @ `服务器内网IP` 's password:
```
此时输入服务端用户的密码即可进行连接
### 6. 客户端关闭
**客户端：** <br>
终端执行指令
```bash
exit
```
此时你的主机将与服务器断开连接
### 7. 服务端关闭
**服务端：** <br>
终端执行指令
```bash
sudo systemctl stop sshd
```
