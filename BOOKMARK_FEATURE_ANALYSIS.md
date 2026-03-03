# Ora Browser - 书签和文件夹功能分析

## 📊 现状总结

### 路线图需求
根据 `ROADMAP.md`，**Bookmark management with folders** 被列为 Beta 1 的 **Personalization** 功能，目前状态为 ❌ **未实现**

### 现有代码状态

#### ✅ 已实现的部分

##### 1. 数据模型已定义 (24 行)
**文件**: `ora/Models/Folder.swift`

```swift
@Model
class Folder: ObservableObject, Identifiable {
    var id: UUID
    var name: String
    var isOpened: Bool  // 用于展开/收起状态

    @Relationship(inverse: \TabContainer.folders)
    var container: TabContainer
}
```

**分析**:
- ✅ 模型类已定义为 `@Model`（SwiftData 支持持久化）
- ✅ 与 TabContainer 建立了关系（inverse relationship）
- ✅ 有 `isOpened` 字段用于 UI 折叠状态
- ⚠️ **缺失**:
  - 没有子文件夹支持（父文件夹层级关系）
  - 没有排序字段（order）
  - 没有书签项（Bookmark）模型

##### 2. TabContainer 关联了 Folder
**文件**: `ora/Models/TabContainer.swift` (第 15 行)

```swift
@Relationship(deleteRule: .cascade) var folders: [Folder] = []
```

**分析**:
- ✅ 每个容器可以包含多个文件夹
- ✅ 级联删除规则（删除容器时自动删除所有文件夹）
- ⚠️ **缺失**: 没有书签项（Bookmark）的存储

---

#### ❌ 未实现的部分

##### 1. 缺少 Bookmark 数据模型
目前完全没有书签项（Bookmark）的定义：
- 没有 `Bookmark.swift` 文件
- 没有 `@Model class Bookmark` 定义
- 无法存储单个书签

##### 2. 缺少书签 UI 组件
搜索结果表明：
- 没有 `BookmarkView.swift`
- 没有 `FolderView.swift`
- 没有 `BookmarkManager.swift` 服务
- 侧边栏中没有任何书签/文件夹相关的 UI 代码

##### 3. 缺少业务逻辑服务
没有任何服务处理：
- 书签的 CRUD 操作
- 文件夹的 CRUD 操作
- 书签与文件夹的关联管理
- 拖拽排序等交互

##### 4. 侧边栏没有集成
**文件**: `ora/Modules/Sidebar/ContainerView.swift`

当前侧边栏只显示：
- ✅ 收藏标签（FavTabsList）
- ✅ 固定标签（PinnedTabsList）
- ✅ 普通标签（NormalTabsList）
- ❌ **没有书签文件夹区域**

---

## 🏗️ 完整实现所需的代码架构

### 1. 数据模型层 (Models/)

需要创建以下模型：

```swift
// Folder.swift - 需要增强
@Model
class Folder: ObservableObject, Identifiable {
    var id: UUID
    var name: String
    var emoji: String?           // [新增] 文件夹表情标记
    var order: Int               // [新增] 排序位置
    var isOpened: Bool
    var createdAt: Date          // [新增]

    // 文件夹层级关系
    @Relationship(deleteRule: .cascade)
    var children: [Folder]?      // [新增] 子文件夹

    @Relationship(deleteRule: .cascade)
    var bookmarks: [Bookmark]    // [新增] 包含的书签

    @Relationship(inverse: \TabContainer.folders)
    var container: TabContainer
}
```

```swift
// Bookmark.swift - [新增] 书签模型
@Model
class Bookmark: ObservableObject, Identifiable {
    var id: UUID
    var title: String
    var url: URL
    var favicon: URL?            // 网站图标
    var faviconLocalFile: URL?   // 本地缓存的图标
    var order: Int               // 排序位置
    var createdAt: Date
    var lastAccessedAt: Date?    // 最后访问时间

    @Relationship(inverse: \Folder.bookmarks)
    var folder: Folder

    init(
        id: UUID = UUID(),
        title: String,
        url: URL,
        favicon: URL? = nil,
        folder: Folder,
        order: Int
    ) {
        self.id = id
        self.title = title
        self.url = url
        self.favicon = favicon
        self.folder = folder
        self.order = order
        self.createdAt = Date()
    }
}
```

### 2. 服务层 (Services/)

需要创建 `BookmarkManager.swift`:

```swift
@MainActor
class BookmarkManager: ObservableObject {
    @Published var folders: [Folder] = []
    @Published var selectedFolder: Folder?

    private let modelContext: ModelContext

    // CRUD 操作
    func createFolder(name: String, container: TabContainer) -> Folder { }
    func deleteFolder(_ folder: Folder) { }
    func updateFolder(_ folder: Folder, name: String) { }
    func reorderFolders(_ folders: [Folder]) { }

    func createBookmark(title: String, url: URL, in folder: Folder) { }
    func deleteBookmark(_ bookmark: Bookmark) { }
    func updateBookmark(_ bookmark: Bookmark, title: String, url: URL) { }
    func moveBookmark(_ bookmark: Bookmark, to folder: Folder) { }

    // 工具方法
    func getFoldersForContainer(_ container: TabContainer) -> [Folder] { }
    func getBookmarksForFolder(_ folder: Folder) -> [Bookmark] { }
    func searchBookmarks(query: String) -> [Bookmark] { }
}
```

### 3. UI 组件层 (Modules/Sidebar/)

需要创建以下组件文件：

```
Modules/Sidebar/
├── Bookmarks/                [新增]
│   ├── BookmarkFoldersView.swift      # 文件夹树形视图
│   ├── BookmarkFolderItem.swift       # 单个文件夹项
│   ├── BookmarkList.swift            # 文件夹内的书签列表
│   ├── BookmarkItem.swift            # 单个书签项
│   ├── BookmarkContextMenu.swift     # 右键菜单
│   └── AddBookmarkModal.swift        # 添加书签对话框
└── ...其他现有组件
```

#### 关键组件伪代码

```swift
// BookmarkFoldersView.swift
struct BookmarkFoldersView: View {
    @EnvironmentObject var bookmarkManager: BookmarkManager
    @EnvironmentObject var tabManager: TabManager

    var body: some View {
        VStack(alignment: .leading) {
            HStack {
                Label("Bookmarks", systemImage: "bookmark.fill")
                    .font(.subheadline)
                    .fontWeight(.semibold)
                Spacer()
                // 添加书签按钮
                Button(action: { /* 添加书签 */ }) {
                    Image(systemName: "plus")
                        .font(.caption)
                }
            }
            .padding(.horizontal, 10)

            ScrollView {
                ForEach(bookmarkManager.folders) { folder in
                    BookmarkFolderItem(folder: folder)
                        .contextMenu {
                            // 编辑、删除、新建子文件夹
                        }
                }
            }
        }
    }
}

// BookmarkFolderItem.swift
struct BookmarkFolderItem: View {
    @State var isExpanded: Bool = false
    let folder: Folder

    var body: some View {
        DisclosureGroup(isExpanded: $isExpanded) {
            ForEach(folder.bookmarks ?? []) { bookmark in
                BookmarkItem(bookmark: bookmark)
                    .padding(.leading, 16)
            }
            if !folder.children?.isEmpty ?? true {
                ForEach(folder.children ?? []) { subfolder in
                    BookmarkFolderItem(folder: subfolder)
                        .padding(.leading, 16)
                }
            }
        } label: {
            HStack {
                Image(systemName: "folder.fill")
                    .foregroundColor(.orange)
                Text(folder.emoji ?? "📁")
                Text(folder.name)
                    .lineLimit(1)
            }
        }
    }
}

// BookmarkItem.swift - 单个书签显示
struct BookmarkItem: View {
    let bookmark: Bookmark
    @EnvironmentObject var tabManager: TabManager

    var body: some View {
        Button(action: {
            // 打开书签链接
            tabManager.openURL(bookmark.url)
        }) {
            HStack {
                // 显示网站 favicon
                if let faviconLocalFile = bookmark.faviconLocalFile,
                   let image = NSImage(contentsOf: faviconLocalFile) {
                    Image(nsImage: image)
                        .resizable()
                        .frame(width: 16, height: 16)
                } else {
                    Image(systemName: "link")
                        .font(.caption)
                }

                Text(bookmark.title)
                    .lineLimit(1)
                    .font(.caption)

                Spacer()

                Text(bookmark.url.host ?? "")
                    .font(.caption2)
                    .foregroundColor(.secondary)
                    .lineLimit(1)
            }
            .contextMenu {
                Button("Open") { /* ... */ }
                Button("Open in New Tab") { /* ... */ }
                Button("Copy Link") { /* ... */ }
                Button("Edit") { /* ... */ }
                Button("Delete", role: .destructive) { /* ... */ }
            }
        }
    }
}
```

### 4. 侧边栏集成

修改 `Modules/Sidebar/ContainerView.swift`:

```swift
struct ContainerView: View {
    // ... 现有代码 ...

    var body: some View {
        VStack(alignment: .leading, spacing: 16) {
            // ... 现有的 URL Display, Fav Tabs, Pinned Tabs ...

            ScrollView(.vertical, showsIndicators: false) {
                VStack(alignment: .leading, spacing: 16) {
                    // ... 现有的标签列表 ...

                    Divider()

                    // [新增] 书签文件夹区域
                    BookmarkFoldersView()
                        .environmentObject(bookmarkManager)
                }
            }
        }
    }
}
```

---

## 🎯 实现优先级和难度

| 优先级 | 任务 | 难度 | 预计工作量 |
|--------|------|------|----------|
| 1️⃣ | 创建 Bookmark 模型 | ⭐ 低 | 30分钟 |
| 2️⃣ | 增强 Folder 模型 | ⭐ 低 | 20分钟 |
| 3️⃣ | 创建 BookmarkManager 服务 | ⭐⭐ 中 | 2小时 |
| 4️⃣ | 创建书签 UI 组件 | ⭐⭐⭐ 高 | 4-6小时 |
| 5️⃣ | 侧边栏集成 | ⭐⭐ 中 | 1小时 |
| 6️⃣ | 拖拽排序和上下文菜单 | ⭐⭐⭐ 高 | 2-3小时 |
| 7️⃣ | 书签导入（Chrome/Safari） | ⭐⭐⭐⭐ 很高 | 4-6小时 |
| 8️⃣ | 测试和调试 | ⭐⭐ 中 | 2小时 |

**总工作量**: 大约 18-25 小时

---

## 🔗 与现有功能的关系

### 可以复用的代码

```swift
// 1. TabManager 中的标签操作模式
// 可参考 TabManager 实现类似的书签操作

// 2. 拖拽系统
// 参考 TabDropDelegate, SectionDropDelegate 的拖拽实现

// 3. 右键菜单和对话框
// 参考 DialogManager, 已有对话框系统

// 4. FaviconService
// 直接复用获取网站 favicon 的功能

// 5. 侧边栏列表样式
// 参考 NormalTabsList, PinnedTabsList 的列表UI模式
```

### 需要修改的现有文件

```
1. Models/
   ✏️ Folder.swift - 增强模型
   ✏️ TabContainer.swift - 可能需要增加便利方法

2. Services/
   ✏️ OraRoot.swift - 初始化 BookmarkManager

3. Modules/Sidebar/
   ✏️ ContainerView.swift - 添加书签显示区域
   ✏️ SidebarView.swift - 可能需要调整布局
```

---

## 💡 设计建议

### 1. 文件夹结构设计
- **支持无限层级**: 使用递归关系允许多级文件夹
- **显示深度**: 建议只显示 3-4 级深度，避免过度嵌套
- **快速访问**: 顶级文件夹默认展开，子文件夹默认收起

### 2. 书签添加方式

```swift
// 方式 1: 从浮动按钮添加
FloatingAddBookmarkButton()

// 方式 2: 右键菜单
// 在任何标签页上右键 → "Add to Bookmarks"

// 方式 3: 快捷键
Cmd + D  // 类似 Safari 和 Chrome

// 方式 4: 拖拽标签到书签区域
// 支持拖拽标签到文件夹快速添加
```

### 3. 性能优化
- 使用 `@Query` 只加载当前容器的书签
- 实现虚拟列表（如果书签超过 1000 个）
- 缓存 favicon 到本地文件系统

### 4. 搜索功能
```swift
// 在设置中或侧边栏顶部添加搜索框
@State var searchText: String = ""

var filteredBookmarks: [Bookmark] {
    if searchText.isEmpty {
        return allBookmarks
    }
    return allBookmarks.filter {
        $0.title.localizedCaseInsensitiveContains(searchText) ||
        $0.url.absoluteString.localizedCaseInsensitiveContains(searchText)
    }
}
```

---

## 📋 实现检查清单

如果你想实现这个功能，按以下顺序进行：

- [ ] **Phase 1: 数据模型**
  - [ ] 创建 `ora/Models/Bookmark.swift`
  - [ ] 增强 `ora/Models/Folder.swift` (添加 children、order、emoji)
  - [ ] 修改 `TabContainer.swift` 验证关系

- [ ] **Phase 2: 业务逻辑**
  - [ ] 创建 `ora/Services/BookmarkManager.swift`
  - [ ] 实现 CRUD 操作
  - [ ] 实现拖拽排序逻辑

- [ ] **Phase 3: UI 组件**
  - [ ] 创建 `Modules/Sidebar/Bookmarks/` 目录
  - [ ] 实现 `BookmarkFoldersView.swift`
  - [ ] 实现 `BookmarkFolderItem.swift`
  - [ ] 实现 `BookmarkItem.swift`
  - [ ] 实现右键菜单和编辑对话框

- [ ] **Phase 4: 集成**
  - [ ] 修改 `Modules/Sidebar/ContainerView.swift`
  - [ ] 在 `OraRoot.swift` 初始化 `BookmarkManager`
  - [ ] 添加快捷键 (Cmd + D)

- [ ] **Phase 5: 完善**
  - [ ] 添加搜索功能
  - [ ] 实现拖拽排序
  - [ ] 添加书签导入功能
  - [ ] 编写测试

---

## 🎓 参考资源

### 代码参考位置
- **标签管理逻辑**: `Services/TabManager.swift` (462 行) - 学习 CRUD 和状态管理
- **UI 列表组件**: `Modules/Sidebar/TabList/*.swift` - 学习列表样式
- **拖拽实现**: `Services/TabDropDelegate.swift` - 学习拖拽逻辑
- **对话框系统**: `UI/Dialog/` - 学习模态框实现

### 相关功能实现中的对标
- **类似功能**: Safari 书签侧边栏、Chrome 书签管理器
- **现有参考**: Ora 中的标签管理和容器系统

---

**总结**: 📌 **有框架无实现**

当前代码提供了：
✅ 数据模型基础框架 (Folder 定义)
✅ 数据库关系设置 (TabContainer.folders)
❌ 完整的 Bookmark 模型
❌ 业务逻辑服务
❌ UI 组件和交互

**建议**: 按照上述 Phase 1-5 的顺序实现，可以完整地为 Ora Browser 添加专业的书签管理功能。
