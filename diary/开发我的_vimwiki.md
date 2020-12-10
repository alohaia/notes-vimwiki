# 开发我的 vimwiki

## Features

- 多级标签，标签值
- TODOs management
- 生词管理，支持查看出现位置
- 原生 markdown 语法（有些功能用注释实现）
- 中文标点符号支持
- 内部链接管理、跳转
- 从其他笔记中通过标题、特殊注释引用文本

## 开发思路

- NeoVim Lua API
- ~~[MySQL](../MySQL.md)~~ [MariaDB](../MariaDB.md) + Lua( + C&C++)
- 只占用一个缓冲区
