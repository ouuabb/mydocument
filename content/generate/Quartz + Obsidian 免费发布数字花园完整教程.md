> 本教程教你如何使用 GitHub 的 `template-quartz` 模板，将自己的 Obsidian 笔记免费发布成静态网站。

---

## 📖 目录

1. [项目介绍](#项目介绍)
2. [准备工作](#准备工作)
3. [创建你的仓库](#创建你的仓库)
4. [克隆到本地](#克隆到本地)
5. [迁移现有笔记](#迁移现有笔记)
6. [配置 GitHub Pages](#配置-github-pages)
7. [推送与发布](#推送与发布)
8. [可选配置](#可选配置)
9. [常见问题](#常见问题)
10. [附录：命令速查](#附录命令速查)

---

## 项目介绍

### 这是什么？

`template-quartz` 是一个专门为 Obsidian 笔记用户设计的、免费发布数字花园（Digital Garden）网站的模板。它基于 Quartz v4 工具，做了优化封装，让你能专注写笔记，一键发布成静态网站。

### 核心机制

- 使用 Obsidian 在 `content` 文件夹写笔记
- 推送到 GitHub 后，GitHub Actions 自动构建
- 发布到 GitHub Pages，免费访问

### 技术栈

| 组件 | 说明 |
|------|------|
| Quartz v4 | 静态网站生成器，支持 Obsidian 双链 |
| GitHub Pages | 免费托管服务 |
| GitHub Actions | 自动构建和部署 |

---

## 准备工作

在开始前，请确保你拥有以下账号和工具：

### 必需的账号

- [GitHub 账号](https://github.com)（免费注册）

### 必需的软件

| 软件 | 用途 | 下载地址 |
|------|------|----------|
| Git | 版本控制、推送代码 | [https://git-scm.com/downloads](https://git-scm.com/downloads) |
| Obsidian | 编写笔记 | [https://obsidian.md/](https://obsidian.md/) |

### 可选的软件

| 软件 | 用途 | 下载地址 |
|------|------|----------|
| Node.js | 本地预览网站 | [https://nodejs.org/](https://nodejs.org/)（LTS版本） |
| VS Code | 编辑配置文件 | [https://code.visualstudio.com/](https://code.visualstudio.com/) |

---

## 创建你的仓库

1. 打开模板链接：  
   [https://github.com/ouuabb/template-quartz](https://github.com/ouuabb/template-quartz)

2. 点击页面上的绿色按钮 **“Use this template”** → **“Create a new repository”**

3. 填写仓库信息：
   - **Repository name**：填一个名字，比如 `my-notes`
   - **Description**（可选）：写一段描述
   - 选择 **Public**（公开，才能免费使用 GitHub Pages）

4. 点击 **Create repository**

✅ 完成！你的仓库地址类似：`https://github.com/你的用户名/my-notes`

---

## 克隆到本地

### 第 1 步：打开终端

- Windows：使用 Git Bash 或命令提示符
- Mac/Linux：使用终端

### 第 2 步：执行克隆命令

```bash
# 进入你想放项目的目录（比如桌面）
cd Desktop

# 克隆你的仓库（注意 --recursive 很重要！）
git clone --recursive https://github.com/你的用户名/my-notes.git

# 进入项目文件夹
cd my-notes
```

> ⚠️ **注意**：`--recursive` 参数会同时下载 `quartz` 子模块。如果你忘记加这个参数，执行：
> ```bash
> git submodule update --init --recursive
> ```

### 第 3 步：理解目录结构

```
my-notes/              ← 根目录（Git 主仓库）
├── .git/              ← 主仓库的版本记录
├── content/           ← 你的笔记放在这里（用 Obsidian 打开这个文件夹）
├── quartz/            ← 子模块（框架代码，有自己的 .git）
├── quartz.config.ts   ← 网站配置文件
├── quartz.layout.ts   ← 布局配置文件
└── 其他文件...
```

---

## 迁移现有笔记

如果你已经有 Obsidian 笔记仓库，按以下步骤迁移。

### 方法一：复制粘贴（推荐）

```bash
# 1. 清空模板自带的示例内容
cd my-notes
rm -rf content/*
# Windows 用户用：del /q content\*

# 2. 复制你的笔记进去
# Mac/Linux：
cp -r /你的原笔记路径/* content/

# Windows：
xcopy D:\你的原笔记路径\* content\ /E
```

### 方法二：直接移动（适合没有版本控制需求）

直接把原 Obsidian 仓库的所有文件剪切到 `my-notes/content/` 文件夹。

### 迁移后检查

打开 Obsidian，用 **“打开仓库”** 选择 `my-notes/content` 文件夹，检查：
- ✅ 所有笔记都在
- ✅ 附件（图片等）能正常显示
- ⚠️ 特殊插件语法（Dataview、Excalidraw 等）无法在网站显示

---

## 配置 GitHub Pages

这一步让 GitHub 知道用 Actions 来构建你的网站。

1. 进入你的 GitHub 仓库页面：`https://github.com/你的用户名/my-notes`

2. 点击 **Settings**（设置）

3. 左侧菜单找到 **Pages**

4. 在 **Source** 下拉框里，选择 **GitHub Actions**

5. 不需要其他操作，设置自动保存

✅ 完成！

---

## 推送与发布

### 提交笔记到 GitHub

在 Obsidian 里写完笔记后，回到终端：

```bash
# 进入根目录（重要！）
cd my-notes

# 查看修改
git status

# 添加所有修改
git add .

# 提交（-m 后面写你的备注）
git commit -m "更新笔记"

# 推送到 GitHub
git push
```

> 📍 **注意**：推送命令必须在**根目录**执行，不是在 `content` 文件夹里。

### 等待自动构建

1. 去你的 GitHub 仓库页面
2. 点击上方的 **Actions** 选项卡
3. 你会看到一个正在运行的 workflow（如 “pages build and deployment”）
4. 等它变成绿色的 ✔️（通常 1-2 分钟）

### 访问你的网站

网站地址：`https://你的用户名.github.io/仓库名/`

例如：`https://zhangsan.github.io/my-notes/`

✅ 网站上线！以后每次 `git push` 都会自动更新。

---

## 可选配置

### 1. 修改网站名称和描述

编辑根目录的 `quartz.config.ts`：

```typescript
const config: QuartzConfig = {
  configuration: {
    pageTitle: "我的知识花园",        // 改这里
    pageTitleSuffix: "数字花园",
    description: "个人学习笔记",      // 改这里
    // ...
  }
}
```

修改后提交推送即可生效。

### 2. 设置网站图标（favicon）

1. 准备一个正方形图片（如 `logo.png`）
2. 放到 `content/` 目录下
3. 修改 `quartz.config.ts` 中的 `favicon`：
   ```typescript
   favicon: "/content/logo.png"
   ```

### 3. 创建首页

确保 `content/` 里有一个 `index.md` 作为首页：

```markdown
---
title: 欢迎
---
# 我的笔记库

这里是个人知识花园。

## 最近更新
- [[笔记1]]
- [[笔记2]]
```

### 4. 本地预览（需要 Node.js）

```bash
cd my-notes/quartz
npm i
npx quartz build -d ../content --serve
```

然后打开浏览器访问 `http://localhost:8080`

### 5. 自定义域名（可选）

1. 在 GitHub 仓库 Settings → Pages → Custom domain 填写你的域名
2. 在你的域名服务商添加 CNAME 记录，指向 `你的用户名.github.io`

### 6. 修改主题颜色

创建 `content/styles/custom.scss` 文件：

```scss
// 修改链接颜色
a {
  color: #e67e22;
}

// 修改暗色模式背景
.dark {
  background-color: #1a1a2e;
}
```

---

## 常见问题

### Q1：网站显示 404？

**原因**：GitHub Pages 未正确配置或 Actions 尚未完成。

**解决**：
1. 确认 Settings → Pages → Source 选择了 **GitHub Actions**
2. 等待 Actions 完成（约 1-2 分钟）
3. 刷新页面

### Q2：我应该在哪个目录执行 git push？

**答**：在**根目录**（`my-notes/`）执行，不是在 `content` 文件夹里。

验证方法：运行 `ls`（Mac/Linux）或 `dir`（Windows），能看到 `content` 和 `quartz` 两个文件夹。

### Q3：VSCode 显示两个 Git 仓库？

**答**：正常现象。一个是主仓库（你的笔记），一个是 `quartz` 子模块（框架代码）。提交笔记时选择主仓库即可。

### Q4：图片无法显示？

**解决**：
1. 把图片放到 `content/attachments/` 文件夹
2. 引用时使用相对路径：`![描述](attachments/图片.png)`
3. 确保图片文件名没有中文或特殊字符

### Q5：Obsidian 双链能正常显示吗？

**答**：能。Quartz 原生支持 `[[双链]]` 语法，会自动转换为正常链接。

### Q6：Dataview 查询块无法显示？

**答**：Quartz 不支持 Obsidian 插件语法。需要手动将结果写成普通 Markdown。

### Q7：如何更新 Quartz 到最新版？

```bash
cd my-notes/quartz
git pull origin v4
cd ..
git add quartz
git commit -m "更新 Quartz 子模块"
git push
```

### Q8：克隆时忘记加 `--recursive` 怎么办？

```bash
cd my-notes
git submodule update --init --recursive
```

---

## 附录：命令速查

### Git 常用命令

| 操作 | 命令 |
|------|------|
| 查看状态 | `git status` |
| 添加所有修改 | `git add .` |
| 提交 | `git commit -m "备注"` |
| 推送 | `git push` |
| 查看远程仓库 | `git remote -v` |

### 本地预览命令

```bash
cd my-notes/quartz
npm i
npx quartz build -d ../content --serve
```

### 目录导航

```bash
# 查看当前位置
pwd        # Mac/Linux
cd         # Windows

# 查看当前目录内容
ls         # Mac/Linux
dir        # Windows
```

---

## 总结

你只需要记住三件事：

1. **写笔记**：在 Obsidian 里编辑 `content/` 文件夹
2. **推送**：在 `my-notes/` 根目录执行 `git add .` → `git commit -m "xxx"` → `git push`
3. **访问**：`https://你的用户名.github.io/仓库名/`

祝你搭建顺利！🎉

---

*文档更新日期：2026年5月*