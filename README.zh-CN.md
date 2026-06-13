# claude-meeting-slides

[English](README.md) | **简体中文**

一个 [Claude Code](https://claude.ai/code) 技能，从项目代码和工作进度直接生成横版 LaTeX PDF 幻灯片 —— 用于**半正式技术汇报**（组会、周报、内部评审）。

## 适合什么场合

✅ **合适** —— 组会、周报、月报、内部技术评审、代码讲解、给团队同步项目进度
❌ **不合适** —— 学术会议正式 talk、答辩、客户对外宣讲。这些场合请手搓 Beamer / Keynote / PowerPoint

甜点区是"**技术准确性 + 速度 > 视觉精美**"。Claude Code 直接读你的源代码，生成贴近实现细节的幻灯片：TikZ 架构图、状态机、参数表，还能嵌入运行结果截图来清晰展示流程和成果。

## 示例输出

3 页横版 PDF 演示（已脱敏，所有名称/产品/参数均为占位）：

| 第 1 页：系统架构 | 第 2 页：节点集成 | 第 3 页：算法复盘 |
|:---:|:---:|:---:|
| ![第 1 页](examples/crab-inspection/page-1.png) | ![第 2 页](examples/crab-inspection/page-2.png) | ![第 3 页](examples/crab-inspection/page-3.png) |

> 完整示例在 [`examples/crab-inspection/`](examples/crab-inspection/) —— 3 页 PDF + LaTeX 源码 + 讲稿。

## 快速开始

### 1. 安装依赖

```bash
# Ubuntu/Debian/WSL
sudo apt install texlive-lang-chinese texlive-latex-extra texlive-fonts-recommended

# macOS (Homebrew)
brew install --cask mactex

# Windows：用 WSL 跑上面的 Ubuntu 命令（推荐），或装 MiKTeX + ctex 包
```

### 2. 把技能加到你的项目

```bash
# 方式 A：复制到你的项目
cp -r .claude/ /path/to/your/project/.claude/

# 方式 B：克隆后软链
cd /path/to/your/project
ln -s /path/to/claude-meeting-slides/.claude .claude
```

### 3. 使用

在你的项目目录里打开 Claude Code，输入：

```
/meeting-slides
```

Claude 会问你汇报周期、主要工作、页数等，然后生成完整的 LaTeX PDF。

## 工作原理

技能遵循 5 步工作流：

1. **信息收集** —— 读你的进度表/截图，浏览源代码确认技术细节
2. **页面规划** —— 提出每页大纲，**写 LaTeX 前先跟你确认**
3. **写 LaTeX** —— 用内置模板生成横版 PDF，含 TikZ 图
4. **编译迭代** —— 用 `xelatex` 编译，按你的反馈调整
5. **讲稿** —— 为每页生成口述讲稿

## 项目结构

```
claude-meeting-slides/
├── .claude/commands/
│   └── meeting-slides.md      ← Claude Code 技能定义
├── templates/
│   └── landscape-tikz.tex     ← 独立 LaTeX 模板（用来测试环境）
├── examples/
│   └── crab-inspection/       ← 完整示例输出
├── README.md                   ← 英文文档
├── README.zh-CN.md             ← 中文文档
├── LICENSE                     ← MIT
└── .gitignore
```

## 测试你的环境

编译独立模板，验证 LaTeX 能跑通：

```bash
cd templates && xelatex landscape-tikz.tex
```

能产出 2 页 PDF 就 OK。

## 自定义

**颜色**：编辑模板或技能文件里的 `\definecolor` 部分。

**项目特定引用**：在 `.claude/commands/meeting-slides.md` 末尾加上你已有的报告/图/样式指南的路径，让 Claude 知道去哪找。

**输出目录**：默认输出到 `reports/meeting_YYYY_M_D/`，改技能文件即可。

## 依赖

- [Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code)
- TeX Live with `xelatex` + `ctex` 包（用于中文支持）
- 一个有源代码的项目让 Claude Code 读

## License

MIT
