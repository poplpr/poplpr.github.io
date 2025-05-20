---
layout: wiki
title: latex
cate1: Tools
cate2: Editor
description: LaTeX 从安装到部分指令使用。
keywords: LaTeX
---

最新版的 Texlive 中的 xelatex 自动支持中文，不用再去费劲地装 CTeX 啦！

# LaTeX (texlive) 安装

从清华源找到 texlive 的 [image](https://mirrors.tuna.tsinghua.edu.cn/CTAN/systems/texlive/Images/)。

打开 `.iso` 后缀的安装包后，打开 `install-tl-windows.bat` 文件。看网上说大概要安装一个小时，我从 22:25 分开始安装，看看需要安装多久。23:11 分安装成功，一共安装了 46 分钟。

再在 VsCode 安装插件 Latex Workshop，安装后在设置中打开 settings.json，加入如下内容：

```json
{
    "latex-workshop.latex.tools": [
        {
            "name": "latexmk",
            "command": "latexmk",
            "args": [
                "-synctex=1",
                "-interaction=nonstopmode",
                "-file-line-error",
                "-xelatex",
                "-outdir=%OUTDIR%",
                "-cd",
                "%DOC%"
            ]
        },
        {
            "name": "xelatex",
            "command": "xelatex",
            "args": [
                "-synctex=1",
                "-interaction=nonstopmode",
                "-file-line-error",
                "-pdf",
                "-outdir=%OUTDIR%",
                "-cd",
                "%DOC%"
            ]
        },
        {
            "name": "biber",
            "command": "biber",
            "args": [
                "%DOCFILE%"
            ]
        },
        {
            "name": "pdflatex",
            "command": "pdflatex",
            "args": [
                "-synctex=1",
                "-interaction=nonstopmode",
                "-file-line-error",
                "%DOC%"
            ]
        },
        {
            "name": "bibtex",
            "command": "bibtex",
            "args": [
                "%DOCFILE%"
            ]
        }
    ],
    "latex-workshop.latex.recipes": [
        {
            "name": "latexmk",
            "tools": [
                "latexmk"
            ]
        },
        {
            "name": "xelatex",
            "tools": [
                "xelatex"
            ]
        },
        {
            "name": "pdflatex -> bibtex -> pdflatex*2",
            "tools": [
                "pdflatex",
                "bibtex",
                "pdflatex",
                "pdflatex"
            ]
        },
        {
            "name": "xelatex -> biber -> xelatex * 2",
            "tools": [
                "xelatex",
                "biber",
                "xelatex",
                "xelatex"
            ]
        }
    ],
    "latex-workshop.latex.clean.fileTypes": [
        "*.aux",
        "*.bbl",
        "*.blg",
        "*.idx",
        "*.ind",
        "*.lof",
        "*.lot",
        "*.out",
        "*.toc",
        "*.acn",
        "*.acr",
        "*.alg",
        "*.glg",
        "*.glo",
        "*.gls",
        "*.ist",
        "*.fls",
        "*.log",
        "*.fdb_latexmk"
    ]
}
```

然后北理工的毕业论文模板使用 `latexmk` 编译或者 `xelatex -> biber -> xelatex * 2` 编译都可以编译成功。

注意：接下来需要重启一下，否则从图标/开始菜单打开的 VsCode 没办法正常读取环境变量，但是不知道为什么从终端打开的 VsCode 可以。