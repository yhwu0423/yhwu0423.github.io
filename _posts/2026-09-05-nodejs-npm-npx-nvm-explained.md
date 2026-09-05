---
layout: post
title: "一文理清 Node.js 全家桶：Node.js、npm、npx、nvm 都是什么"
subtitle: "用「发动机与配件」的类比，讲透四者的关系与日常用法"
date: 2026-09-05 17:00:00
author: "wyh"
header-style: text
catalog: true
tags:
  - 笔记
  - Node.js
  - npm
  - 工具
---

装个前端项目、跑个 CLI 工具（比如你用的 Codex CLI），总会碰到这四个名词：Node.js、npm、npx、nvm。它们名字像亲兄弟，干的活却完全不同：一个是**运行时**、一个是**包管理器**、一个是**临时执行器**、一个是**版本管理器**。

先给一个贯穿全文的类比：

| 名词 | 类比 | 职责 |
|---|---|---|
| Node.js | 发动机 | 让 JavaScript 代码能跑起来 |
| npm | 配件商店 + 采购员 | 下载、管理别人写好的代码包 |
| npx | 共享汽车 | 临时用一次某个工具，用完就还 |
| nvm | 车库管理员 | 同时管理多台不同年代的"发动机" |

---

## 一、一张图看懂关系

```text
              nvm（版本管理器）
                │ 安装、切换多个 Node 版本
                ▼
        Node.js（运行时）
           │        │ 每个 Node 版本自带一套
           │        ▼
           │      npm（包管理器）◄──── npm 仓库（registry，代码包的"应用商店"）
           │        │ 下载安装
           │        ▼
           │   node_modules（项目依赖）
           │        ▲
           │        │ npx：找到包的可执行文件直接运行；
           │        │      没有就临时下载，用完即扔
           ▼
     运行你的 JS 代码（网站、脚本、CLI 工具）
```

一句话串起来：**nvm 负责装 Node.js → Node.js 自带 npm → npm 负责下载代码包 → npx 负责"不安装直接运行"代码包**。

## 二、Node.js：让 JavaScript 跑出浏览器

JavaScript 天生只能跑在浏览器里（由浏览器内核的 JS 引擎执行），能力被限制在网页沙箱中——不能读写文件、不能监听端口、不能开服务器。

**Node.js 做的事情**：把 Chrome 的 V8 引擎单独抽出来，再配上一套操作系统能力的 API（读写文件 `fs`、网络 `http/net`、进程 `process` 等），让 JavaScript 挣脱浏览器，**成为一种能写服务器、写命令行工具、写桌面软件的通用语言**。

所以 Node.js 是一个**运行时（Runtime）**，不是编程语言，也不是框架。

**"运行时"是什么？** 很多人会以为"运行时 = 把代码翻译成机器码的翻译器"，其实翻译只是它的第一份工作。跟着一条真实命令走一遍就懂了。假设 `app.js` 的内容是：

```js
const fs = require('fs');
console.log(fs.readFileSync('a.txt', 'utf8'));
```

在终端敲 `node app.js` 后，真实发生的事情是：

1. Windows 启动 `node.exe`——它才是真正的程序（C++ 写的、编译好的）；你的 `app.js` 此刻只是一个**文本文件**，CPU 执行不了"文字"；
2. node.exe 里的 **V8 引擎**读出这段文字，现场编译成机器码交给 CPU——"翻译"只发生在这一步，译完它并不下班；
3. 执行到 `fs.readFileSync('a.txt')` 时：JS 语言本身**根本不会读文件**（语言规范里就没有这个功能），是 Node.js 的 `fs` 模块替你调用 Windows 的系统接口，把文件从硬盘读出来交还给代码；
4. 运行期间变量占用的内存，由 V8 的**垃圾回收器**自动释放，不用你管；
5. 如果代码里有 `setTimeout`、HTTP 服务器，Node 的**事件循环**会一直运行不退出，盯着"定时器到点没、网络包来了没"，到点就调用你的回调函数。

看到这里可能有个疑问：**既然都翻译成机器码了，CPU 直接执行不就行了，为什么 node 还必须在场？** 因为翻译是完全的，但翻译出来的机器码**不是自给自足的**——`fs.readFileSync('a.txt')` 编译出来的机器码，效果相当于"调用 node.exe 内部的读文件函数，参数是 'a.txt'"：真正读文件的活是 node 的 C++ 代码干的，你的机器码里到处是这样的"跳回运行时"调用，把 node 删掉这些调用就全部落空。从 CPU 的视角看，它从头到尾执行的进程都是 **node.exe**：你的代码被编译后只是这个进程里的一段，执行几行就会跳回 node 自己的代码（读文件、分配内存、等待事件），垃圾回收甚至会在中途暂停你的代码去做清理。换句话说，不是"翻译完交给 CPU，node 就下班了"，而是你的机器码根本离不开这个进程。（想让代码脱离 node 独立运行也行，办法是把整个运行时打包进 exe——claude.exe 那 218 MB 的大头就是它。）

所以：**运行时 = 翻译官（编译机器码）+ 全程管家（系统调用、内存回收、事件调度）**，从程序启动到退出全程在线服务。

对比 C 语言就更清楚了：C 代码在**运行之前**就被编译器一次性翻译成机器码、生成 exe，双击直接跑，操作系统直接伺候它，不需要专门的运行时。你可以亲手验证 JS 的处境——双击一个 `.js` 文件，Windows 只会问"用什么程序打开"，因为操作系统根本不认识 JS 文本，必须靠 `node` 这个程序全程伺候。

最后呼应三个词的区别：JavaScript 是**语言**（纸面上的语法规则）；浏览器和 Node.js 是两种**运行时**（同一段 JS，在浏览器里能操作网页、在 Node 里能读写文件——语言相同，环境不同，能干的活就不同）；Express、Vue 这类**框架**则是运行时之上的"半成品结构"，你往里填业务逻辑。上一节的 claude.exe 不需要装 Node，正是因为运行时被一起打包进了 exe。它的典型用途：

- **Web 后端**：Express/Koa/NestJS 等框架写接口服务；
- **命令行工具**：你在服务器上用的 Codex CLI、Claude Code，都是用 JavaScript/TypeScript 编写、以 npm 包形式分发的工具；
- **前端构建工具链**：Vite、Webpack、TypeScript 编译器都运行在 Node.js 上——这就是为什么写前端必须先装 Node，哪怕你的代码最终跑在浏览器里；
- **桌面应用**：VS Code、Discord 用的 Electron，内核就是 Node.js + Chromium。

## 三、npm：包管理器 + 包仓库

npm（Node Package Manager）这个名字实际上指**两样东西**：

**1. npm 仓库（registry）**：网址 npmjs.com，全球最大的代码包仓库，托管着几百万个别人写好的开源包（React、lodash、Express……），可以类比成"代码界的应用商店"。

**2. npm 命令行工具**：负责从这个仓库下载、安装、升级、卸载包。**它随 Node.js 一起自动安装**，装了 Node 就有 npm，不用单独装。

日常开发中，npm 围绕两个文件工作：

- **`package.json`**：项目的"配置清单"，记录项目名称、依赖了哪些包及版本范围、可执行哪些脚本命令；
- **`node_modules/`**：依赖包实际下载存放的目录（体积巨大，绝不提交 git，删了随时能按清单重装）。

常用命令就这几个：

```bash
npm install            # 按 package.json 安装全部依赖（克隆项目后第一件事）
npm install lodash     # 安装指定包（写入依赖清单）
npm install -g @openai/codex   # 全局安装（CLI 工具的典型装法，任何目录都能用）
npm run dev            # 执行 package.json 里 scripts 定义的 dev 命令
npm uninstall lodash   # 卸载
```

注意**本地安装 vs 全局安装**的区别：不加 `-g` 装到当前项目的 node_modules，只有这个项目能用；加 `-g` 装到系统级目录，任意终端都能调用——CLI 工具（codex、typescript、vite）用全局，项目依赖用本地。

### "包"就是软件吗？以 Claude Code 为例

**Claude Code 本身就是一个 npm 包**，全名 `@anthropic-ai/claude-code`（`@` 后面、斜杠前面是组织名，斜杠后是包名，这种叫"作用域包"）。借它把几个常见困惑一次说清：

**1. 包 = 软件吗？** 不完全是。一个 npm 包就是"**一坨打包好的代码 + 一份 package.json 说明书**"，分两种角色：

- **库（library）**：给别的代码调用的"零件"，自己不能单独运行，如 lodash、react——npm 上绝大多数包是这种；
- **CLI 工具/应用**：带可执行入口的完整程序，最接近你理解的"软件"。Claude Code、Codex、create-vite 都是这种。

**2. `npm install` 就是在下载这个软件吗？** 对，准确说它做了三件事：从 npm 仓库下载包的压缩包（.tgz）→ 解压到 node_modules（全局安装则解压到全局目录）→ 读取 package.json 里的 `bin` 字段（声明了命令名到入口文件的映射），在 PATH 目录里建立对应的命令入口（Windows 上是个 `.cmd` 小脚本）。

**3. 跑起来的一定是 Node.js 吗？** 对大多数 CLI 包，是的。以一个典型工具 `mytool` 为例，你在终端敲 `mytool` 时的完整链路：

```text
mytool 命令
  → 系统沿 PATH 找到 mytool.cmd（命令入口脚本）
    → 脚本内部调用 node <全局目录>/.../cli.js
      → Node.js 执行这个包的代码
```

所以"用 npm 下载软件、用 Node.js 跑起来"的理解基本正确，中间只多一步"建立命令入口"。

**但 Claude Code（2.x）恰恰是个著名的例外，而且很能说明问题。** 它已经从"JS 分发"改成了**二进制分发**，我在自己电脑上查证过安装目录：package.json 里 `bin` 直接指向一个 **218 MB 的 `claude.exe`**，`claude.cmd` 里没有任何 node 调用，直接运行这个 exe。它的 npm 包其实只是个**引导安装器**：安装时的 `postinstall` 脚本（install.cjs）按你的平台，从 `optionalDependencies` 列出的平台专属包（`claude-code-win32-x64`、`darwin-arm64` 等共 8 个）中取出对应的原生二进制放到 bin/ 下。这个 exe 内部已把 JS 代码和 JS 运行时一起编译成单文件原生程序（Bun 的单文件编译），所以**运行时完全不需要 Node.js**——只有走 npm 安装这个过程才用到 node（npm 本身和引导脚本都靠它运行）。官方也顺势提供了完全不碰 npm 的原生安装脚本。esbuild、swc、Biome 等工具用的也是同款分发模式。

这个例外的意义在于：**npm 只是一个"代码/文件分发渠道"，包里装的可以是 JS，也可以是编译好的二进制**——不要默认"npm 包 = 跑在 Node.js 上的 JS"。

### 一个 npm 包是如何被开发出来的？

还以 Claude Code 这类 CLI 工具为例，开发者从写出它到你能安装，大致六步：

1. **初始化**：`npm init` 生成 package.json，填写包名、版本号、入口文件；
2. **写代码 + 声明依赖**：用 JS/TS 实现功能，用到别人的包就写进 `dependencies`（终端界面、网络请求、文件处理……）。npm 包都是"站在巨人肩膀上"，一个工具往往又依赖几十上百个包，安装时会被递归地一起拉下来；
3. **声明命令入口**：在 package.json 里写 `bin` 字段，如 `"bin": { "mytool": "./cli.js" }`，并在 cli.js 第一行写上 `#!/usr/bin/env node`（告诉操作系统"请用 node 执行我"）——上面那条运行链路能成立，靠的就是这两行声明；
4. **本地调试**：`npm link` 把开发中的包临时链接到全局，模拟"已安装"的状态边改边测；
5. **发布**：在 npmjs.com 注册账号 → `npm login` → `npm publish`。几秒后全世界都能 `npm install` 到这个包，npmjs.com 上也会生成它的主页（版本历史、周下载量等，可以搜 `@anthropic-ai/claude-code` 实地看看）；
6. **迭代**：修 bug、加功能 → 按语义化版本规则升级版本号 → 再次 publish；用户端 `npm update -g` 即可升级。

如果你想发布自己的包，流程一模一样——npm 仓库对所有人免费开放，这也是它包数量全球第一的原因。

## 四、npx：临时运行一个包，用完即扔

npx 是 npm 5.2 之后**自带的另一个命令**（Node Package Execute），专门解决一个痛点：**有些工具你只想用一次，不想为它做全局安装**。

典型场景是项目脚手架，比如创建一个新前端项目：

```bash
npx create-vite my-app
```

这条命令做了什么：npx 发现本地没有 `create-vite` 这个包，就**临时下载到缓存目录 → 运行 → 用完丢弃**。全程没有全局安装任何东西，下次再用时再拉最新版。

npx 的查找顺序：当前项目的 `node_modules` → 全局已安装的包 → 都没有才临时下载。所以它还有两个实用功能：

```bash
npx cowsay@latest "hello"   # 指定版本运行
npx tsc --init              # 运行项目本地安装的 tsc（不用关心它在 node_modules 深处的路径）
```

一句话对比：**npm 管"安装"，npx 管"运行"**；一次性工具用 npx，天天用的工具才 `npm install -g`。

## 五、nvm：Node 版本管理器

问题来了：Node.js 版本迭代很快，而**不同项目对版本的要求不一样**——老项目只认 Node 14，新项目要求 Node 20+，直接升级系统里的 Node 会把老项目搞崩。

**nvm（Node Version Manager）就是干这个的**：让多个 Node 版本在同一台机器上和平共处，一条命令切换：

```bash
nvm install 20        # 安装 Node 20
nvm install 18        # 再装个 Node 18，互不干扰
nvm use 20            # 当前终端切换到 Node 20
nvm list              # 查看已安装的所有版本
node -v               # 确认当前生效的版本
```

两个关键点：

- **每个 Node 版本自带一套独立的 npm 和全局包**。用 nvm 切到 Node 20 后，`npm install -g` 装的东西只存在于 Node 20 环境下，切到 18 就看不到了——这正是"环境隔离"的意义；
- **平台差异**：macOS/Linux 用原版 nvm（一个 shell 脚本）；Windows 用的是独立项目 **nvm-windows**（GitHub 搜 `coreybutler/nvm-windows`，下载 `nvm-setup.exe` 安装），命令几乎一样。

### nvm 把 Node.js 装到了哪里？

搞清楚安装目录，就能理解"切换版本"的本质：

- **Windows（nvm-windows）**：所有版本装在 `C:\Users\<用户名>\AppData\Roaming\nvm\` 下，一个版本一个子目录（如 `v22.14.0\`）。同时 nvm 维护一个**符号链接** `C:\Program Files\nodejs`，永远指向当前启用的版本目录，而系统 PATH 里添加的正是这个链接。所以 `nvm use` 切换版本的本质，就是**把链接重新指向另一个目录**——瞬间完成，不用重装任何东西。
- **macOS / Linux（nvm）**：版本装在 `~/.nvm/versions/node/v22.14.0/` 这样的目录里，nvm 通过修改当前 shell 的 PATH 环境变量来指向选中的版本。

这也解释了一个常见现象：**全局安装的 npm 包"跟着版本走"**。全局包装在各版本自己的目录里（如 Windows 上的 `v22.14.0\node_modules\`），切到另一个版本后，之前装的全局工具就"消失"了——其实没丢，只是躺在上一个版本的目录里。所以切换版本后，常用的全局工具（如 codex、claude-code）需要重新 `npm install -g` 一遍。

随时可以用命令确认当前环境的位置：

```bash
where node        # Windows：查看当前 node.exe 在哪
which node        # macOS / Linux：同上
npm root -g       # 查看当前全局包的安装目录
```

类似的工具还有 fnm、volta 等，作用相同，nvm 最老牌、教程最多，新手选它即可。

## 六、常见周边名词

看教程时还会碰到这些词，一并交代：

- **LTS 与 Current**：Node.js 的版本线。偶数版本（18、20、22）会进入 **LTS**（长期支持，稳定，生产和学习都用它）；奇数版本是尝鲜的 Current。下载时永远优先选 LTS。
- **npm 镜像源**：npm 官方仓库在国外，国内直连很慢。换成国内镜像（原淘宝镜像）后飞起：

  ```bash
  npm config set registry https://registry.npmmirror.com
  ```

- **yarn / pnpm**：npm 的竞品包管理器，安装更快、更省磁盘（pnpm 用硬链接共享依赖）。用法和 npm 几乎一样，项目说用哪个就用哪个，新手先把 npm 用熟。
- **package-lock.json**：npm 自动生成的"精确版本锁定文件"。package.json 里写的往往是版本范围（如 `^4.17.0`），lock 文件锁死实际安装的具体版本，保证你和队友、服务器装出来的一模一样。**要提交 git，别删**。

## 七、实战：从零跑起一个项目

以 Windows 为例，完整走一遍上面所有概念：

```powershell
# 1. 安装 nvm-windows（GitHub: coreybutler/nvm-windows，下 nvm-setup.exe 一路下一步）

# 2. 用 nvm 安装并使用 LTS 版 Node.js（npm 会自动跟着装好）
nvm install lts
nvm list                # 查看装好的版本号
nvm use 22.14.0         # 切换使用（换成你实际装好的版本号）
node -v                 # 验证：v22.x.x
npm -v                  # 验证：10.x.x

# 3.（国内必做）换镜像源
npm config set registry https://registry.npmmirror.com

# 4. 跑一个项目：克隆 → 装依赖 → 启动
git clone <项目地址>
cd <项目目录>
npm install             # 按 package.json + lock 文件安装全部依赖
npm run dev             # 启动开发服务器

# 5. 体验 npx：不安装，直接运行一个小工具
npx cowsay "Node.js 真香"
```

全程对应关系：第 1~2 步是 **nvm** 在工作，第 2 步装出了 **Node.js** 和 **npm**，第 4 步是 **npm** 在工作，第 5 步是 **npx** 在工作。

## 八、总结

| 名词 | 是什么 | 一句话 |
|---|---|---|
| Node.js | JavaScript 运行时 | 让 JS 脱离浏览器，能写后端和工具 |
| npm | 包管理器 + 包仓库 | 下载安装别人的代码包，随 Node 自带 |
| node_modules | 依赖存放目录 | 体积巨大，不提交 git，可删可重装 |
| package.json | 项目配置清单 | 记录依赖和脚本命令 |
| package-lock.json | 精确版本锁定文件 | 保证各环境装出一致，要提交 git |
| npx | 包临时执行器 | 不安装直接运行，用完即扔 |
| nvm | Node 版本管理器 | 多版本共存，一条命令切换 |
| LTS | 长期支持版本 | 偶数版本，生产学习都用它 |
| 镜像源 | npm 仓库的国内镜像 | 国内加速下载必备 |

最后记住那条主线：**nvm 装 Node.js → Node.js 自带 npm → npm 下载包 → npx 临时运行包**。四个工具各管一层，看到任何相关教程都不会再懵了。
