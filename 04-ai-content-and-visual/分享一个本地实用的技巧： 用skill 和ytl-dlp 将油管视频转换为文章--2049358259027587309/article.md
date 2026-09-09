# 分享一个本地实用的技巧： 用skill 和ytl-dlp 将油管视频转换为文章

@PandaTalk8 · [X 原文](https://x.com/PandaTalk8/article/2049358259027587309)

![封面](assets/001.jpg)

## 创建 SKILL：将你喜欢看的 YOUTUBE 视频转换为文章

![图片 1](assets/002.jpg)

我每周会看不少英文 YouTube 视频，主题集中在 AI 研究、创业访谈、技术分享。看完之后常常想做两件事：一是把核心观点摘出来存成笔记，二是把内容改写成中文文章发出去。

手工做这件事的成本不低。一个 30 分钟的视频，光是把字幕一条条复制下来再清洗时间戳就要十几分钟。更不用说后续的内容重组、翻译、改写。做过三五次之后，我决定把它沉淀成一个 Skill。

但这次我没有自己写 Skill。我让 Claude Code 写——更准确地说，我用一段 Prompt 让它从零生成了这个 Skill 的全部文件。整个过程不到五分钟。

这篇文章讲两件事：第一，YouTube 视频转文章背后的实现原理；第二，怎么让 Claude Code、OpenClaw 或 Codex 这类编码 Agent 替你把这个 Skill 写出来，并且写完就能用。

### 一、实现原理：三步把视频变成文章

整条链路只有三步，先说清楚每一步在做什么，后面写 Prompt 才有依据。

**第一步：拿到视频字幕**

YouTube 上绝大多数视频都有字幕——要么是创作者上传的人工字幕，要么是平台自动生成的。yt-dlp 这个命令行工具可以直接把字幕拉下来，不需要下载视频本体。

关键命令长这样：

yt-dlp --write-subs --write-auto-subs \\
       --sub-langs "en,zh-Hans,zh-Hant" \\
       --skip-download \\
       --sub-format vtt \\
       "[https://www.youtube.com/watch?v=VIDEO\_ID](https://www.youtube.com/watch?v=VIDEO_ID)"

--skip-download 决定了不下载视频本体，只取字幕。优先级上人工字幕 \> 自动字幕。同时用 yt-dlp --dump-json --skip-download 可以拿到标题、作者、时长、上传时间，文章开头的"原视频信息"区块要用。

**第二步：清洗字幕**

原始字幕是 VTT 或 SRT 格式，长这样：

00:00:02.879 --\> 00:00:02.889 align:start position:0%
so\<00:00:00.319\>\<c\> today\</c\>\<00:00:00.560\>\<c\> we're\</c\>

自动字幕的麻烦在于：每句话会重复两到三次（滚动效果），单词级时间戳标签 \<c\> 把句子切得支离破碎，行间还有大量重复。要做的是：去 HTML 标签、去时间戳、按行去重、合并段落。处理完一份 800 行的 VTT 大致能浓缩成 60 个段落、5000 字左右的纯文本，Token 用量降 60% 以上。

**第三步：交给大模型改写**

清洗后的纯文本作为输入，按一份模板让模型改写成中文文章。模板要点是：

- 不按时间顺序排，而是按主题逻辑重组
- 保留视频里的故事和数据
- 补上下文（视频里嘉宾默认听众懂的背景，文章读者不一定懂）
- 把口语化措辞换成书面语，但保留观点本身

这三步加起来，就是一个完整的"视频→文章"流水线。把它沉淀成 Skill 之后，使用门槛就只剩"丢一个 YouTube 链接"。

### 二、为什么用 AGENT 来写 SKILL

Skill 本身是几个文件：一个 SKILL.md、一两个脚本、一份 Prompt 模板。手工写大概要一两个小时——主要时间不在写代码，而在调试触发词、试 yt-dlp 各种参数、迭代清洗逻辑。

但这种"骨架清晰、需要试错"的工作，恰好是 Claude Code、OpenClaw、Codex 这类编码 Agent 最擅长的：

- **Claude Code**：Anthropic 官方 CLI，原生理解 Skill 概念，知道 ~/.claude/skills/ 目录结构和 frontmatter 格式
- **OpenClaw**：开源的 Claude Code 兼容方案，行为基本一致，适合自己定制 Harness
- **Codex**：OpenAI 的 Coding Agent，能力相当，差别主要在工具协议和模型

它们三个都能直接读写文件、执行命令、跑测试，所以"写 Skill"这件事可以完全交给它们：你描述清楚要什么，它生成文件、试运行、修 Bug、再写一份测试用例。

下面给一段可以直接复用的 Prompt，扔给三者中任意一个都能跑通。

### 三、用 PROMPT 生成整个 SKILL

把下面这段话完整复制给 Claude Code（或 OpenClaw / Codex），它会从零创建出整个 Skill 目录、写完所有文件、自我验证一次。

我要在 ~/.claude/skills/youtube-to-article/ 下创建一个 Skill，
功能是把 YouTube 视频转换为中文文章。请按下面的要求实现。
​
# 功能流程
1. 输入：用户给一个 YouTube 链接
2. 用 yt-dlp 拉取视频元信息（--dump-json --skip-download）
3. 用 yt-dlp 拉取字幕（优先 en/zh-Hans/zh-Hant 人工字幕，
   退而求其次自动字幕，--skip-download --sub-format vtt）
4. 清洗 VTT 字幕成连续纯文本（去 HTML 标签、去时间戳、
   按行去重、合并段落）
5. 按文章模板改写成中文文章，输出到当前工作目录
​
# 目录结构
youtube-to-article/
├── SKILL.md
├── scripts/
│   ├── fetch\_subs.sh    # 调 yt-dlp 拉元信息和字幕
│   └── clean\_vtt.py     # 清洗 VTT 为纯文本
└── prompts/
    └── article\_template.md  # 改写文章用的模板
​
# SKILL.md 要求
- frontmatter 包含 name 和 description
- description 里穷举触发词：「把这个视频做成文章」「YouTube 转文章」
  「视频改写」「video to article」「这个 YouTube 视频帮我整理一下」
- 写明触发条件：用户提供了 [youtube.com](https://youtube.com/) 或 [youtu.be](https://youtu.be/) 链接
- 写明不触发场景：用户只是分享链接没有改写意图、链接不是 YouTube
- 列出依赖：yt-dlp、python3
- 给出六步执行流程，每步注明输入输出和调用的脚本
- 写明不下载视频本体、保留原作者署名等注意事项
​
# clean\_vtt.py 要求
- 输入 VTT 文件路径，输出清洗后的纯文本到 stdout
- 处理 WEBVTT 头部、HTML 标签、时间戳行、对齐参数
- 按行去重保留顺序
- 按句号/问号/感叹号合并成段落
- 不超过 50 行，注释精简
​
# article\_template.md 要求
- 文章标题为视频的中文译名
- 引用区块带上原视频标题、作者、时长、上传日期、链接
- 主体用 ## 拆 3-5 个章节（按主题不按时间）
- 改写规则写在模板末尾的注释里：保留例子、补上下文、
  口语化表达改书面语、保留专业术语英文原词
​
# 验证
全部写完之后：
1. python3 -c "import re" 确认 Python 可用
2. yt-dlp --version 确认工具存在；如果没装提醒用户
   brew install yt-dlp
3. 用一个真实 YouTube 链接（你可以选一个 5 分钟以内的公开视频）
   端到端跑一次，确认能输出 transcript 和文章草稿
4. 打印整个 Skill 的目录树和每个文件的前 10 行供我确认
​
要点：
- 字幕清洗一定要去重，自动字幕滚动重复是大坑
- 输出文章要符合中文标点规范（中文双引号、全角逗号）
- 文章最多两级标题（# 和 ##），不用 ###
- 不要把 API key、路径写死在 Skill 里

这段 Prompt 的设计有几个细节值得说一下：

**把流程拆到原子粒度**：六步执行流程每一步都写明了输入输出和工具，不留歧义空间。Agent 越是高自由度，越需要外部约束把它框在正确轨道上。

**显式列出验证步骤**：让 Agent 自己跑一遍真实链接，比"写完就交差"靠谱得多。Claude Code 和 OpenClaw 都能直接执行 yt-dlp 命令，Codex 在沙箱模式下也可以。验证不通过它会自己定位问题、修 Bug。

**写出已知坑点**：自动字幕滚动重复、中文标点规范这些经验，写进 Prompt 比让 Agent 自己踩一次效率高得多。

**约束输出粒度**：clean\_vtt.py 不超过 50 行、文章最多两级标题——这些限制让 Agent 不至于过度发挥。

跑完这段 Prompt，三五分钟之内你会拿到一个能直接用的 Skill。如果哪一步不对，再回去补一句"重新生成 clean\_vtt.py，多处理 align:start 这个对齐参数"，Agent 会增量修复。

### 四、CLAUDE CODE / OPENCLAW / CODEX 三者的微小差别

同一段 Prompt 在三个工具里跑，结果大同小异，但有几个细节差异值得注意。

**Claude Code**

原生支持 ~/.claude/skills/ 目录约定，写完 Skill 之后下次启动 Claude Code 会自动加载。SKILL.md 的 frontmatter 格式也是它的"母语"。如果你在 Claude Code 生态里，用它写 Skill 是最顺的选择。

**OpenClaw**

行为和 Claude Code 几乎一致，但 Skill 的加载路径可能在自定义配置里。生成的文件本身一模一样，差异只在"放哪里能被识别"。如果你部署了自己的 Harness，记得在 Prompt 末尾加一句："Skill 安装路径以 OpenClaw 配置为准，必要时打印当前 Skill 搜索路径"。

**Codex**

Codex 不一定原生认 ~/.claude/skills/ 这个约定。我的做法是让它先生成文件到这个路径，然后单独写一段"如何在 Claude Code 里启用这个 Skill"的说明文档。这样 Codex 写、Claude Code 用，两边各司其职。

三个工具背后的能力差距已经很小，差别更多在工具协议、沙箱权限、CLI 体验上。**真正决定 Skill 质量的是 Prompt，不是工具。**

### 五、生成的 SKILL 长什么样

Agent 跑完上面那段 Prompt 之后，最终目录会是这样：

~/.claude/skills/youtube-to-article/
├── SKILL.md
├── scripts/
│   ├── fetch\_subs.sh
│   └── clean\_vtt.py
└── prompts/
    └── article\_template.md

SKILL.md 的 frontmatter 区块大致是：

---
name: youtube-to-article
description: 将 YouTube 视频转换为中文文章。用 yt-dlp 拉取字幕和元信息，
  清洗后按文章模板改写。当用户说「把这个视频做成文章」「YouTube 转文章」
  「视频改写」「video to article」「这个 YouTube 视频帮我整理一下」时触发。
  触发条件：用户提供了 [youtube.com](https://youtube.com/) 或 [youtu.be](https://youtu.be/) 链接。
---

clean\_vtt.py 大致 30 行，关键是去重和段落合并：

import re, sys
from pathlib import Path
​
def clean(path):
    text = Path(path).read\_text(encoding="utf-8")
    text = re.sub(r"^WEBVTT.\*?\\n\\n", "", text, count=1, flags=re.DOTALL)
    text = re.sub(r"\<\[^\>\]+\>", "", text)
    text = re.sub(r"^\\d{2}:\\d{2}:\\d{2}.\*--\>.\*$", "", text, flags=re.M)
    text = re.sub(r"align:start position:\\d+%", "", text)
    seen, lines = set(), \[\]
    for line in text.splitlines():
        line = line.strip()
        if line and line not in seen:
            seen.add(line); lines.append(line)
    full = re.sub(r"\\s+", " ", " ".join(lines))
    return re.sub(r"(\[.!?\])\\s+", r"\\1\\n\\n", full).strip()
​
if \_\_name\_\_ == "\_\_main\_\_":
    print(clean(sys.argv\[1\]))

article\_template.md 是改写质量的关键，结构上是：

**# {{中文标题}}**
​
\> 原视频：\[{{英文标题}}\]({{URL}})
\> 作者：{{uploader}} ｜ 时长：{{duration}} ｜ 上传：{{upload\_date}}
​
{{开篇导语：用两三句话点出核心观点}}
​
**## 一、{{核心论点 A}}**
​
{{展开论述，引用具体例子和数据}}
​
**## 二、{{核心论点 B}}**
​
...
​
**## 结尾**
​
{{总结：从视频里提炼对读者有用的启示}}
​
\<!--
改写规则：
- 按主题重组，不按时间顺序
- 保留视频里的故事和数据
- 补上下文（嘉宾默认听众懂的背景，文章读者不一定懂）
- 口语化表达改书面语，保留专业术语英文原词
--\>

这三个文件加起来不到 80 行代码 + Markdown，但配合起来就是一条完整的视频改写流水线。

### 六、用 SKILL 完成一次视频转文章

Skill 装好之后，在 Claude Code 里直接说：

把这个视频做成中文文章 [https://www.youtube.com/watch?v=xxx](https://www.youtube.com/watch?v=xxx)

Claude 的执行轨迹大致是：

检测到 YouTube 链接 → 触发 youtube-to-article Skill
→ 创建 /tmp/yt2article/xxx/
→ yt-dlp 拉元信息：标题 "How to Build Agents"，作者 Anthropic，
  时长 32 分钟
→ yt-dlp 拉字幕：找到人工 en 字幕
→ clean\_vtt.py 清洗：920 行 VTT → 4800 字纯文本
→ 按 article\_template.md 改写：4 个主题章节
→ 输出到当前目录：How-to-Build-Agents-中文整理.md

整个过程 90 秒左右。yt-dlp 拉字幕通常 5 秒以内，剩下的时间花在改写上。

### 七、几个值得加的进阶能力

基础版能用之后，可以让 Agent 继续追加几个增强（同样是一段 Prompt 的事）：

**支持播放列表**：用 yt-dlp --flat-playlist 拿播放列表所有视频，循环处理。适合批量整理某个频道的内容。

**自动判断中英文输出**：检测原视频字幕语言，本身是中文的话跳过翻译只做改写。

**接入封面生成**：改写完成后调用 cover Skill 生成文章封面图，凑齐发布所需的全部资产。

**串联到 wechat-article**：把这个 Skill 作为 wechat-article 的前置步骤，输入一个 YouTube 链接直接发到公众号草稿箱。这种**子 Skill 串联**是 Skill 体系真正强大的地方。

### 结尾

把 YouTube 视频转成文章这件事，技术上没有任何难点——yt-dlp 三行命令搞定字幕，剩下交给大模型改写。真正的红利在两个层面：

第一，**把它做成 Skill**，意味着以后不用每次重新组织流程，一句话启动整条流水线。

第二，**让 Agent 来写这个 Skill**，意味着你连"写 Skill"本身的成本都免了。从想法到能用的能力，只隔一段 Prompt。

Claude Code、OpenClaw、Codex 已经是相当成熟的编码 Agent，写一个 Skill 这种小型工程任务在它们手里几乎零失误。把工具理解为生产力的杠杆——你提供的是判断和经验，它们提供的是执行和速度。

如果你也有类似的"信息加工"任务——视频转文章、播客转笔记、PDF 总结、网页摘录——挑一个出来，把上面那段 Prompt 改改扔给 Agent。三五分钟之后，你的 Skill 库里又多一个稳定能力。
