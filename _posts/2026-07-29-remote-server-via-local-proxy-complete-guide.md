---
layout: post
title: "让远程 Linux 服务器稳定复用本机 Clash：SSH 反向代理完整指南"
subtitle: "使用独立 SSH 反向隧道，为 VS Code、FinalShell 与命令行工具提供稳定代理"
date: 2026-07-29 10:00:00
author: "wyh"
header-style: text
catalog: true
tags:
  - 笔记
  - Linux
  - SSH
  - Clash
  - 远程开发
---

很多远程服务器无法直接访问 OpenAI、Anthropic、GitHub、npm 或 PyPI，而自己的 Windows 电脑已经可以通过 Clash 正常访问这些服务。一个实用的解决办法，是利用 SSH 反向端口转发，把远程服务器上的 HTTP/HTTPS 请求送回本机，再交给 Clash 处理。

这个方案本身并不复杂，但如果直接把反向转发写进日常使用的 SSH 配置，很容易遇到以下问题：

- VS Code 第一次可以连接，打开第二个远程窗口却连接失败。
- FinalShell、Xshell 等软件没有读取 Windows 的 `~/.ssh/config`，所以代理没有建立。
- 本机 Clash 或 SSH 断开后，远程终端中的 Git、pip、Codex 等全部超时。
- 在共享服务器上，其他用户可能借用这个代理端口。
- `curl` 可以使用代理，但 VS Code 扩展、后台任务或其他工具仍然无法联网。

本文会先给出一套稳定、清晰、适合长期使用的推荐方案，再解释它的工作原理、影响范围、安全边界和常见故障。

---

## 一、先给结论：最优方案是什么

推荐把连接分成两个完全独立的部分：

1. **代理隧道连接**：由本机 PowerShell 单独建立，只负责维持反向端口转发。
2. **日常工作连接**：VS Code、FinalShell、普通 SSH 等软件照常连接服务器，不负责建立代理隧道。

整体结构如下：

```text
Windows 本机

PowerShell: ssh -N my-server-proxy
        |
        | 只维护一条 SSH 反向代理隧道
        v
远程服务器 127.0.0.1:17897
        ^
        |
        +-- VS Code 远程终端
        +-- FinalShell 终端
        +-- 普通 SSH 终端
        +-- curl / Git / npm / pip / Codex / Claude Code
```

这样设计有三个直接好处：

- VS Code 可以建立多条连接，不会争抢同一个远程端口。
- FinalShell 是否读取 Windows 的 SSH 配置不再重要，因为隧道已经由 PowerShell 建好。
- 即使关闭某个 VS Code 或 FinalShell 窗口，代理隧道仍然存在。

> 核心原则：只允许一条专用 SSH 连接创建 `17897` 端口，其他连接只使用这个端口，不再重复创建它。

---

## 二、工作原理

本文使用以下示例参数：

| 项目 | 示例值 | 说明 |
| --- | --- | --- |
| 本机操作系统 | Windows | 运行 Clash 和 SSH 客户端 |
| 远程服务器 | Linux | VS Code、FinalShell 等连接的服务器 |
| 本机 Clash 端口 | `127.0.0.1:7897` | HTTP 或 Mixed 代理端口 |
| 远程代理入口 | `127.0.0.1:17897` | SSH 在服务器上创建的监听端口 |
| 普通连接别名 | `my-server` | VS Code、终端日常使用 |
| 隧道连接别名 | `my-server-proxy` | 只用来建立代理隧道 |

实际网络路径是：

```text
远程程序
  -> 读取 http_proxy / https_proxy
  -> 远程服务器 127.0.0.1:17897
  -> SSH 加密隧道
  -> Windows 本机 127.0.0.1:7897
  -> Clash 匹配规则
       -> PROXY：从代理节点访问
       -> DIRECT：从 Windows 本机网络直接访问
       -> REJECT：拒绝访问
  -> 目标网站或 API
```

这里使用的是 SSH `RemoteForward`，也就是命令行中的 `-R`。监听端口开在远程服务器上，但真正的目标服务 Clash 位于发起 SSH 连接的 Windows 本机，因此称为“反向端口转发”。

需要注意：这不是服务器全局 VPN。只有支持 HTTP 代理并读取代理环境变量的程序才会自动使用它。

### SSH 的两种端口转发：`-L` 与 `-R`

SSH 除了远程登录服务器，还可以建立端口转发，常用的有两个方向：

- **`-L` 本地端口转发**：格式为 `-L 本地端口:目标地址:目标端口`，把“服务器上的服务”映射到本地。例如：

  ```bash
  ssh -L 7861:localhost:7861 wyh@172.16.1.24
  ```

  这表示在本机监听 `7861` 端口，访问本地 `localhost:7861` 的流量会通过 SSH 加密隧道转发到服务器 `172.16.1.24` 的 `localhost:7861`。如果服务器上的 WebUI 运行在 `7861` 端口，本地浏览器直接访问 `http://localhost:7861` 就能打开它。SSH 连接断开后，这个转发也随之失效。

- **`-R` 远程端口转发**：格式为 `-R 服务器端口:目标地址:目标端口`，方向相反，把“本地的服务”映射到服务器。本文的 Clash 复用方案用的正是它。本机 Clash 运行在 `127.0.0.1:7897`，通过：

  ```bash
  ssh -R 17897:localhost:7897 wyh@172.16.1.24
  ```

  服务器上的 `localhost:17897` 就会经 SSH 隧道连接到本机 Clash。这条命令与后文 SSH 配置中的 `RemoteForward 127.0.0.1:17897 127.0.0.1:7897` 完全等价，只是一个写在命令行、一个写在配置文件里。

因此，服务器上的程序把代理配置为 `http://127.0.0.1:17897`（前提是本机 Clash 的 `7897` 是 HTTP/Mixed 代理端口）后，完整网络路径就是：

```text
服务器程序 -> 服务器:17897 -> SSH 隧道 -> 本机:7897 -> Clash -> Internet
```

如果 Clash 提供的是 SOCKS5 端口，则应使用 `socks5://127.0.0.1:17897` 的形式。

> 补充说明：VS Code Remote SSH 本质上也是 SSH 连接。如果没有额外配置端口转发，它不会自动把服务器上的端口映射到本地；当然，VS Code 本身也支持配置端口转发。“普通 SSH 连接”和“SSH 端口转发”并不互斥——端口转发只是 SSH 的一种附加功能。本文把转发拆给一条专用连接来做，正是为了让它与 VS Code 的日常连接互不干扰。

### 给初学者的通俗解释：可以把监听端口理解成一个“小应用”

为了方便理解，可以暂时把服务器上的 `127.0.0.1:17897` 想象成一个正在运行的“小应用”或“接待窗口”。它一直监听这个端口，等待服务器上的程序把网络请求交给它。

不过从技术上说，它并不是一个真正理解 HTTP 的代理应用，而是 SSH 在服务器上创建的 TCP 转发入口。它的工作很简单：收到数据后，不分析请求内容，直接通过已经建立的 SSH 隧道把数据送回 Windows 本机的 Clash `127.0.0.1:7897`。真正理解代理协议、识别目标网站并决定访问线路的是 Clash。

可以把各部分的职责理解为：

```text
远程服务器 127.0.0.1:17897 = 接待窗口，接收远程程序交来的请求
SSH 隧道                      = 加密运输通道，把请求送回本机
本机 Clash 127.0.0.1:7897    = 真正的代理和调度员，决定如何访问目标
目标网站                     = 最终收件人
```

假设远程服务器上的程序准备访问：

```text
http://example.com/test
```

目标网站使用普通 HTTP 时，默认端口通常是 `80`。配置代理后，远程程序不会先直接连接 `example.com:80`，而是先连接服务器自己的 `127.0.0.1:17897`，并在 HTTP 请求中告诉代理自己真正想访问的是 `example.com`：

```http
GET http://example.com/test HTTP/1.1
Host: example.com
```

接下来的完整流程是：

```text
1. 远程程序把请求交给服务器的 127.0.0.1:17897
2. 17897 将数据放进 SSH 隧道
3. SSH 把数据送到 Windows 本机的 127.0.0.1:7897
4. Clash 读取目标地址并匹配 DIRECT、PROXY 或 REJECT 规则
5. Clash 自己直连目标网站，或者让选中的代理节点访问目标网站
6. 网站响应沿着 Clash -> SSH 隧道 -> 17897 原路返回远程程序
```

如果访问的是：

```text
https://api.openai.com
```

目标网站通常使用 `443` 端口。远程程序仍然先连接 `127.0.0.1:17897`，然后通过代理发送类似下面的请求：

```http
CONNECT api.openai.com:443 HTTP/1.1
```

Clash 建立到目标网站或代理节点的连接后，远程程序再通过这条通道与目标网站进行 HTTPS/TLS 通信。目标网站的内容仍然受到 TLS 加密保护，服务器到本机之间的数据还会额外经过 SSH 加密。

因此，可以把整个过程概括成一句话：

> 远程程序先把“我要访问哪个网站”这件事交给服务器上的 `17897` 接待窗口，SSH 负责把请求运回本机，Clash 再根据规则代表它访问目标网站，最后把响应按原路送回去。

这种“小应用”比喻适合帮助初学者建立整体印象，但应记住：`17897` 本身只是负责转发字节的 SSH 监听入口，Clash 才是真正的代理应用。此外，只有主动使用代理配置的程序才会把请求交给这个入口，服务器上的所有网络请求并不会自动经过它。

---

## 三、使用前需要确认的条件

开始前应满足以下条件：

- Windows 本机已经运行 Clash。
- Clash 已开启 HTTP 或 Mixed 代理端口。
- 本机能够通过 Clash 访问目标网站。
- Windows 自带的 OpenSSH 可以正常登录服务器。
- 服务器 SSH 服务允许 TCP 端口转发。
- 使用期间本机不能关机或休眠，Clash 和隧道连接需要保持运行。

这个方案不需要：

- 在远程服务器安装 Clash。
- 对公网开放服务器的 `17897` 端口。
- 启用 SSH 的 `GatewayPorts`。
- 关闭 HTTPS/TLS 证书校验。
- 把 Codex、Claude 或其他服务的登录凭据复制到本机代理中。

### 先判断服务器是否适合使用这个方案

这个方案更适合个人独占、自己管理或完全可信的服务器。

如果服务器上还有其他不可信用户，需要格外谨慎。虽然 `127.0.0.1:17897` 不会直接暴露到公网，但 Linux 的回环 TCP 端口通常不是“当前用户专用”的。服务器上的其他本地用户也可能连接这个端口，借用你的 Clash、代理节点和本机网络。

另外，能够使用这个代理的远程程序可能尝试访问 Windows 本机或局域网地址。是否能够访问取决于 Clash 的规则和安全设置。因此，不要在不可信或已经失陷的远程服务器上建立这条隧道。

---

## 四、第一步：确认本机 Clash 端口

在 Windows PowerShell 中执行：

```powershell
Test-NetConnection 127.0.0.1 -Port 7897
```

如果结果包含：

```text
TcpTestSucceeded : True
```

说明有程序正在监听 `7897`。接着验证它确实是可用的 HTTP/Mixed 代理：

```powershell
curl.exe --max-time 15 -x http://127.0.0.1:7897 https://api.ipify.org
```

如果返回一个公网 IP，说明本机代理可以正常工作。这个 IP 可能是代理节点出口，也可能是 Windows 本机出口，具体由 Clash 规则决定。

如果 Clash 的实际端口不是 `7897`，后续所有配置都应使用实际端口。

---

## 五、第二步：配置两个独立的 SSH 别名

打开 Windows 当前用户的 SSH 配置文件：

```powershell
notepad $env:USERPROFILE\.ssh\config
```

加入以下配置，并替换服务器地址、用户名、端口和私钥路径：

```sshconfig
# 普通连接：VS Code、PowerShell 和其他 OpenSSH 客户端使用
Host my-server
    HostName 服务器IP或域名
    User 用户名
    Port 22
    IdentityFile ~/.ssh/id_ed25519
    ServerAliveInterval 30
    ServerAliveCountMax 3

# 专用代理连接：只建立隧道，不用于 VS Code 日常连接
Host my-server-proxy
    HostName 服务器IP或域名
    User 用户名
    Port 22
    IdentityFile ~/.ssh/id_ed25519
    RemoteForward 127.0.0.1:17897 127.0.0.1:7897
    ExitOnForwardFailure yes
    ServerAliveInterval 30
    ServerAliveCountMax 3
```

如果服务器使用密码登录，可以删除 `IdentityFile`。如果 SSH 端口不是 `22`，应修改两个配置块中的 `Port`。

配置完成后，可以检查 OpenSSH 实际读取到的内容：

```powershell
ssh -G my-server
```

```powershell
ssh -G my-server-proxy
```

在输出中确认 `hostname`、`user`、`port` 和 `remoteforward` 是否正确。

### 为什么不能把 `RemoteForward` 放进 `my-server`

VS Code Remote-SSH 可能为了终端、扩展、文件系统或新的远程窗口建立多条 SSH 连接。

第一次连接会成功监听服务器的 `127.0.0.1:17897`。第二条连接再次尝试监听同一个地址和端口时，服务器会返回端口已被占用。因为配置中还有：

```sshconfig
ExitOnForwardFailure yes
```

第二条 SSH 连接就会直接退出，于是出现“VS Code 第一次能连接，第二次连接失败”。

仅仅删除 `ExitOnForwardFailure yes` 并不是理想解决方案。这样虽然第二条连接可能不再退出，但转发失败会被隐藏，而且代理仍然依赖第一条连接。正确做法是让专用连接独占这个转发。

---

## 六、第三步：单独启动代理隧道

如果之前已经通过 VS Code 或普通 SSH 建立过相同的 `RemoteForward`，应先断开这些旧连接，并从普通连接配置中删除 `RemoteForward`。否则旧连接仍可能占用 `17897`。

然后在 Windows PowerShell 中运行：

```powershell
ssh -N my-server-proxy
```

参数 `-N` 表示不打开远程 Shell，只建立转发。成功后窗口通常没有输出，也不会返回命令提示符，这是正常现象。

这个 PowerShell 窗口需要保持运行。结束隧道可以按 `Ctrl+C` 或关闭窗口。

在远程服务器上检查监听状态：

```bash
ss -lnt | grep ':17897'
```

正常情况下可以看到类似结果：

```text
LISTEN 0 128 127.0.0.1:17897 0.0.0.0:*
```

然后执行完整链路测试：

```bash
curl --max-time 15 --proxy http://127.0.0.1:17897 https://api.ipify.org
```

如果能够返回公网 IP，说明以下链路全部正常：

```text
远程 curl -> 服务器 17897 -> SSH -> 本机 7897 -> Clash -> 外网
```

### 保活配置能做什么，不能做什么

`ServerAliveInterval` 和 `ServerAliveCountMax` 可以帮助 SSH 更快发现已经失效的网络连接，但不会自动重连。

如果本机网络切换、休眠、重启或 SSH 进程退出，需要重新执行：

```powershell
ssh -N my-server-proxy
```

长期无人值守使用时，可以再通过 Windows 任务计划程序或专门的隧道守护工具启动这条连接。无论使用哪种方式，都应保持“只有一个隧道管理者”的原则。

---

## 七、VS Code 应该怎样连接

在 VS Code 中：

1. 按 `Ctrl+Shift+P` 打开命令面板。
2. 选择 `Remote-SSH: Connect to Host...`。
3. 选择 `my-server`，不要选择 `my-server-proxy`。

因为 `User`、`HostName`、`Port` 和私钥都已写入 SSH 配置，VS Code 可以直接使用别名，不需要输入 `用户名@服务器IP`。

在本机 PowerShell 中，普通登录也可以直接使用：

```powershell
ssh my-server
```

只要专用 PowerShell 中的 `ssh -N my-server-proxy` 仍在运行，VS Code 可以打开多个远程窗口，它们都会复用服务器上已经存在的 `127.0.0.1:17897`，不会再次争抢端口。

如果 VS Code 中没有显示 `my-server`，检查设置 `Remote.SSH: Config File` 是否指向：

```text
C:\Users\你的用户名\.ssh\config
```

---

## 八、FinalShell、Xshell 等软件为什么表现不同

FinalShell、Xshell、SecureCRT 等软件通常使用自己的 SSH 实现、服务器列表和密钥配置，不一定读取 Windows OpenSSH 的：

```text
C:\Users\你的用户名\.ssh\config
```

因此，即使该文件中配置了 `RemoteForward`，这些软件建立连接时也可能完全不知道这项配置。此时它们只能登录服务器，不能替你创建代理隧道。

这也解释了一个常见现象：

> 直接使用 FinalShell 登录时无法通过代理访问外网；先在本机 PowerShell 建立一条 SSH 连接，再打开 FinalShell，就可以访问外网。

原因不是 FinalShell 突然读取了 `.ssh/config`，而是 PowerShell 中的 OpenSSH 已经在远程服务器上创建了 `127.0.0.1:17897`。这个监听端口属于服务器网络环境，之后进入服务器的 FinalShell 会话也能使用它。

正确的日常操作是：

1. 本机启动 Clash。
2. PowerShell 运行 `ssh -N my-server-proxy`。
3. FinalShell 使用自己的服务器配置正常登录。
4. 在 FinalShell 的远程终端中执行 `proxy_on`。

有些 SSH 软件也提供“反向隧道”或“远程端口转发”设置，但不建议同时在多个软件中配置相同的 `17897`。否则哪个软件先连接，哪个软件就占用端口，其他连接仍会发生冲突。

---

## 九、第四步：在远程服务器配置代理开关

SSH 隧道只提供代理入口，不会强制所有程序使用它。还需要设置代理环境变量。

先确认当前 Shell：

```bash
echo "$SHELL"
```

- Bash 通常修改 `~/.bashrc`。
- Zsh 通常修改 `~/.zshrc`。

以下以 Bash 为例。先备份：

```bash
cp ~/.bashrc ~/.bashrc.backup.$(date +%Y%m%d-%H%M%S)
```

然后在 `~/.bashrc` 末尾加入：

```bash
# HTTP proxy provided by the dedicated SSH reverse tunnel.
proxy_on() {
    local proxy_url="http://127.0.0.1:17897"

    # Test the complete path before changing the current shell.
    if ! curl --silent --show-error --max-time 10 \
        --proxy "$proxy_url" https://api.ipify.org >/dev/null; then
        echo "代理未开启：请检查本机 Clash 和 ssh -N my-server-proxy。" >&2
        return 1
    fi

    export http_proxy="$proxy_url"
    export https_proxy="$proxy_url"
    export all_proxy="$proxy_url"
    export HTTP_PROXY="$http_proxy"
    export HTTPS_PROXY="$https_proxy"
    export ALL_PROXY="$all_proxy"

    # Add internal domains or addresses here when necessary.
    export no_proxy="127.0.0.1,localhost,::1"
    export NO_PROXY="$no_proxy"

    echo "代理已开启：$proxy_url"
}

proxy_off() {
    unset http_proxy https_proxy all_proxy
    unset HTTP_PROXY HTTPS_PROXY ALL_PROXY
    unset no_proxy NO_PROXY
    echo "代理已关闭。"
}

proxy_status() {
    echo "代理环境变量："
    env | grep -iE '^(http|https|all|no)_proxy=' || true
    echo
    echo "远程监听端口："
    ss -lnt 2>/dev/null | grep ':17897' || echo "未发现 17897 监听端口"
}
```

检查 Bash 语法并加载：

```bash
bash -n ~/.bashrc
```

```bash
source ~/.bashrc
```

需要代理时执行：

```bash
proxy_on
```

不需要时执行：

```bash
proxy_off
```

查看当前状态：

```bash
proxy_status
```

### 为什么推荐手动执行 `proxy_on`

不建议在 `.bashrc` 最后一行无条件写入：

```bash
proxy_on
```

如果本机 Clash 没启动、电脑休眠或 SSH 隧道已经断开，每个新终端仍会把请求发送到失效代理。Git、pip、npm、Codex 等程序可能长时间等待，看起来像服务器网络完全坏了。

手动开启多一步操作，但故障边界最清晰。本文提供的 `proxy_on` 还会先测试完整链路，测试失败就不会污染当前 Shell 的代理环境。

如果目标服务器只在本机在线时使用，也可以选择自动执行，但需要接受隧道不可用时终端初始化变慢或联网失败的影响。

### 登录 Shell 没有读取 `.bashrc` 怎么办

部分客户端启动的是 Bash 登录 Shell，它可能读取 `~/.bash_profile`，而不是直接读取 `~/.bashrc`。

可以检查 `~/.bash_profile` 是否包含：

```bash
if [ -f ~/.bashrc ]; then
    . ~/.bashrc
fi
```

如果没有，可以在确认原文件逻辑后加入。也可以临时手动执行：

```bash
source ~/.bashrc
```

然后再执行：

```bash
proxy_on
```

---

## 十、验证代理是否真正生效

先查看环境变量：

```bash
env | grep -i proxy
```

再测试出口 IP：

```bash
curl --max-time 15 https://api.ipify.org
```

测试常见 API：

```bash
curl -I --max-time 15 https://api.openai.com
```

```bash
curl -I --max-time 15 https://api.anthropic.com
```

返回 `200` 表示成功。对于 API 域名，`401`、`403`、`404` 或 `405` 也经常说明网络已经连通，只是没有凭据、请求方法不正确、账号受限或出口地区不符合服务要求。

真正的网络故障通常表现为：

- `Connection refused`：端口没有监听或 Clash 没有接收连接。
- `Connection timed out`：隧道、Clash、节点或目标网络超时。
- `Could not resolve host`：直连 DNS 或工具自身的解析路径有问题。
- `Proxy CONNECT aborted`：HTTP 代理链路建立失败。

---

## 十一、这套操作会带来哪些影响

### 1. 远程流量会占用本机带宽

请求和响应都会经过远程服务器与 Windows 本机之间的 SSH 连接。远程下载大文件、安装大量依赖或进行模型通信，会占用本机带宽和代理节点流量。

### 2. 最终出口由 Clash 决定

Clash 匹配 `PROXY` 时，目标网站看到的是代理节点出口 IP；匹配 `DIRECT` 时，目标网站看到的是 Windows 本机公网 IP，而不是远程服务器 IP。

出口地区变化可能触发某些网站或账号的安全检查、地区限制或重新登录。

### 3. 延迟通常会增加

请求会先从服务器返回本机，再由本机访问目标网站。服务器、本机和代理节点之间任意一段网络较慢，都会影响整体速度。

### 4. 它不会代理所有协议

通常可以工作的程序包括：

- `curl`、`wget` 等 HTTP 工具。
- Git 的 HTTPS 远程地址。
- npm、pip 等支持代理变量的包管理器。
- 支持这些变量的 Codex、Claude Code 和其他命令行程序。

通常不会自动工作的流量包括：

- `ping` 使用的 ICMP。
- Git 的 `git@github.com:...` SSH 协议。
- 原始 TCP/UDP 程序。
- 不读取代理环境变量的软件。
- 没有继承当前 Shell 环境的 systemd、cron 和后台服务。

### 5. 本机成为远程请求的出口

远程程序实际上获得了使用本机 Clash 的能力。可信远程程序可以借此访问外网；恶意程序也可能消耗代理流量、访问不希望访问的地址，甚至尝试访问本机局域网资源。

### 6. HTTPS 内容仍然由 TLS 保护

`HTTPS_PROXY=http://127.0.0.1:17897` 是正常写法，表示通过 HTTP 代理的 `CONNECT` 方法传输 HTTPS。目标网站的 TLS 加密仍然存在，服务器到本机之间另外还有 SSH 加密。

不应为了使用代理关闭证书校验，例如：

```text
NODE_TLS_REJECT_UNAUTHORIZED=0
http.proxyStrictSSL=false
```

这些设置会降低安全性，也不能修复 SSH 隧道或 Clash 端口错误。

---

## 十二、常见问题、原因和解决方案

### 问题 1：VS Code 第一次能连接，第二次连接失败

**原因**：`RemoteForward` 被写进 VS Code 使用的 Host 配置。第一条连接已经占用远端 `17897`，第二条连接再次绑定失败；`ExitOnForwardFailure yes` 随即中止第二条 SSH 连接。

**解决方案**：使用本文的两个别名。`my-server-proxy` 独占隧道，VS Code 永远连接不含 `RemoteForward` 的 `my-server`。

### 问题 2：FinalShell 登录成功，但不能通过代理访问外网

**原因**：FinalShell 通常不读取 Windows OpenSSH 的 `.ssh/config`，因此没有创建反向转发。

**解决方案**：先在 PowerShell 运行：

```powershell
ssh -N my-server-proxy
```

再使用 FinalShell 登录，并在远端执行 `source ~/.bashrc` 和 `proxy_on`。

### 问题 3：先打开 PowerShell SSH，再打开 FinalShell 就能联网

**原因**：这是正常现象。PowerShell 中的 OpenSSH 已经在服务器上创建了 `127.0.0.1:17897`，FinalShell 只是复用这个现成入口。

**解决方案**：把 PowerShell 中的连接明确改为专用的 `ssh -N my-server-proxy`，不要依赖某个普通 SSH 或 VS Code 窗口偶然维持隧道。

### 问题 4：提示 `remote port forwarding failed for listen port 17897`

**可能原因**：

- 旧的 SSH 或 VS Code 连接仍占用端口。
- 另一条专用隧道已经在运行。
- 服务器 SSH 策略禁止端口转发。
- 服务器上其他程序正在使用这个端口。

**排查方法**：

```bash
ss -lnt | grep ':17897'
```

关闭旧连接后再启动专用隧道。如果确认没有进程占用但仍失败，需要由服务器管理员检查 `AllowTcpForwarding` 或相关 SSH 策略。

不要同时在 PowerShell、VS Code 和 FinalShell 中创建相同的转发。

### 问题 5：服务器能看到 `17897` 监听，但代理请求仍然失败

**原因**：远程监听存在，只说明 SSH 创建了入口，不代表本机 Clash 一直可用。Clash 退出、端口改变或节点失效后，SSH 监听可能仍然存在。

**解决方案**：本机依次检查：

```powershell
Test-NetConnection 127.0.0.1 -Port 7897
```

```powershell
curl.exe --max-time 15 -x http://127.0.0.1:7897 https://api.ipify.org
```

如果本机测试失败，先修复 Clash；如果本机成功，再重新启动 `ssh -N my-server-proxy` 并查看 SSH 错误输出。

### 问题 6：手动指定 `--proxy` 成功，直接运行 curl 失败

**原因**：隧道正常，但当前 Shell 没有代理环境变量。

**解决方案**：

```bash
source ~/.bashrc
proxy_on
env | grep -i proxy
```

如果使用 Zsh，应修改并加载 `~/.zshrc`。

### 问题 7：重新打开终端后找不到 `proxy_on`

**原因**：客户端启动的 Shell 没有读取你修改的配置文件，或者当前使用的并不是 Bash。

**解决方案**：检查：

```bash
echo "$SHELL"
ps -p $$ -o args=
```

Bash 检查 `~/.bashrc` 和 `~/.bash_profile`，Zsh 检查 `~/.zshrc`。必要时手动 `source` 对应文件。

### 问题 8：curl 成功，但 Git、npm、pip 或某个工具失败

**可能原因**：

- 工具没有读取通用代理变量。
- 工具在执行 `proxy_on` 之前已经启动。
- 工具有自己的代理配置并覆盖环境变量。
- Git 使用的是 SSH 地址，而不是 HTTPS 地址。
- 实际问题是认证、账号权限、地区策略或 API Key，而不是网络。

**解决方案**：先重启该工具，再检查它的官方代理配置和自身设置。分别测试：

```bash
env | grep -i proxy
curl -I https://目标域名
```

HTTP `401` 或 `403` 通常代表已经连到服务端，应继续检查身份认证和服务策略。

### 问题 9：`sudo` 后程序不再使用代理

**原因**：`sudo` 默认可能清理 `http_proxy`、`https_proxy` 等环境变量。

**解决方案**：优先避免让需要联网的用户级工具通过 `sudo` 运行。确有需要时，由管理员针对必要命令配置安全的环境变量保留规则。不要为了方便无条件保留所有用户环境变量。

### 问题 10：cron、systemd 或后台任务不使用代理

**原因**：后台服务通常不会读取用户的 `.bashrc`，也不会继承当前交互终端的环境。

**解决方案**：在对应的任务或服务配置中显式设置代理环境变量，并确保隧道是长期稳定运行的。修改 systemd 等系统服务前应评估权限和影响。

### 问题 11：VS Code 终端能联网，但某个远程扩展不能联网

**原因**：远程扩展进程可能在执行 `proxy_on` 之前已经启动，或者根本不读取交互式 Shell 的 `.bashrc`。

**解决方案**：查看该扩展自身是否提供代理设置；必要时在确认隧道正常后重启远程扩展宿主或重新连接 VS Code。不要仅凭终端中的环境变量判断所有扩展都已继承代理。

### 问题 12：访问服务器内网服务时也被送进了代理

**原因**：目标地址没有加入 `no_proxy`。

**解决方案**：根据实际环境，把内部域名或地址加入 `no_proxy`，例如：

```bash
export no_proxy="127.0.0.1,localhost,::1,.example.internal,10.0.0.10"
export NO_PROXY="$no_proxy"
```

不同工具对通配符和 CIDR 的支持不完全一致，应针对实际工具测试，不要假设所有程序都支持 `10.0.0.0/8` 这样的写法。

### 问题 13：在 Docker 容器中访问 `127.0.0.1:17897` 失败

**原因**：容器中的 `127.0.0.1` 指向容器自己，不是宿主 Linux 服务器。SSH 创建的监听端口位于宿主网络命名空间。

**解决方案**：在容器内单独建立受控转发，或由管理员通过明确的宿主地址和防火墙规则提供入口。不要为了省事直接把远程转发改成 `0.0.0.0:17897`，这可能把代理暴露给局域网或公网。

### 问题 14：本机关闭“系统代理”或 TUN 后，远程代理是否还能用

**答案**：通常可以。本文直接连接 Clash 的 HTTP/Mixed 端口 `7897`，不依赖 Windows 的“系统代理”开关，也不要求 TUN。只要 Clash 内核仍运行且 `7897` 正常监听即可。

### 问题 15：关闭 VS Code 后代理也断了

**原因**：旧方案让 VS Code 的 SSH 连接负责维持 `RemoteForward`。

**解决方案**：改用独立的 `ssh -N my-server-proxy`。之后关闭 VS Code 或 FinalShell 不会影响隧道。

### 问题 16：本机休眠或切换网络后代理失效

**原因**：原 SSH TCP 连接已经失效，保活只能检测断线，不能自动重新建立连接。

**解决方案**：重新运行：

```powershell
ssh -N my-server-proxy
```

如果经常发生，可以使用任务计划程序或隧道守护工具自动重启，但仍要避免同时启动多个隧道实例。

---

## 十三、安全建议

### 1. 只监听远程回环地址

应始终保持：

```sshconfig
RemoteForward 127.0.0.1:17897 127.0.0.1:7897
```

不要轻易改成：

```text
0.0.0.0:17897
```

`0.0.0.0` 可能让其他网络设备访问你的代理，造成代理滥用、流量损失和安全风险。

### 2. 回环地址不等于当前用户专用

绑定 `127.0.0.1` 只能阻止外部机器直接访问，不能自动阻止服务器上的其他本地用户。如果是共享服务器，应避免使用这套无认证 TCP 入口，或者由管理员提供用户隔离、防火墙限制或带认证的代理方案。

仅仅换一个随机高端口只能减少偶然发现，不能构成可靠的访问控制。

### 3. 只在可信服务器上开放本机代理能力

远程账户一旦被入侵，攻击者可能借助这条隧道消耗代理节点流量，或者尝试访问 Clash 允许连接的本机和局域网目标。

### 4. 不要关闭 TLS 校验

代理无法连接时，应排查端口、SSH、Clash 和规则，而不是关闭证书验证。

### 5. 留意 Clash 日志和流量

Clash 通常能显示目标域名、匹配规则、最终策略和使用节点。出现异常流量时，应立即关闭隧道、执行 `proxy_off` 并检查远程账户安全。

---

## 十四、推荐的日常使用流程

每次使用时按以下顺序操作：

### Windows 本机

1. 启动 Clash。
2. 确认 `7897` 端口正常。
3. 打开一个专用 PowerShell，运行：

```powershell
ssh -N my-server-proxy
```

### 连接服务器

- VS Code 选择 `my-server`。
- PowerShell 使用 `ssh my-server`。
- FinalShell 使用软件中已经保存的服务器连接。

这些连接都不负责创建代理，只使用专用隧道提供的远程端口。

### 远程终端

```bash
proxy_on
```

```bash
curl https://api.ipify.org
```

然后再启动需要代理的程序：

```bash
codex
```

```bash
claude
```

已经运行的程序不会自动获得后来设置的环境变量，需要退出并重新启动。

### 使用结束

远程当前终端可以执行：

```bash
proxy_off
```

然后在专用 PowerShell 中按 `Ctrl+C` 停止隧道。

---

## 十五、最终配置速查

### Windows `~/.ssh/config`

```sshconfig
Host my-server
    HostName 服务器IP或域名
    User 用户名
    Port 22
    IdentityFile ~/.ssh/id_ed25519
    ServerAliveInterval 30
    ServerAliveCountMax 3

Host my-server-proxy
    HostName 服务器IP或域名
    User 用户名
    Port 22
    IdentityFile ~/.ssh/id_ed25519
    RemoteForward 127.0.0.1:17897 127.0.0.1:7897
    ExitOnForwardFailure yes
    ServerAliveInterval 30
    ServerAliveCountMax 3
```

### 启动专用隧道

```powershell
ssh -N my-server-proxy
```

### VS Code

```text
Remote-SSH: Connect to Host... -> my-server
```

### 远程服务器

```bash
ss -lnt | grep ':17897'
proxy_on
curl https://api.ipify.org
```

---

## 结语

SSH 反向端口转发可以在不安装服务器全局代理、不开放公网端口的情况下，让远程开发工具复用本机 Clash。真正决定这套方案是否稳定的，不是某一条命令，而是连接职责是否清晰。

最推荐的结构始终是：

```text
一条独立、长期运行的 SSH 连接负责隧道
+
任意数量的 VS Code、FinalShell 和普通 SSH 连接负责工作
+
远程 Shell 按需开启代理环境变量
```

这样既解决了 VS Code 第二次连接时的端口冲突，也解决了 FinalShell 不读取 Windows SSH 配置的问题，同时让故障排查、安全边界和日常维护都更加清楚。
