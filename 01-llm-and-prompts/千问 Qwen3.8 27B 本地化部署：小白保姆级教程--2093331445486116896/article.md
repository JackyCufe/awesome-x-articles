# 千问 Qwen3.8 27B 本地化部署：小白保姆级教程

@AdrianPunk115 · [X 原文](https://x.com/AdrianPunk115/article/2093331445486116896)

![封面](assets/001.jpg)

最近终于拿到了自己的 Linux 电脑，我开始拿它折腾本地大模型。这台机器装的是 Ubuntu 24.04，处理器是 AMD Ryzen AI MAX+ 395，配了 Radeon 8060S 和 128GB 统一内存，硬件很适合测试大参数模型。

一开始，我安装的是 **Qwen3.7 27B 的未审查版本，**体验了一把未审查的快乐！最近 **Qwen3.8 27B** 发布，我又重新跑了一遍完整部署。

![图片 1](assets/002.jpg)

为什么还要花时间部署到本地？对我来说，理由都很实际：

- 私人资料可以留在自己的电脑里；
- 模型下载完成后，断网也能继续使用；
- 没有网页会员的次数限制，想怎么测试都行；
- 本地 API 可以接进脚本、知识库和其他 AI 工具；
- 亲手跑一遍，才能知道自己的电脑能力边界在哪里。

所以我把这次安装过程完整记录了下来。没有命令行基础也没关系，文中会解释每一个容易卡住的名词、每一条命令在做什么，以及看到什么结果才算安装成功。

> 最后只需要一条命令，就能在自己的 Windows 或 Linux 电脑里跑起千问 Qwen3.8 27B。模型下载约 13GB，断网也能聊，文件和对话不用先传到云端。

下面分成 **Windows 和 Linux 两种安装方法**。Windows 使用图形安装包，Linux 使用官方安装脚本；从下载模型开始，两边使用同一条命令。

Linux 实机环境：

- 系统：Ubuntu 24.04.4 LTS
- 处理器：AMD Ryzen AI MAX+ 395
- 显卡：Radeon 8060S
- 内存：128GB 统一内存
- 运行工具：Ollama 0.33.1
- 模型：Julioa/Qwen3.8-27B-UD-Q3\_K\_XL

最终要运行的核心命令只有这一条：

```bash
ollama run Julioa/Qwen3.8-27B-UD-Q3_K_XL
```

先别急着复制。27B 模型比普通软件大得多，安装成功和用得舒服也不是同一个标准。前面的名词解释两种系统都通用；从第 2 章开始，按 **Windows** 或 **Linux** 小标题走自己的路线即可。

## 第 1 章 你到底要装什么

第一次接触本地大模型，最容易被一串名字劝退：Linux、Ollama、27B、GGUF、Q3、ROCm。

先把核心命令翻译成人话：

```text
让 Ollama 去下载 Julioa 发布的千问 Qwen3.8 27B 压缩模型，然后在这台电脑上启动它。
```

## 1.1 什么叫本地化部署

平时打开网页使用 AI，你输入的问题会通过网络发到服务商的服务器，答案算完后再传回来。

本地化部署是把模型文件下载到自己的硬盘，再用自己的 CPU、GPU 和内存完成计算。

它带来三个很直接的变化：

1. 模型下载完后，断网也能继续运行。
2. 日常对话和本地文档不用先上传到第三方服务器。
3. 没有按次收费，但速度、耗电和能跑多大的模型，都由自己的电脑决定。

可以把云端模型理解成叫网约车，本地模型更像自己买车。前者省心，后者掌控感更强，也要自己负责停车位、油耗和保养。

![图片 2](assets/003.jpg)

## 1.2 千问、Qwen3.8 和 27B 分别是什么意思

**千问**是模型家族的中文名，英文名是 **Qwen**。

**Qwen3.8** 是这次使用的模型版本名。

**27B** 代表大约 270 亿个参数。B 是 billion，也就是十亿。参数可以先理解成模型内部大量经过训练的数值旋钮；数量会影响模型规模，也会影响文件大小和运行时占用。

参数更多不保证每次回答都更好，但通常会带来更高的硬件门槛。8B 模型像一只登机箱，27B 已经接近大号托运行李，电脑得先腾出位置。

## 1.3 Ollama 是什么

模型权重只是一堆文件，自己不会弹出聊天窗口。

**Ollama** 负责三件事：

1. 下载和保存模型；
2. 把模型加载进内存或显存；
3. 提供终端聊天和本地 API，方便其他软件调用。

你可以把它理解成本地大模型的启动器。类似 Steam 管理游戏，Ollama 管理模型。

## 1.4 Q3_K_XL 又是什么

原始 27B 模型很大。为了让个人电脑装得下，社区会对模型做**量化**。

量化可以理解成压缩照片：保留主要信息，同时降低每个参数使用的精度，换来更小的文件和更低的内存占用。

模型名里的 Q3 表示大约 3 bit 级别的量化，K 是一类量化方案，XL 是发布者对这个变体的命名。压得越狠，文件通常越小，模型质量也可能多损失一点。

本教程选它的理由很现实：这台 128GB 统一内存的 AMD 机器能完整加载，实测运行尺寸约 29GB，还能把模型全部交给 GPU。

## 1.5 先记住四个容量

![图片 3](assets/004.jpg)

- 27.3B 是模型规模，约 273 亿参数。
- 13GB 是第一次运行需要下载的文件量。
- 29GB 是模型启动后在内存中的实测运行尺寸。
- 262K 是最大上下文，完整数字为 262144 token。

**token** 是模型读写文字时使用的最小单位。它不完全等于汉字：一个汉字可能占一个或多个 token，英文单词也会被拆分。

**上下文** 是模型当前一次能看到的全部内容，包含你的问题、历史对话和贴进去的资料。上下文开得越长，额外占用的内存越多。

## 第 2 章 先看电脑能不能装

先花两分钟检查环境。模型下载到 90% 才发现磁盘不够，最浪费时间。

## 2.1 Windows 和 Linux 都要看这三项

**第一是内存或显存容量。**

模型启动时必须放进内存。独立显卡通常使用自己的显存；AMD Ryzen AI MAX+ 395 这类机器采用统一内存，CPU 和集成显卡可以共享同一块内存。

本文这套 27B 量化模型在 Linux 实机上运行约占 29GB。64GB 比较从容，32GB 会很紧，尤其别一开始就把 262K 上下文全开。

**第二是磁盘。**

模型下载约 13GB，还要给临时文件、Ollama 本体和后续模型留位置。建议模型所在磁盘至少有 25GB 空闲空间；准备长期玩本地模型，留 100GB 更省心。

**第三是 GPU 支持。**

CPU 也能运行，只是速度通常慢很多。NVIDIA 和 AMD 显卡都可能被 Ollama 使用，具体能否加速还取决于显卡型号和驱动。

## 2.2 Windows：用 PowerShell 检查

点击开始菜单，搜索 **PowerShell** 或 **Windows Terminal**，打开后逐段复制下面三组命令。

先看 Windows 版本、系统位数和内存：

```powershell
Get-CimInstance Win32_OperatingSystem |
  Select-Object Caption, OSArchitecture,
    @{Name='RAM_GB';Expression={[math]::Round($_.TotalVisibleMemorySize/1MB)}}
```

再看显卡名称：

```powershell
Get-CimInstance Win32_VideoController | Select-Object Name
```

最后看 C 盘还剩多少空间：

```powershell
Get-PSDrive C |
  Select-Object Name,
    @{Name='Free_GB';Expression={[math]::Round($_.Free/1GB,1)}}
```

Ollama 官方要求 **Windows 10 22H2 或更新版本**。OSArchitecture 看到 64-bit，RAM\_GB 能确认内存，Free\_GB 最好不低于 25，这条路线就可以继续。

如果模型准备放在 D 盘，把最后一条命令里的 C 换成 D。后文也会说明如何改模型保存位置。

## 2.3 Linux：用终端检查

Ubuntu 中按 Ctrl + Alt + T 打开终端，再复制：

```bash
uname -m
cat /etc/os-release | sed -n '1,6p'
free -h
df -h "$HOME"
lspci | grep -Ei 'VGA|Display|3D'
ls -l /dev/kfd /dev/dri/renderD* 2>/dev/null
id -nG
```

每一行在查什么，可以先看这张 Linux 安装前质检图：

![图片 4](assets/005.jpg)

x86\_64 表示常见的 64 位 PC 架构。/dev/kfd 和 /dev/dri/renderD\* 是 Linux 暴露给程序的 GPU 入口；render 和 video 是允许账户访问显卡的权限组。

![图片 5](assets/006.jpg)

如果最后一行没有 render 或 video，执行：

```bash
sudo usermod -aG render,video "$USER"
```

然后退出 Ubuntu 账户并重新登录。权限组通常要重新登录才会生效。

如果系统提示 lspci: command not found，先安装它：

```bash
sudo apt update
sudo apt install -y pciutils
```

### sudo 是什么，密码输在哪里

sudo 表示临时用管理员权限执行命令。系统要求密码时，输入当前 Linux 账户的登录密码，然后按回车。

终端不会显示星号、圆点，光标也可能没有变化。这是正常的安全设计。密码只输入在自己电脑的系统提示里，不要发给任何人，也不要放进截图。

**这一章的过关标准：**能确认系统版本、64 位架构、内存、磁盘和显卡。Linux AMD 用户再确认 GPU 设备和权限组。

## 第 3 章 安装 Ollama

Ollama 是模型启动器。Windows 使用图形安装包，Linux 使用官方安装命令，装好以后运行模型的命令完全相同。

## 3.1 Windows：双击官方安装包

打开 Ollama Windows 官方下载页：

```text
https://ollama.com/download/windows
```

下载 OllamaSetup.exe，双击并按提示完成安装。官方安装器默认装在当前用户目录，不要求管理员权限，也不要从第三方下载站找安装包。

安装完成后，Ollama 会在后台运行。关闭原来的 PowerShell，再重新打开一个，执行：

```powershell
ollama --version
```

能显示版本号，说明命令已经装好。再执行：

```powershell
curl.exe -s http://127.0.0.1:11434/api/version
```

能返回版本信息，说明后台服务也正常。如果没有响应，从开始菜单搜索并启动 **Ollama**，再试一次。

### Windows 的 C 盘不够怎么办

在开始菜单搜索“环境变量”，打开“编辑账户的环境变量”，新建：

```text
变量名：OLLAMA_MODELS
变量值：D:\OllamaModels
```

保存后，先从系统托盘退出 Ollama，再从开始菜单重新启动。以后下载的模型就会放到新目录。

## 3.2 Linux：执行官方安装命令

在终端执行：

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

curl 负责从 Ollama 官网下载安装脚本，| sh 把脚本交给系统执行。安装时出现 sudo 密码提示，就输入当前 Linux 账户的登录密码。

安装结束后检查版本：

```bash
ollama --version
```

再检查后台服务：

```bash
sudo systemctl status ollama --no-pager
```

重点找：

```text
Active: active (running)
```

最后测试本机接口：

```bash
curl -s http://127.0.0.1:11434/api/version
```

能返回版本信息，这一步就过关。127.0.0.1 代表当前电脑，11434 是 Ollama 默认端口，可以把端口理解成软件在电脑里的门牌号。

下面是这台 Linux 电脑的安装结果，重点看版本号和后台服务状态：

![图片 6](assets/007.jpg)

## 第 4 章 下载并跑通第一次对话

从这一章开始，Windows 和 Linux 使用同一条命令。Windows 粘贴到 PowerShell，Linux 粘贴到终端。

## 4.1 复制准确的模型命令

```text
ollama run Julioa/Qwen3.8-27B-UD-Q3_K_XL
```

命令各部分的意思：

```text
ollama run                         下载并运行
Julioa/                            模型发布者
Qwen3.8-27B                        千问 3.8，约 270 亿参数
UD-Q3_K_XL                         这个仓库提供的量化变体
```

斜杠、连字符和下划线都要保留。最稳的方法是完整复制，不要手敲，也不要把 Q3\_K\_XL 写成别的版本。

## 4.2 第一次运行会下载约 13GB

本地没有这个模型时，Ollama 会自动下载。窗口会显示百分比、下载速度和剩余时间，实际耗时取决于网速。

网络中断后，重新执行同一条命令，Ollama 通常会接着已有文件继续下载。不用删除已经下载的部分，也别同时开多个窗口重复下载。

## 4.3 跑一个本地模型的 Hello World

下载完成后，窗口会进入聊天状态。输入：

```text
请只回复一句：千问 Qwen3.8 27B 已在这台电脑本地运行。
```

模型能返回中文，说明模型文件、Ollama、机器内存和终端对话都已经跑通。

输入下面这行退出聊天：

```text
/bye
```

看到模型回答就是本地大模型的 Hello World。到这里已经能用，下一章继续确认 GPU 有没有接手计算。

## 第 5 章 确认 GPU 真的在干活

模型会回答，只能证明它启动了。CPU 也能完成这件事，只是速度可能慢得多。

## 5.1 先认识 GPU、ROCm 和 offload

**GPU** 是显卡里的计算芯片，很适合同时处理大量相似计算。

**ROCm** 是 AMD 的 GPU 计算平台，可以理解成 Ollama 和 Radeon 显卡之间的翻译员。Windows 上的部分 AMD 显卡也会使用 Vulkan 作为加速通道。

日志里的 **offload** 指把模型计算层交给 GPU。66/66 layers offloaded 表示 66 层全部交给显卡处理，和删除文件无关。

## 5.2 两种系统都先运行这两条

让模型保持运行，再开第二个 PowerShell 或终端窗口，执行：

```text
ollama ps
```

重点看 PROCESSOR 一栏：

```text
PROCESSOR    100% GPU
```

100% GPU 表示模型全部放在 GPU 上运行，不代表显卡每一秒都保持 100% 满载。如果列表为空，通常是模型已经退出；重新运行模型，再立刻查询。

再核对模型身份：

```text
ollama show Julioa/Qwen3.8-27B-UD-Q3_K_XL
```

本文 Linux 实机看到：

```text
parameters      27.3B
context length  262144
quantization    Q3_K_L
```

27.3B 是实际参数规模，262144 是最大上下文，Q3\_K\_L 是 Ollama 读到的内部量化元数据。仓库名里的 UD-Q3\_K\_XL 和内部元数据不同，运行时继续使用完整仓库名即可。

## 5.3 Windows 显示 CPU 怎么办

先更新显卡驱动和 Ollama，再从系统托盘退出 Ollama，并从开始菜单重新启动。NVIDIA 用户按官方要求使用 551.61 或更新驱动；AMD 用户安装当前 Radeon 驱动，Ollama 会按硬件情况选择 ROCm 或 Vulkan。

重启后再次运行模型和 ollama ps。仍然只显示 CPU，就到 %LOCALAPPDATA%\\Ollama 查看 server.log，搜索 gpu、cuda、rocm、vulkan 或 error。

## 5.4 Linux AMD 显示 CPU 怎么办

先查设备和权限：

```bash
ls -l /dev/kfd /dev/dri/renderD* 2>/dev/null
id -nG
```

确认账户属于 render、video，并且修改权限后已经重新登录。再看 Ollama 日志：

```bash
journalctl -u ollama -n 120 --no-pager | grep -Ei 'rocm|gfx|gpu|offload|error'
```

设备文件不存在，或者日志一直找不到 ROCm 时，需要按具体 AMD 显卡型号检查驱动和 Ollama 兼容说明。

## 5.5 为什么下载 13GB，运行却占到 29GB

硬盘里的量化权重约 13GB，启动后还要给上下文缓存留空间。

这块缓存叫 **KV Cache**，可以理解成模型的临时草稿纸。对话越长，草稿纸越大，模型不用每次从第一句话重新计算。

本文 Linux 实机打开 262K 上下文时，KV Cache 约 16GB，加上模型缓冲区约 11.7GB，最终运行尺寸接近 29GB。

![图片 7](assets/008.jpg)

所以 32GB 机器即使能下载，加载超长上下文时也会很吃紧。小内存机器先把上下文降到 32768，成功率会高很多。

下面两张都是 Ubuntu 24.04 实机截图。Windows 用户看同样的 PROCESSOR、参数和模型名字段，不要求窗口外观一致。

![图片 8](assets/009.jpg)

![图片 9](assets/010.jpg)

这台 Linux 机器实测 66/66 层全部交给 ROCm，ollama ps 显示 100% GPU，热机生成速度是 **14.59 token/s**。速度会随硬件、提示词长度、上下文、系统负载和 Ollama 版本变化，这个数字只代表本文实机。

## 第 6 章 让网页和其他软件调用它

只想在终端聊天，可以先跳过这一章。想把模型接进网页、知识库、自动化脚本或聊天界面，就会用到 API。

## 6.1 API 是什么

API 是软件之间约定好的接口。把 Ollama 想成一家后厨，终端聊天是坐在店里点餐；API 像外卖窗口，其他程序按规定格式下单，再拿走模型返回的结果。

Ollama 默认地址是：

```text
http://127.0.0.1:11434
```

这个地址只供当前电脑访问，适合第一次部署时使用。

## 6.2 Windows：用 PowerShell 测试 API

先把要发送的内容保存成 [$body](https://x.com/search?q=%24body&src=cashtag_click)：

```powershell
$body = @{
  model = "Julioa/Qwen3.8-27B-UD-Q3_K_XL"
  stream = $false
  messages = @(
    @{role = "user"; content = "请用一句中文确认你正在本地运行。"}
  )
  options = @{num_ctx = 32768}
} | ConvertTo-Json -Depth 5
```

再把它发送给 Ollama：

```powershell
Invoke-RestMethod -Uri "http://127.0.0.1:11434/api/chat" -Method Post -ContentType "application/json" -Body $body | ConvertTo-Json -Depth 5
```

输出中出现 "done": true，并且 message 里有中文回答，说明 Windows 本地 API 已经跑通。

## 6.3 Linux：用终端测试 API

```bash
curl -s http://127.0.0.1:11434/api/chat \
  -H 'Content-Type: application/json' \
  -d '{
    "model": "Julioa/Qwen3.8-27B-UD-Q3_K_XL",
    "stream": false,
    "messages": [
      {"role": "user", "content": "请用一句中文确认你正在本地运行。"}
    ],
    "options": {"num_ctx": 32768}
  }' | python3 -m json.tool
```

同样检查 "done": true 和 message 里的中文回答。如果系统没有 Python，删掉最后的 | python3 -m json.tool，接口仍然能正常调用，只是输出会挤在一行。

num\_ctx: 32768 表示这次先用 32K 上下文，能明显降低内存压力。

## 6.4 为什么先别开放公网

127.0.0.1 只允许本机访问，安全边界很清楚。

有些教程会让 Ollama 监听 0.0.0.0，这样局域网甚至公网设备都可能连进来。公开服务还需要身份验证、访问控制、防火墙和日志，新手先保持默认地址。

## 第 7 章 小白最常遇到的六个问题

Windows 先关闭 PowerShell，重新打开，再执行：

```powershell
where.exe ollama
ollama --version
```

Linux 也先重开终端并执行 ollama --version。仍然找不到，就重新走第 3 章对应系统的官方安装步骤。

## 7.2 无法连接 127.0.0.1:11434

Windows 从开始菜单启动 **Ollama**，然后重新执行：

```powershell
curl.exe -s http://127.0.0.1:11434/api/version
```

Linux 重启后台服务：

```bash
sudo systemctl restart ollama
sudo systemctl status ollama --no-pager
```

看到 active (running) 后，再测试接口。

## 7.3 Linux 输入密码时屏幕完全没变化

这是 Linux 终端的正常行为。输入当前账户登录密码并按回车即可，屏幕不会显示星号。

密码错误时系统会重新提示。不要把密码发进聊天窗口，也不要为了截图把密码写在命令后面。

## 7.4 模型下载一半断线

Windows 和 Linux 都重新执行：

```text
ollama run Julioa/Qwen3.8-27B-UD-Q3_K_XL
```

Ollama 通常会继续已有下载，不需要从零清空。

## 7.5 ollama ps 显示 CPU

先确认模型仍在运行，再按第 5 章对应系统的检查方法处理。

Windows 先更新驱动、更新 Ollama、重启 Ollama；Linux AMD 先查 /dev/kfd、权限组和服务日志。处理后重新运行模型，再用 ollama ps 验收。

## 7.6 一启动就吃掉接近 30GB

本文实机加载了 262K 上下文，KV Cache 本身约占 16GB。API 调用时先设置：

```json
"options": {"num_ctx": 32768}
```

32K 上下文已经能容纳很长的普通对话，也更适合内存紧张的机器。仍然吃紧，就换更小的模型或占用更低的量化版本。

## 第 8 章 一张图完成最终验收

![图片 10](assets/011.jpg)

先做系统自己的服务检查：

- Windows：curl.exe -s http://127.0.0.1:11434/api/version 返回版本。
- Linux：sudo systemctl status ollama --no-pager 显示 active (running)。

后面六项两种系统相同：

- 版本：ollama --version 能显示版本号。
- 模型：ollama list 出现完整模型名。
- 参数：ollama show Julioa/Qwen3.8-27B-UD-Q3\_K\_XL 显示 27.3B。
- GPU：模型运行时，ollama ps 显示 100% GPU。
- 对话：ollama run Julioa/Qwen3.8-27B-UD-Q3\_K\_XL 后能正常回答中文。
- API：调用 /api/chat 后返回 done: true。

### 关于作者

**Punk**｜中科大管理学硕士｜AI提示词、AI小白教程｜Punk系列Skills作者｜3个月赚了8位数｜Learn in Public｜FDE文章浏览量240w｜[@AdrianPunk115](https://x.com/@AdrianPunk115)

![图片 11](assets/012.jpg)
