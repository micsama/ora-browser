# Ora Browser 书签功能 - 完整实现指南

> 根据实际代码分析和路线图，这是一份关于 Ora Browser 侧边栏文件夹/书签功能的完整实现指南。

## 📋 一句话总结

**现状**: Folder 数据模型已定义，但缺少 Bookmark 模型、BookmarkManager 服务、UI 组件，以及侧边栏集成。需要 18-25 小时完整实现。

---

## 🔍 现有代码分析

### ✅ 已有代码

#### 1. Folder 数据模型 (24 行)
```swift
// ora/Models/Folder.swift
@Model
class Folder: ObservableObject, Identifiable {
    var id: UUID
    var name: String
    var isOpened: Bool  // 用于展开/收起
    
    @Relationship(inverse: \TabContainer.folders)
    var container: TabContainer
}
```

**特点**:
- ✅ 标记为 `@Model` - 支持 SwiftData 持久化
- ✅ 有 `isOpened` 字段 - UI 状态管理已考虑
- ✅ 与 TabContainer 建立了关系 - 可以获取所属容器

#### 2. TabContainer 中的 Folder 关系
```swift
// ora/Models/TabContainer.swift - 第 15 行
@Relationship(deleteRule: .cascade) var folders: [Folder] = []
```

**特点**:
- ✅ 级联删除 - 删除容器时自动清理文件夹
- ✅ 关系完整 - 每个容器可以有多个文件夹

### ❌ 缺失代码

#### 1. Bookmark 模型
```swift
// ora/Models/Bookmark.swift - [不存在]
@Model
class Bookmark: ObservableObject, Identifiable {
    var id: UUID
    var title: String
    var url: URL
    var favicon: URL?
    var order: Int
    var createdAt: Date
    
    @Relationship(inverse: \Folder.bookmarks)
    var folder: Folder
}
```

#### 2. BookmarkManager 服务
```swift
// ora/Services/BookmarkManager.swift - [不存在]
@MainActor
class BookmarkManager: ObservableObject {
    // CRUD 操作、搜索、排序等
}
```

#### 3. 书签 UI 组件
```
Modules/Sidebar/Bookmarks/ - [整个目录不存在]
  ├── BookmarkFoldersView.swift
  ├── BookmarkFolderItem.swift
  ├── BookmarkItem.swift
  └── ...
```

---

## 🎯 完整实现步骤

### Step 1: 创建 Bookmark 数据模型 (30 分钟)

**新建文件**: `ora/Models/Bookmark.swift`

```swift
import Foundation
import SwiftData

@Model
class Bookmark: ObservableObject, Identifiable {
    var id: UUID
    var title: String
    var url: URL
    var urlString: String  // 存储 URL 字符串便于查询
    var favicon: URL?
    var faviconLocalFile: URL?
    var order: Int
    var createdAt: Date
    var lastAccessedAt: Date?
    
    @Relationship(inverse: \Folder.bookmarks)
    var folder: Folder
    
    init(
        id: UUID = UUID(),
        title: String,
        url: URL,
        folder: Folder,
        order: Int
    ) {
        self.id = id
        self.title = title
        self.url = url
        self.urlString = url.absoluteString
        self.folder = folder
        self.order = order
        self.createdAt = Date()
    }
}
```

---

### Step 2: 增强 Folder 模型 (20 分钟)

**修改文件**: `ora/Models/Folder.swift`

```swift
import Foundation
import SwiftData

@Model
class Folder: ObservableObject, Identifiable {
    var id: UUID
    var name: String
    var emoji: String = "📁"           // 新增
    var order: Int = 0                 // 新增：用于排序
    var isOpened: Bool = false         // 现有
    var createdAt: Date = Date()       // 新增
    
    // 文件夹层级关系（新增）
    @Relationship(deleteRule: .cascade)
    var children: [Folder]?
    
    // 包含的书签（新增）
    @Relationship(deleteRule: .cascade)
    var bookmarks: [Bookmark]?
    
    // 所属容器（现有）
    @Relationship(inverse: \TabContainer.folders)
    var container: TabContainer
    
    init(
        id: UUID = UUID(),
        name: String,
        container: TabContainer,
        emoji: String = "📁",
        order: Int = 0,
        isOpened: Bool = false
    ) {
        self.id = id
        self.name = name
        self.emoji = emoji
        self.order = order
        self.isOpened = isOpened
        self.container = container
        self.children = []
        self.bookmarks = []
        self.createdAt = Date()
    }
}
```

---

### Step 3: 创建 BookmarkManager 服务 (2 小时)

**新建文件**: `ora/Services/BookmarkManager.swift`

```swift
import Foundation
import SwiftData
import Combine

@MainActor
class BookmarkManager: ObservableObject {
    @Published var bookmarks: [Bookmark] = []
    @Published var folders: [Folder] = []
    @Published var selectedFolder: Folder?
    
    private let modelContext: ModelContext
    
    init(modelContext: ModelContext) {
        self.modelContext = modelContext
        loadFolders()
    }
    
    // MARK: - Folder 操作
    
    func createFolder(
        name: String,
        emoji: String = "📁",
        in container: TabContainer,
        parentFolder: Folder? = nil
    ) -> Folder {
        let newFolder = Folder(
            name: name,
            container: container,
            emoji: emoji
        )
        
        if let parent = parentFolder {
            parent.children?.append(newFolder)
        } else {
            container.folders.append(newFolder)
        }
        
        modelContext.insert(newFolder)
        selectedFolder = newFolder
        loadFolders()
        return newFolder
    }
    
    func deleteFolder(_ folder: Folder) {
        modelContext.delete(folder)
        if selectedFolder?.id == folder.id {
            selectedFolder = nil
        }
        loadFolders()
    }
    
    func updateFolder(_ folder: Folder, name: String, emoji: String) {
        folder.name = name
        folder.emoji = emoji
        loadFolders()
    }
    
    func reorderFolders(_ sortedFolders: [Folder]) {
        for (index, folder) in sortedFolders.enumerated() {
            folder.order = index
        }
        loadFolders()
    }
    
    // MARK: - Bookmark 操作
    
    func createBookmark(
        title: String,
        url: URL,
        in folder: Folder
    ) -> Bookmark {
        let newBookmark = Bookmark(
            title: title,
            url: url,
            folder: folder,
            order: (folder.bookmarks?.count ?? 0)
        )
        
        folder.bookmarks?.append(newBookmark)
        modelContext.insert(newBookmark)
        loadBookmarks(for: folder)
        return newBookmark
    }
    
    func deleteBookmark(_ bookmark: Bookmark) {
        modelContext.delete(bookmark)
        loadBookmarks(for: bookmark.folder)
    }
    
    func updateBookmark(
        _ bookmark: Bookmark,
        title: String,
        url: URL
    ) {
        bookmark.title = title
        bookmark.url = url
        bookmark.urlString = url.absoluteString
        bookmark.lastAccessedAt = Date()
    }
    
    func moveBookmark(_ bookmark: Bookmark, to folder: Folder) {
        bookmark.folder.bookmarks?.removeAll { $0.id == bookmark.id }
        bookmark.folder = folder
        folder.bookmarks?.append(bookmark)
        loadBookmarks(for: folder)
    }
    
    func reorderBookmarks(_ sortedBookmarks: [Bookmark]) {
        for (index, bookmark) in sortedBookmarks.enumerated() {
            bookmark.order = index
        }
    }
    
    // MARK: - 查询操作
    
    func loadFolders() {
        // 刷新文件夹列表
        if let selected = selectedFolder {
            bookmarks = (selected.bookmarks ?? [])
                .sorted { $0.order < $1.order }
        }
    }
    
    func loadBookmarks(for folder: Folder) {
        bookmarks = (folder.bookmarks ?? [])
            .sorted { $0.order < $1.order }
    }
    
    func searchBookmarks(
        query: String,
        in folder: Folder
    ) -> [Bookmark] {
        return (folder.bookmarks ?? []).filter {
            $0.title.localizedCaseInsensitiveContains(query) ||
            $0.url.absoluteString.localizedCaseInsensitiveContains(query)
        }
    }
    
    func getChildFolders(_ folder: Folder) -> [Folder] {
        return (folder.children ?? []).sorted { $0.order < $1.order }
    }
}
```

---

### Step 4: 创建 UI 组件 (4-6 小时)

#### 4.1 创建 Bookmarks 目录和文件

```bash
mkdir -p ora/Modules/Sidebar/Bookmarks
touch ora/Modules/Sidebar/Bookmarks/BookmarkFoldersView.swift
touch ora/Modules/Sidebar/Bookmarks/BookmarkFolderItem.swift
touch ora/Modules/Sidebar/Bookmarks/BookmarkItem.swift
touch ora/Modules/Sidebar/Bookmarks/BookmarkContextMenu.swift
touch ora/Modules/Sidebar/Bookmarks/AddBookmarkModal.swift
```

#### 4.2 BookmarkFoldersView.swift

```swift
import SwiftUI

struct BookmarkFoldersView: View {
    @EnvironmentObject var bookmarkManager: BookmarkManager
    @EnvironmentObject var tabManager: TabManager
    @State private var showAddFolderSheet = false
    @State private var newFolderName = ""
    
    let container: TabContainer
    
    var body: some View {
        VStack(alignment: .leading, spacing: 8) {
            // 标题栏
            HStack {
                Label("Bookmarks", systemImage: "bookmark.fill")
                    .font(.subheadline)
                    .fontWeight(.semibold)
                
                Spacer()
                
                // 添加按钮
                Button(action: { showAddFolderSheet = true }) {
                    Image(systemName: "plus.circle.fill")
                        .font(.caption)
                        .foregroundColor(.blue)
                }
                .buttonStyle(.plain)
                .help("Add new bookmark")
            }
            .padding(.horizontal, 10)
            
            // 文件夹列表
            ScrollView(.vertical, showsIndicators: false) {
                VStack(alignment: .leading, spacing: 4) {
                    ForEach(container.folders) { folder in
                        BookmarkFolderItem(
                            folder: folder,
                            container: container
                        )
                    }
                }
                .padding(.horizontal, 6)
            }
        }
        .sheet(isPresented: $showAddFolderSheet) {
            AddFolderSheet(
                isPresented: $showAddFolderSheet,
                container: container
            )
        }
    }
}

struct AddFolderSheet: View {
    @Binding var isPresented: Bool
    @EnvironmentObject var bookmarkManager: BookmarkManager
    @State private var folderName = ""
    @State private var selectedEmoji = "📁"
    
    let container: TabContainer
    let emojis = ["📁", "⭐", "📌", "💼", "🎯", "📚", "🎨", "🔖"]
    
    var body: some View {
        VStack(spacing: 16) {
            HStack {
                Text("New Bookmark Folder")
                    .font(.headline)
                Spacer()
                Button("Cancel") { isPresented = false }
                    .buttonStyle(.plain)
            }
            .padding()
            
            VStack(alignment: .leading, spacing: 12) {
                TextField("Folder name", text: $folderName)
                    .textFieldStyle(.roundedBorder)
                
                Text("Choose an emoji:")
                    .font(.caption)
                    .foregroundColor(.secondary)
                
                LazyVGrid(columns: Array(repeating: GridItem(), count: 4)) {
                    ForEach(emojis, id: \.self) { emoji in
                        Button(emoji) {
                            selectedEmoji = emoji
                        }
                        .frame(height: 40)
                        .frame(maxWidth: .infinity)
                        .background(
                            RoundedRectangle(cornerRadius: 8)
                                .fill(selectedEmoji == emoji ? Color.blue.opacity(0.3) : Color.gray.opacity(0.1))
                        )
                    }
                }
            }
            .padding()
            
            HStack {
                Spacer()
                Button("Create") {
                    bookmarkManager.createFolder(
                        name: folderName,
                        emoji: selectedEmoji,
                        in: container
                    )
                    isPresented = false
                }
                .buttonStyle(.borderedProminent)
                .disabled(folderName.isEmpty)
            }
            .padding()
        }
        .frame(maxWidth: 400, maxHeight: 300)
    }
}
```

#### 4.3 BookmarkFolderItem.swift

```swift
import SwiftUI

struct BookmarkFolderItem: View {
    @State private var isExpanded = false
    @EnvironmentObject var bookmarkManager: BookmarkManager
    let folder: Folder
    let container: TabContainer
    
    var body: some View {
        VStack(alignment: .leading, spacing: 0) {
            // 文件夹项
            Button(action: { withAnimation { isExpanded.toggle() } }) {
                HStack(spacing: 8) {
                    Image(systemName: isExpanded ? "chevron.down" : "chevron.right")
                        .font(.caption)
                    
                    Text(folder.emoji)
                        .font(.system(size: 14))
                    
                    Text(folder.name)
                        .lineLimit(1)
                        .font(.caption)
                    
                    Spacer()
                    
                    Text("\((folder.bookmarks ?? []).count)")
                        .font(.caption2)
                        .foregroundColor(.secondary)
                }
                .frame(height: 28)
                .contentShape(Rectangle())
            }
            .buttonStyle(.plain)
            .contextMenu {
                BookmarkContextMenu(folder: folder)
            }
            
            // 展开后显示内容
            if isExpanded {
                VStack(alignment: .leading, spacing: 2) {
                    ForEach(folder.bookmarks ?? []) { bookmark in
                        BookmarkItem(bookmark: bookmark)
                    }
                    
                    // 子文件夹
                    ForEach(folder.children ?? []) { subfolder in
                        BookmarkFolderItem(
                            folder: subfolder,
                            container: container
                        )
                        .padding(.leading, 12)
                    }
                }
                .padding(.leading, 16)
                .padding(.vertical, 4)
            }
        }
    }
}

struct BookmarkContextMenu: View {
    @EnvironmentObject var bookmarkManager: BookmarkManager
    let folder: Folder
    
    var body: some View {
        Button("Rename") {
            // TODO: 弹出重命名对话框
        }
        
        Button("Delete", role: .destructive) {
            bookmarkManager.deleteFolder(folder)
        }
    }
}
```

#### 4.4 BookmarkItem.swift

```swift
import SwiftUI

struct BookmarkItem: View {
    @EnvironmentObject var tabManager: TabManager
    let bookmark: Bookmark
    
    var body: some View {
        Button(action: openBookmark) {
            HStack(spacing: 6) {
                // Favicon
                if let faviconLocalFile = bookmark.faviconLocalFile,
                   let image = NSImage(contentsOf: faviconLocalFile) {
                    Image(nsImage: image)
                        .resizable()
                        .scaledToFit()
                        .frame(width: 16, height: 16)
                } else if let favicon = bookmark.favicon,
                          let data = try? Data(contentsOf: favicon),
                          let image = NSImage(data: data) {
                    Image(nsImage: image)
                        .resizable()
                        .scaledToFit()
                        .frame(width: 16, height: 16)
                } else {
                    Image(systemName: "link")
                        .font(.caption)
                        .foregroundColor(.secondary)
                }
                
                // 标题
                Text(bookmark.title)
                    .lineLimit(1)
                    .font(.caption)
                
                Spacer()
                
                // 域名
                Text(bookmark.url.host ?? "")
                    .font(.caption2)
                    .foregroundColor(.secondary)
                    .lineLimit(1)
            }
            .frame(height: 24)
            .contentShape(Rectangle())
        }
        .buttonStyle(.plain)
        .contextMenu {
            Button("Open") { openBookmark() }
            Button("Open in New Tab") { 
                tabManager.openNewTab(with: bookmark.url)
            }
            Button("Copy Link") {
                NSPasteboard.general.setString(
                    bookmark.url.absoluteString,
                    forType: .string
                )
            }
            Divider()
            Button("Delete", role: .destructive) {
                // TODO: 删除书签
            }
        }
    }
    
    private func openBookmark() {
        if let activeTab = tabManager.activeTab {
            activeTab.url = bookmark.url
            activeTab.urlString = bookmark.url.absoluteString
        }
    }
}
```

---

### Step 5: 集成到侧边栏 (1 小时)

#### 5.1 修改 ContainerView.swift

**编辑文件**: `ora/Modules/Sidebar/ContainerView.swift`

在现有的标签列表后添加书签部分：

```swift
// 在 ScrollView 中的 VStack 里添加

Divider()

// 书签文件夹部分
if !privacyMode.isPrivate {
    BookmarkFoldersView(container: container)
        .environmentObject(bookmarkManager)
        .padding(.horizontal, 10)
}
```

#### 5.2 修改 OraRoot.swift

在初始化部分添加 BookmarkManager：

```swift
struct OraRoot: View {
    // ... 现有的 @StateObject 声明 ...
    
    @StateObject private var bookmarkManager: BookmarkManager
    
    init(isPrivate: Bool = false) {
        // ... 现有的初始化代码 ...
        
        // 初始化 BookmarkManager
        _bookmarkManager = StateObject(wrappedValue: BookmarkManager(
            modelContext: modelContext
        ))
    }
    
    var body: some View {
        // ... 在主视图中注入 bookmarkManager ...
        BrowserView()
            .environmentObject(bookmarkManager)
    }
}
```

#### 5.3 添加快捷键 (可选)

**修改文件**: `ora/Common/Constants/KeyboardShortcuts.swift`

```swift
// 添加
Cmd + D  // 添加书签
```

在相应视图中：

```swift
.keyboardShortcut("d", modifiers: .command)
```

---

## 🧪 测试检查清单

- [ ] **模型层**
  - [ ] Bookmark 对象可以创建和保存
  - [ ] Folder 对象支持子文件夹
  - [ ] 关系映射正确
  - [ ] 级联删除工作正常

- [ ] **服务层**
  - [ ] BookmarkManager 初始化正常
  - [ ] CRUD 操作正确
  - [ ] 持久化到 SwiftData

- [ ] **UI 层**
  - [ ] 文件夹可以展开/收起
  - [ ] 书签列表显示正确
  - [ ] 右键菜单正确
  - [ ] 点击书签可以打开链接

- [ ] **集成测试**
  - [ ] 侧边栏正确显示书签
  - [ ] 私密模式隐藏书签
  - [ ] 容器切换时书签正确刷新

---

## 🚀 后续优化（可选）

1. **拖拽排序**: 使用 `.onDrop` 和 `.draggable()` 实现
2. **搜索功能**: 在顶部添加搜索框
3. **书签导入**: 从 Safari/Chrome 导入书签
4. **快捷键**: Cmd+D 快速添加书签
5. **性能**: 对大量书签使用虚拟列表

---

## 📊 预期成果

完成上述实现后，你将获得：

✅ 完整的书签管理系统
✅ 支持文件夹层级
✅ 侧边栏集成展示
✅ 上下文菜单操作
✅ 书签搜索（可选）
✅ 私密模式隐藏

---

## 📚 参考资源

- **TabManager 参考**: `ora/Services/TabManager.swift` (462 行)
- **UI 列表参考**: `ora/Modules/Sidebar/TabList/*.swift`
- **拖拽参考**: `ora/Services/TabDropDelegate.swift`
- **SwiftData 参考**: `ora/Models/*.swift`

---

**总结**: 
按照上述 5 步循序渐进，可以在 18-25 小时内为 Ora Browser 实现一个完整且专业的书签管理功能。
