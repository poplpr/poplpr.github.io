---
layout: post
title: Neovim 快速复制粘贴的方法
categories: [neovim, vim]
description: Neovim 快速复制粘贴的方法
keywords: neovim, vim
mermaid: false
---

## 快速复制粘贴

具体操作是在 `init.lua` 文件中加入下面这一行：

```lua
vim.opt.clipboard = 'unnamedplus'
```

或者是在 `init.vim` 文件中加入：

```vim
set clipboard+=unnamedplus
```

我的环境是 Windows11 + neovim，那么它的路径为 `C:\Users\<username>\AppData\Local\nvim\init.lua`。

相应的 linux 路径为 `~/.config/nvim/init.lua`（没有测试）

然后在 nvim 中的所有复制/剪切操作就会和系统的剪切板连接起来了。

## 其他

有 Telescope 插件可以查看所有寄存器内容。

## 复制时高亮

在配置文件中加入：

```lua
vim.api.nvim_create_autocmd('TextYankPost', {
  callback = function()
    vim.highlight.on_yank { higroup = 'IncSearch', timeout = 200 }
  end,
})
```

不过好像必须要 `higroup = 'IncSearch'` 才对 vscode neovim 生效，不知道为什么。

## 全部复制

```vim
:%+y<Enter>
```

