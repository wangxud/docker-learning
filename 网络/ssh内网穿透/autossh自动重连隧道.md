### autossh自动重连隧道

* 自动重连的前提是sshkey免密登陆，autossh才可以掉线自动重连  

```
# ssh-copy-id 账号@IP地址 -p 端口
ssh-copy-id root@266.266.266.266 -p 22
```

* autossh重连命令
```
# autossh -h 查看参数
# autossh -M 0 -o serverAliveInterval=30 -o ServerAliveCountMax=10 -gfnNTR 远程机器的隧道端口:127.0.0.1:22 账号@IP地址 -p 端口
autossh -M 0 -o serverAliveInterval=30 -o ServerAliveCountMax=10 -gfnNTR 30022:127.0.0.1:22 root@266.266.266.266 -p 22
```

#### 制作autossh.sh脚本，以后手动运行脚本，避免手动敲命令，容易出错
```
tee autossh.sh <<-'EOF'
#!/bin/bash

autossh -M 0 -o serverAliveInterval=30 -o ServerAliveCountMax=10 -gfnNTR 远程机器的隧道端口:127.0.0.1:22 账号@IP地址 -p 端口
EOF
chmod a+x autossh.sh
```