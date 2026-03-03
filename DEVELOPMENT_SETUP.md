# 🚀 开发设置指南

## 📌 重要事项

这个项目现在有两种分支类型：

### 1. Main 分支（文档分支）
**用途**：包含项目架构和功能分析文档
**内容**：
- PROJECT_ARCHITECTURE.md
- BOOKMARK_***.md 系列文档
- DOCUMENTATION_GUIDE.md

这些文件在 main 分支上，作为项目文档被保留。

### 2. 开发分支（feature 分支）
**用途**：用于实现新功能
**规则**：
- 从 main 创建你的开发分支（如：`feature/bookmark-management`）
- 在开发分支上实现功能，**不包含** MD 文档文件
- 提 PR 时只包含代码改动，不包含 MD 文件

## ⚠️ 关键：MD 文件不会进入你的 PR

由于这些 MD 文件已经在 main 分支上了，当你：

```bash
# 创建新的开发分支
git checkout -b feature/bookmark-management

# 开发和提交你的代码
git add <你的代码文件>
git commit -m "feat: implement bookmark feature"

# 推送分支
git push origin feature/bookmark-management

# 提 PR：main <- feature/bookmark-management
```

**这时 PR 里只会有你新增的代码文件，不会包含已经在 main 分支的 MD 文件。**

## 📋 本地开发流程

### Step 1: 创建开发分支
```bash
# 确保在 main 分支上
git checkout main

# 创建新的开发分支
git checkout -b feature/bookmark-management
```

### Step 2: 开发功能
参考 `BOOKMARK_IMPLEMENTATION_GUIDE.md` 的 Step 1-5 实现功能

```bash
# 只提交你的代码文件
git add ora/Models/Bookmark.swift
git add ora/Services/BookmarkManager.swift
git add ora/Modules/Sidebar/Bookmarks/
git commit -m "feat: add bookmark management feature"
```

### Step 3: 推送和提 PR
```bash
git push origin feature/bookmark-management
```

然后在 GitHub 上：
1. 打开你的 fork
2. 点击 "Compare & pull request"
3. 确保是：`micsama/ora-browser:main <- your-fork:feature/bookmark-management`
4. 写好描述（参考 CONTRIBUTING.md）
5. 提交 PR

## ✨ 为什么这样做？

1. **文档独立**：MD 文件在 main 分支上保持稳定
2. **PR 清晰**：你的 PR 只包含功能代码，易于审查
3. **无冲突**：其他贡献者不会因为文档修改而冲突
4. **易维护**：文档和代码分开管理

## 🔍 查看你的 PR 会包含什么

在提 PR 前，可以在本地检查：

```bash
# 查看当前分支相对于 main 的改动
git diff main...HEAD --name-only

# 这应该只显示你的代码文件，不包含 MD 文件
```

## 📚 参考文档

- 项目架构：`PROJECT_ARCHITECTURE.md`
- 书签功能实现：`BOOKMARK_IMPLEMENTATION_GUIDE.md`
- 文档导航：`DOCUMENTATION_GUIDE.md`
- 贡献指南：`CONTRIBUTING.md`

## 💡 常见问题

**Q: 我的开发分支里能不能有 MD 文件？**
A: 可以！只要最终 PR 里不包含就行。Git 会自动处理。

**Q: 如果我修改了 MD 文件怎么办？**
A: 在提 PR 前，用 `git checkout main -- <filename>` 恢复它们

**Q: 如何同步上游的更新？**
A: 
```bash
git fetch upstream
git rebase upstream/main
git push -f origin your-branch
```

---

**祝开发顺利！** 🎉
