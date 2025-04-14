# Star实验室服务器使用简明指南

## 目录

[toc]

## 公网访问

目前服务器公网访问的策略是使用frp协议开放12号服务器的公网IP，利用12号服务器作为中转，再在12号服务器上通过ssh连接到内网的其他服务器。

访问流程：

首先在获取本地的ssh公钥。对于Windows系统，ssh公钥的位置在`C:\Users\用户名\.ssh\id_rsa.pub`，对于Linux系统，ssh公钥的路径在`~/.ssh/id_rsa.pub`。

若没有公钥，可以使用`ssh-keygen`命令生成。

联系林昂骏添加密钥。

添加密钥后可以直接使用ssh连接到12号服务器。

```bash
# 由于本仓库开源，因此不便公开公网ip和端口，如果需要访问请联系林昂骏
# ------ 本机环境 ------
ssh 12号服务器ip、端口和用户名
# ------ 12号服务器 ------
ssh 内网服务器ip、端口和用户名
# ------ 进入内网服务器 ------
```

或者，在进行中转机的ssh连接时直接使用`-t`参数进行ssh连接：

```bash

ssh -t 12号服务器ip、端口和用户名 -t 'ssh 内网服务器ip、端口和用户名'
```

此方法由于进行了一次ssh中转，因此VSCode等IDE的远程开发功能可能无法正常使用。解决方法详见[下文](#进行ssh中转配置服务)。

### 设置别名

在中转服务器的`.ssh/config`文件中可以进行配置，简化ssh的连接命令。

如，对于以下连接命令：

```bash
ssh -p 4027 -X adrian@10.254.29.57
```

可以在`~/.ssh/config`中添加以下内容：

```config
Host adrian
    HostName 10.254.29.57
    Port 4027
    User adrian
    ForwardX11 yes
```

进行以上配置后，可以直接使用`ssh adrian`命令连接到内网服务器的adrian账户。

在Windows或Mac系统中也可以进行类似配置，这里不多赘述。

### 进行ssh中转配置服务

以上流程可以进一步进行简化，使得只需要一次ssh连接就可以完成中转。

修改**本地**的ssh配置文件。Windows用户的配置文件位于`C:\Users\用户名\.ssh\config`，Linux和Mac用户的配置文件位于`~/.ssh/config`。

在配置文件中添加以下内容：

```config

# 跳板中转服务器的配置
Host jump
    HostName frp服务公网ip
    User roamer
    Port 端口
    IdentityFile ~/.ssh/id_rsa      # Windows用户需要改成Windows下对应路径

Host internal
    HostName 内网服务器ip
    User 用户名
    Port 端口
    ProxyJump jump

```

在以上配置中，`jump`是中转服务器的别名，`internal`是内网服务器的别名。

进行以上配置后，使用`ssh internal`即可连接到内网服务器。使用本方法后，由于只进行了一次ssh连接，因此可以直接使用VSCode等IDE的远程开发功能。

### 注意事项

> [!WARNING]
> 注意：请勿在目标服务器上使用中转服务器的公钥，以防止中转服务器被攻击后导致内网其他服务器也被攻击。
> 账户密码不要相同，尽可能设置复杂密码以应对暴力破解。

### 终端持久化

若直接使用ssh进行连接，在一段时间不操作后，ssh连接会自动断开。可以使用`tmux`进行终端持久化。

在ssh连接后，输入`tmux`命令即可进入tmux环境。

常用的tmux命令：

- `Ctrl + b`，然后按`d`：退出tmux会话
- `Ctrl + b`，然后按`c`：创建新的tmux窗口
- `Ctrl + b`，然后按数字：切换到指定的tmux窗口
- `Ctrl + b`，然后按`?`：查看tmux帮助文档
- `Ctrl + b`，然后按`:`：输入命令
- `Ctrl + b`，然后按`%`：水平分割窗口
- `Ctrl + b`，然后按`s`：查找会话
- `tmux new-session`：创建新的tmux会话
- `tmux attach-session -t session_name`：连接到指定的tmux会话

在使用tmux后，即使关闭了ssh连接，tmux会话内的程序仍然会继续运行。

### 目前问题

目前由于进行了一次ssh中转，因此VSCode等IDE的远程开发功能可能无法正常使用。相关问题目前正在解决。
