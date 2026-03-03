# 侧边栏文件夹模式 - 现状总结

## 📊 快速对比

```
┌─────────────────────────────────────────────────────────────┐
│                    Ora Browser 书签功能                      │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  现状: ⚙️ 有框架无实现 (Framework exists, no impl)          │
│                                                              │
│  ✅ 已有                      │  ❌ 缺失                    │
│  ─────────────────────────────┼───────────────────────────  │
│  • Folder 数据模型 (24行)    │  • Bookmark 数据模型       │
│  • TabContainer 关系         │  • BookmarkManager 服务    │
│  • isOpened 字段             │  • 书签 UI 组件            │
│                              │  • 侧边栏集成              │
│                              │  • 拖拽和右键菜单          │
│                              │  • 快捷键 (Cmd+D)          │
│                              │  • 书签搜索                │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

## 🗂️ 文件树现状

```
ora/
├── Models/
│   ├── Folder.swift              ✅ 存在但不完整
│   │   └─ 需要增强:
│   │      - 添加 order 字段
│   │      - 添加 emoji 字段
│   │      - 添加 children 关系 (子文件夹)
│   │      - 添加 bookmarks 关系
│   │
│   ├── Tab.swift                 ✅ 完整
│   ├── TabContainer.swift        ✅ 有 folders 字段
│   ├── History.swift             ✅ 完整
│   ├── Download.swift            ✅ 完整
│   ├── SearchEngine.swift        ✅ 完整
│   └── Keyboard.swift            ✅ 完整
│       ❌ Bookmark.swift         (需要创建)
│
├── Services/
│   ├── TabManager.swift          ✅ 标签管理
│   ├── HistoryManager.swift      ✅ 历史管理
│   ├── DownloadManager.swift     ✅ 下载管理
│   ├── SearchEngineService.swift ✅ 搜索引擎
│   ├── PrivacyService.swift      ✅ 隐私功能
│   ├── AppearanceManager.swift   ✅ 主题管理
│   └── ... 其他 20+ 服务
│       ❌ BookmarkManager.swift  (需要创建)
│
└── Modules/Sidebar/
    ├── SidebarView.swift         ✅ 侧边栏容器
    ├── ContainerView.swift       ✅ 容器内容区域
    │   └─ 当前显示:
    │      ✅ FavTabsGrid (收藏标签网格)
    │      ✅ PinnedTabsList (固定标签列表)
    │      ✅ NormalTabsList (普通标签列表)
    │      ❌ BookmarkFoldersView (书签文件夹 - 需要创建)
    │
    ├── TabList/
    │   ├── FavTabsList.swift
    │   ├── PinnedTabsList.swift
    │   └── NormalTabsList.swift
    │
    ├── BottomOption/
    │   ├── ContainerSwitcher.swift
    │   ├── ContainerForm.swift
    │   ├── NewContainerButton.swift
    │   └── EditContainerModal.swift
    │
    ├── DownloadsWidget.swift
    ├── FloatingSidebar.swift
    ├── SidebarToolbar.swift
    └── SidebarURLDisplay.swift
        ❌ Bookmarks/ (新目录，需要创建)
            ├── BookmarkFoldersView.swift
            ├── BookmarkFolderItem.swift
            ├── BookmarkList.swift
            ├── BookmarkItem.swift
            ├── BookmarkContextMenu.swift
            └── AddBookmarkModal.swift
```

## 📈 Folder 模型现状

**当前代码** (24 行):
```swift
@Model
class Folder: ObservableObject, Identifiable {
    var id: UUID
    var name: String
    var isOpened: Bool              // ✅ 用于展开/收起

    @Relationship(inverse: \TabContainer.folders)
    var container: TabContainer
}
```

**需要增强**:
```swift
@Model
class Folder: ObservableObject, Identifiable {
    var id: UUID
    var name: String
    var emoji: String?              // ❌ 新增: 文件夹表情
    var order: Int                  // ❌ 新增: 排序位置
    var isOpened: Bool              // ✅ 现有: 展开状态
    var createdAt: Date             // ❌ 新增: 创建时间

    @Relationship(deleteRule: .cascade)
    var children: [Folder]?         // ❌ 新增: 子文件夹支持

    @Relationship(deleteRule: .cascade)
    var bookmarks: [Bookmark]?      // ❌ 新增: 包含的书签

    @Relationship(inverse: \TabContainer.folders)
    var container: TabContainer
}
```

## 🏗️ 需要新建的完整架构

```
Phase 1: Models 层 (30分钟)
├── 创建 Bookmark.swift
│   └─ 属性: id, title, url, favicon, order, createdAt, folder关系
└── 增强 Folder.swift
    └─ 添加: emoji, order, children, bookmarks, createdAt

Phase 2: Services 层 (2小时)
└── 创建 BookmarkManager.swift
    ├── 文件夹操作: create, delete, update, reorder
    ├── 书签操作: create, delete, update, move
    ├── 查询方法: getFolders, getBookmarks, search
    └── 持久化: SwiftData 集成

Phase 3: UI Components 层 (4-6小时)
├── 创建 Modules/Sidebar/Bookmarks/ 目录
├── BookmarkFoldersView.swift
│   └─ 显示文件夹树形列表，支持展开/收起
├── BookmarkFolderItem.swift
│   └─ 单个文件夹项，显示子项和子文件夹
├── BookmarkItem.swift
│   └─ 单个书签项，显示 favicon 和链接
├── BookmarkContextMenu.swift
│   └─ 右键菜单: 编辑、删除、打开等
└── AddBookmarkModal.swift
    └─ 添加/编辑书签对话框

Phase 4: Integration 层 (1小时)
├── ContainerView.swift
│   └─ 侧边栏添加 BookmarkFoldersView
├── OraRoot.swift
│   └─ 初始化 BookmarkManager
└── KeyboardShortcuts.swift
    └─ 添加 Cmd+D 快捷键

Phase 5: Polish & Testing (2小时)
├── 拖拽排序功能
├── 书签搜索功能
├── 单元测试
└── 性能优化

总工作量: 18-25 小时
```

## 🔗 与现有代码的关系

### 可以直接复用

```
📋 UI 模式参考
├── TabList/*
│   └─ NormalTabsList, PinnedTabsList 提供列表样式参考
├── BottomOption/*
│   └─ 提供对话框和编辑表单的参考
└── Sidebar.swift
    └─ 侧边栏布局和配置参考

⚙️ 逻辑参考
├── TabManager.swift
│   └─ CRUD 操作、状态管理、持久化模式
├── TabDropDelegate.swift
│   └─ 拖拽排序的实现方法
└── DialogManager.swift
    └─ 对话框和模态框管理

🎨 工具函数
├── FaviconService.swift
│   └─ 复用获取网站 favicon 功能
├── Utils.swift
│   └─ 通用工具函数
└── ClipboardUtils.swift
    └─ 剪贴板操作
```

### 需要修改的现有文件

| 文件 | 修改内容 | 影响 |
|------|---------|------|
| **Models/Folder.swift** | 增强模型字段 | 🟡 中等 - 需要数据迁移 |
| **Models/TabContainer.swift** | 可能增加便利方法 | 🟢 低 |
| **Services/OraRoot.swift** | 初始化 BookmarkManager | 🟢 低 |
| **Modules/Sidebar/ContainerView.swift** | 添加书签区域 | 🟡 中等 |
| **UI/Dialog/** | 复用现有对话框 | 🟢 低 |

## 📌 路线图中的位置

根据 `ROADMAP.md`:

```
┌─────────────────────────────────────────┐
│         Personalization Section         │
├─────────────────────────────────────────┤
│                                         │
│ ☐ Bookmark management with folders     │
│   状态: 未开始 (Not started)           │
│   计划: Beta 1                         │
│   优先级: 中等                          │
│   复杂度: 中高                          │
│                                         │
└─────────────────────────────────────────┘
```

## 🎯 实现建议

### 最小可行产品 (MVP)

如果想快速上线，可以先实现基础版本：

```swift
Phase 1: 基础书签 (8小时)
├── Bookmark 模型
├── BookmarkManager CRUD
├── BookmarkList 显示
└── 侧边栏集成

Phase 2+: 高级功能 (后续)
├── 文件夹层级
├── 拖拽排序
├── 书签搜索
└── 快捷键和导入
```

### 完全实现

按照 Phase 1-5 循序渐进，最终获得完整功能。

## 📂 关键代码片段位置

```
查找 TabManager 的 CRUD 模式:
ora/Services/TabManager.swift:100-200

查找 UI 列表样式:
ora/Modules/Sidebar/TabList/NormalTabsList.swift

查找拖拽实现:
ora/Services/TabDropDelegate.swift

查找对话框系统:
ora/UI/Dialog/DialogManager.swift

查找侧边栏布局:
ora/Modules/Sidebar/SidebarView.swift:52-100
```

## ✨ 总结

```
┌──────────────────────────────────────────────┐
│     Ora Browser 书签功能实现进度             │
├──────────────────────────────────────────────┤
│                                              │
│  路线图需求:  📌 Bookmark management        │
│  当前进度:    ⚙️  有框架 (Folder model)     │
│  完成度:      🟢 5% (仅数据模型框架)        │
│                                              │
│  建议方案:    👉 按 5-Phase 循序实现        │
│  工作量:      📊 18-25 小时                 │
│  难度:        ⭐⭐⭐ 中等                   │
│                                              │
│  详见: BOOKMARK_FEATURE_ANALYSIS.md          │
│                                              │
└──────────────────────────────────────────────┘
```
