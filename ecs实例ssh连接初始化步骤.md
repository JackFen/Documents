# 阿里云ecs创建实例后ssh的设置

## 创建时（这里以debian 12为例）
- 在创建实例时，管理设置中的登录凭证选择创建后设置（ps:选择其余两个均会出现ssh工具连接不上的问题）
## 创建后
- 创建完实例后，点击全部操作中的重置登录密码，此设置将重置root的登录密码
## SSH工具连接（这里以final shell为例）
- 通过ssh工具登录实例，登录用户名选择root。登录方式选择密码
- 验证密码可以成功登录后，去阿里云控制台生成秘钥对并绑定该实例
- 用秘钥方式登录该实例(用户名仍为root)。
- 编辑ssh配置文件，新增Port 22和Port 自定义端口号，保存并退出
```bash
vim /etc/ssh/sshd_config
```
- 重启ssh服务
```bash
systemctl restart ssh
```
- ### 配置防火墙
  - 安装防火墙ufw
  ```bash
  apt update
  apt install ufw
  ```
  - 查看防火墙状态，此时应该是inactive状态
  ```bash
  ufw status verbose
  ```
  - 允许ssh连接
  ```bash
  ufw allow OpenSSH
  ufw allow 自定义端口号/tcp
  ```
  - 启动ufw
  ```bash
  ufw enable
  ```
  - 查看ufw的规则，检查无误后继续下一步
  ```bash
  ufw status numbered
  ```
- 在ssh工具中使用自定义的端口号连接实例
- 连接成功后，修改ssh配置文件，将Port 22注释掉，保存并退出。
```bash
vim /etc/ssh/sshd_config
```
- 重启ssh服务
```bash
systemctl restart ssh
```
- 使用22端口连接实例，若连接失败则代表ssh默认端口已被禁止。然后在阿里云控制台的安全组中将22端口设为禁止访问。
- ### 创建非root用户
  - 添加jack用户
  ```bash
  adduser jack
  ```
  - 查看jack是否成功添加
  ```bash
  id jack
  ```
  - 设置权限
  ```bash
  chown jack:jack /home/jack
  chmod 755 /home/jack
  ```
  - 设置jack用秘钥对登录，这里简化流程，将其和root用户使用同一密钥对
  - 创建jack的.ssh目录并设置权限
  ```bash
  mkdir -p /home/jack/.ssh
  chmod 700 /home/jack/.ssh
  chown jack:jack /home/jack/.ssh
  ```
 - 复制公钥并设置权限
 ```bash
 cp /root/.ssh/authorized_keys /home/jack/.ssh/authorized_keys
 chmod 600 /home/jack/.ssh/authorized_keys
 chown jack:jack /home/jack/.ssh/authorized_keys
 ```
 - 设置jack用户通过只能通过秘钥登录
 ```bash
 vim /etc/ssh/sshd_config
 ```
 - 在sshd_config设置以下内容
 ```bash
 PubkeyAuthentication yes
 PasswordAuthentication no  # 禁用密码登录
 PermitRootLogin no         # 禁用 root 用户登录（可选）
 ```
 - 保存后重启ssh服务
 ```bash
 systemctl restart ssh
 ```
 - 用ssh工具测试jack用户是否可以用秘钥登录
 - 若登录成功，则万事大吉