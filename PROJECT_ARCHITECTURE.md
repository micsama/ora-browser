# Ora Browser - 项目架构完整指南

## 📋 项目概览

**Ora Browser** 是一个为 macOS 构建的快速、安全、现代化的网络浏览器。项目采用 **SwiftUI + AppKit + WebKit** 技术栈，遵循组件化架构模式。

### 项目信息
- **语言**: Swift 5.9+
- **平台**: macOS 15.0+
- **UI 框架**: SwiftUI + AppKit
- **数据存储**: SwiftData
- **构建工具**: XcodeGen + Xcode 15+
- **开发阶段**: Alpha (活跃开发)
- **当前版本**: 0.2.11
- **许可证**: GPL-2.0

---

## 🏗️ 项目结构

### 目录树概览

```
browser/
├── ora/                          # 主源代码目录
│   ├── OraRoot.swift            # 应用根视图和全局状态初始化
│   ├── OraCommands.swift        # 菜单命令定义
│   │
│   ├── Modules/                 # 功能模块（特性化组件）
│   │   ├── Browser/             # 核心浏览器功能
│   │   ├── Sidebar/             # 侧边栏和标签列表
│   │   ├── URLBar/              # 地址栏
│   │   ├── Settings/            # 设置面板
│   │   ├── Launcher/            # 快速启动器
│   │   ├── SplitView/           # 分割视图组件
│   │   ├── TabSwitch/           # 标签切换器
│   │   ├── Find/                # 页面查找功能
│   │   ├── EmojiPicker/         # 表情选择器
│   │   ├── Importer/            # 数据导入工具
│   │   └── Player/              # 媒体播放控制
│   │
│   ├── Services/                # 业务逻辑层
│   │   ├── TabManager.swift     # 标签页管理
│   │   ├── HistoryManager.swift # 浏览历史管理
│   │   ├── DownloadManager.swift# 下载管理
│   │   ├── SearchEngineService.swift  # 搜索引擎配置
│   │   ├── PrivacyService.swift       # 隐私功能
│   │   ├── AppearanceManager.swift    # 主题管理
│   │   ├── MediaController.swift      # 媒体控制
│   │   ├── FaviconService.swift       # 网站图标获取
│   │   ├── DefaultBrowserManager.swift # 默认浏览器设置
│   │   ├── DownloadManager.swift      # 下载功能
│   │   ├── SidebarManager.swift       # 侧边栏状态管理
│   │   └── ...其他服务
│   │
│   ├── Models/                  # 数据模型
│   │   ├── Tab.swift            # 标签页模型
│   │   ├── TabContainer.swift   # 标签容器模型（分组/空间）
│   │   ├── History.swift        # 历史记录模型
│   │   ├── Download.swift       # 下载项模型
│   │   ├── Folder.swift         # 书签文件夹模型
│   │   ├── SearchEngine.swift   # 搜索引擎模型
│   │   └── Keyboard.swift       # 键盘快捷键模型
│   │
│   ├── Common/                  # 通用工具和常量
│   │   ├── Constants/           # 常量定义
│   │   │   ├── AppEvents.swift      # 应用事件常量
│   │   │   ├── KeyboardShortcuts.swift # 快捷键定义
│   │   │   ├── Theme.swift          # 主题常量
│   │   │   └── ContainerConstants.swift # 容器常量
│   │   │
│   │   ├── Extensions/          # Swift 扩展
│   │   │   ├── View+Modifiers.swift      # SwiftUI View 扩展
│   │   │   ├── View+Shortcuts.swift      # 快捷键相关扩展
│   │   │   ├── Color+Hex.swift          # 颜色工具
│   │   │   ├── NSWindow+Extensions.swift # macOS 窗口扩展
│   │   │   ├── EnvironmentValues+Window.swift # 环境值扩展
│   │   │   └── ModelConfiguration+Shared.swift # 数据模型扩展
│   │   │
│   │   ├── Utils/               # 实用工具类
│   │   │   ├── ClipboardUtils.swift  # 剪贴板操作
│   │   │   ├── TabUtils.swift       # 标签页工具
│   │   │   ├── SettingsStore.swift  # 设置存储
│   │   │   ├── WindowFactory.swift  # 窗口工厂
│   │   │   └── Utils.swift          # 通用工具函数
│   │   │
│   │   ├── Representables/      # AppKit 桥接组件
│   │   │   ├── WindowReader.swift      # 窗口读取器
│   │   │   ├── WindowAccessor.swift    # 窗口访问器
│   │   │   ├── BlurEffectView.swift    # 毛玻璃效果
│   │   │   ├── KeyCaptureView.swift    # 键盘捕获
│   │   │   └── GlobalMouseTrackingArea.swift # 鼠标跟踪
│   │   │
│   │   └── Shapes/              # 自定义形状
│   │       └── ConditionallyConcentricRectangle.swift # 同心矩形
│   │
│   ├── UI/                      # 通用 UI 组件
│   │   ├── Buttons/             # 按钮组件
│   │   ├── Dialog/              # 对话框系统
│   │   │   ├── DialogManager.swift
│   │   │   ├── DialogModel.swift
│   │   │   └── DialogView.swift
│   │   │
│   │   ├── Toast/               # 提示信息系统
│   │   │   ├── ToastManager.swift
│   │   │   ├── ToastModel.swift
│   │   │   └── ToastView.swift
│   │   │
│   │   ├── Icons/               # 图标组件
│   │   ├── Modifiers/           # SwiftUI 修饰符
│   │   ├── Inputs/              # 输入控件
│   │   ├── WebView.swift        # WebKit 集成
│   │   ├── URLBar.swift         # 地址栏 UI
│   │   ├── HomeView.swift       # 主页视图
│   │   ├── StatusPageView.swift # 错误页面
│   │   └── ...其他通用组件
│   │
│   ├── Assets.xcassets/         # 图片和颜色资源
│   │   ├── OraIcon.appiconset/  # 发布图标
│   │   ├── OraIconDev.appiconset/ # 开发图标
│   │   └── 其他图片资源
│   │
│   ├── Resources/               # 应用资源
│   │   └── ...配置文件
│   │
│   ├── Icons/                   # macOS 应用图标包
│   │   ├── OraIcon.icon         # 发布版本图标
│   │   └── OraIconDev.icon      # 开发版本图标
│   │
│   ├── Info.plist              # 应用配置
│   ├── ora.entitlements        # 沙盒权限（发布）
│   └── ora-debug.entitlements  # 沙盒权限（调试）
│
├── oraTests/                    # 单元测试
│   └── oraTests.swift
│
├── project.yml                  # XcodeGen 项目配置
├── .lefthook.yml               # Git 钩子配置
├── .swiftformat                # SwiftFormat 配置
├── .swiftlint.yml              # SwiftLint 配置
│
├── scripts/                     # 构建和开发脚本
│   ├── setup.sh                # 开发环境初始化
│   ├── build.sh                # Release 版本构建
│   ├── build-and-install.sh    # 构建并安装到应用程序
│   └── install-app.sh          # 安装应用
│
├── README.md                    # 项目说明
├── ROADMAP.md                   # 发展路线图
├── CONTRIBUTING.md              # 贡献指南
└── LICENSE                      # GPL-2.0 许可证
```

---

## 🔧 核心模块详解

### 1. **Modules** - 功能模块

#### Browser 模块
```
Modules/Browser/
├── BrowserView.swift           # 主浏览器视图容器
├── BrowserContentContainer.swift # 内容区域容器
├── BrowserWebContentView.swift  # WebView 包装器
├── BrowserSplitView.swift       # 分割视图布局
└── FloatingSidebarOverlay.swift # 浮动侧边栏覆盖层
```
**职责**: 核心浏览功能，标签页渲染，WebView 管理

#### Sidebar 模块
```
Modules/Sidebar/
├── SidebarView.swift            # 侧边栏主容器
├── SidebarToolbar.swift         # 顶部工具栏
├── ContainerView.swift          # 容器选择器
├── DownloadsWidget.swift        # 下载小部件
├── FloatingSidebar.swift        # 浮动侧边栏
├── TabList/                     # 标签列表视图
│   ├── NormalTabsList.swift     # 普通标签列表
│   ├── PinnedTabsList.swift     # 固定标签列表
│   ├── FavTabsList.swift        # 收藏标签列表
│   └── NewTabButton.swift       # 新标签按钮
└── BottomOption/                # 底部选项
    ├── ContainerSwitcher.swift  # 容器切换器
    ├── ContainerForm.swift      # 容器编辑表单
    ├── NewContainerButton.swift # 新建容器按钮
    └── EditContainerModal.swift # 编辑容器对话框
```
**职责**: 标签页列表显示、标签管理 UI、容器（Space）管理

#### Settings 模块
```
Modules/Settings/
├── SettingsContentView.swift    # 设置主视图
├── Sections/
│   ├── GeneralSettingsView.swift    # 通用设置
│   ├── SearchEngineSettingsView.swift # 搜索引擎设置
│   ├── PrivacySecuritySettingsView.swift # 隐私/安全设置
│   ├── ShortcutsSettingsView.swift  # 快捷键设置
│   ├── SpacesSettingsView.swift     # 容器设置
│   └── AppearanceSelector.swift     # 外观选择
```
**职责**: 应用设置界面和配置管理

#### Launcher 模块
```
Modules/Launcher/
├── LauncherView.swift           # 启动器主视图
├── Main/
│   ├── LauncherMain.swift       # 主搜索栏
│   ├── LauncherTextField.swift  # 文本输入框
│   └── SearchCapsule.swift      # 搜索胶囊样式
└── Suggestions/
    ├── LauncherSuggestionsView.swift # 建议列表
    └── LauncherSuggestionItem.swift  # 单个建议项
```
**职责**: 快速启动器、搜索建议、命令面板

#### SplitView 模块
```
Modules/SplitView/
├── Split.swift                  # 抽象分割容器
├── HSplit.swift                 # 水平分割
├── VSplit.swift                 # 竖直分割
├── Splitter.swift               # 分割线拖拽控制
├── SplitModifiers.swift         # 分割修饰符
├── SplitConstraints.swift       # 尺寸约束
├── SplitEnums.swift             # 枚举定义
├── SplitHolders.swift           # 数据容器
├── SplitStyling.swift           # 样式定义
└── CursorModifier.swift         # 鼠标光标
```
**职责**: 自定义分割视图系统（用于分屏浏览）

#### URLBar 模块
```
Modules/URLBar/
└── FloatingURLBar.swift         # 浮动地址栏
```
**职责**: 地址栏 UI 和输入管理

#### 其他模块
- **Importer**: 从其他浏览器导入数据（书签、历史等）
- **Find**: 页面内查找功能
- **EmojiPicker**: 表情选择器（用于容器标记）
- **Player**: 全局媒体播放控制
- **TabSwitch**: 标签页快速切换面板

---

### 2. **Services** - 业务逻辑层

#### 关键服务

| 服务 | 职责 |
|------|------|
| **TabManager** | 标签页生命周期、创建/关闭/切换、状态管理 |
| **TabContainerManager** | 容器（分组）操作，容器持久化 |
| **HistoryManager** | 浏览历史记录，CRUD 操作 |
| **DownloadManager** | 文件下载、进度跟踪、暂停/恢复 |
| **SearchEngineService** | 搜索引擎配置、切换、自定义搜索 |
| **PrivacyService** | 隐私浏览模式、追踪防护配置 |
| **AppearanceManager** | 主题切换（亮色/暗色/跟随系统）、颜色方案 |
| **MediaController** | 媒体播放全局控制 |
| **FaviconService** | 获取网站 favicon（使用 FavIconFinder 库） |
| **DefaultBrowserManager** | 设置为默认浏览器 |
| **SidebarManager** | 侧边栏显示/隐藏状态 |
| **ToolbarManager** | 工具栏状态管理 |
| **DialogManager** | 对话框队列管理 |
| **CustomKeyboardShortcutManager** | 自定义快捷键定义和处理 |
| **KeyModifierListener** | 监听修饰键（Cmd, Ctrl, Option 等）状态 |
| **TabScriptHandler** | JavaScript 脚本执行和 WebView 通信 |
| **SectionDropDelegate** | 拖拽释放委托 |
| **TabDropDelegate** | 标签页拖拽处理 |
| **Importer** | 导入 Chrome/Safari 数据 |
| **UpdateService** | Sparkle 自动更新管理 |

#### 服务交互流程图

```
┌─────────────────┐
│   UI 视图层      │ (Modules/*)
└────────┬────────┘
         │ 调用
┌────────▼──────────────────────────────────┐
│        Services（业务逻辑层）              │
├──────────────────────────────────────────┤
│ TabManager  │ HistoryManager │ Download  │
│ SearchEngine│ PrivacyService │ Appearance│
│ Favicon     │ MediaController│ Dialog    │
└────────┬──────────────────────────────────┘
         │ 持久化
┌────────▼──────────────────────────────────┐
│   Models（数据模型） + SwiftData 存储      │
├──────────────────────────────────────────┤
│ Tab, TabContainer, History, Download     │
│ Folder, SearchEngine, Keyboard           │
└──────────────────────────────────────────┘
```

---

### 3. **Models** - 数据模型

#### 核心数据模型

```swift
// 标签页模型
@Model class Tab {
    var id: UUID
    var title: String
    var url: URL?
    var pinned: Bool
    var order: Int
    var container: TabContainer  // 所属容器
    var lastAccessedAt: Date
    // ...其他属性
}

// 容器模型（分组/Space）
@Model class TabContainer {
    var id: UUID
    var name: String              // 容器名称
    var emoji: String             // 容器表情
    @Relationship(deleteRule: .cascade)
    var tabs: [Tab]               // 关联标签页
    @Relationship(deleteRule: .cascade)
    var folders: [Folder]         // 关联书签文件夹
    @Relationship()
    var history: [History]        // 浏览历史
}

// 历史记录模型
@Model class History {
    var url: String
    var title: String
    var timestamp: Date
    var container: TabContainer   // 关联容器
}

// 下载项模型
@Model class Download {
    var id: UUID
    var filename: String
    var url: String
    var downloadedBytes: Int64
    var totalBytes: Int64
    var status: DownloadStatus    // 进行中/已完成/失败
    var createdAt: Date
}

// 书签文件夹模型
@Model class Folder {
    var id: UUID
    var name: String
    var parentFolder: Folder?
    @Relationship(deleteRule: .cascade)
    var children: [Folder]
}

// 搜索引擎模型
@Model class SearchEngine {
    var id: UUID
    var name: String
    var urlTemplate: String       // %s 为搜索词占位符
    var isDefault: Bool
}
```

#### 数据存储方案
- **数据库**: SwiftData（苹果原生，基于 SwiftUI 原生集成）
- **私密浏览**: 使用内存数据库（不持久化）
- **存储位置**: `~/Library/Application Support/Ora/`

---

### 4. **Common** - 通用工具层

#### Constants（常量定义）

```swift
// KeyboardShortcuts.swift - 定义所有快捷键
Cmd + T           // 新标签页
Cmd + W           // 关闭标签页
Cmd + Shift + N   // 新窗口
Cmd + ,           // 打开设置
Cmd + Y           // 打开历史
Cmd + B           // 显示/隐藏书签
Cmd + H           // 主页
Cmd + L           // 聚焦地址栏
Cmd + F           // 页面查找
Cmd + D           // 添加书签
Space             // 快速启动器
...

// Theme.swift - 主题和颜色定义
AppColors
  - primary
  - secondary
  - background
  - ...

// ContainerConstants.swift - 容器相关常量
// AppEvents.swift - 应用事件定义
```

#### Extensions（扩展）

| 扩展 | 功能 |
|------|------|
| **View+Modifiers** | SwiftUI 自定义修饰符（圆角、阴影等） |
| **View+Shortcuts** | 快捷键绑定扩展 |
| **Color+Hex** | 十六进制颜色解析 |
| **NSWindow+Extensions** | macOS 窗口操作（最大化、聚焦等） |
| **EnvironmentValues+Window** | 环境值中注入窗口引用 |
| **ModelConfiguration+Shared** | SwiftData 配置共享 |

#### Utils（工具函数）

```swift
// ClipboardUtils.swift
func copyToClipboard(_ text: String)
func getClipboardContent() -> String?

// TabUtils.swift
func generateNewTab()
func closeTab(_ tab: Tab)
func reorderTabs(from: Tab, to: Tab)

// SettingsStore.swift
@propertyWrapper class AppSettings
  - 读写用户设置
  - UserDefaults 存储

// WindowFactory.swift
func createNewWindow(with container: TabContainer)
func restoreWindowState()

// Utils.swift
func debounce(_ fn: @escaping () -> Void, delay: Double)
func openURL(_ url: URL)
...
```

#### Representables（AppKit 桥接）

这些组件用于 SwiftUI 中集成 macOS/AppKit 功能：

```swift
// WindowReader - 读取当前窗口引用
// WindowAccessor - 访问和修改窗口属性
// BlurEffectView - 实现毛玻璃效果（NSVisualEffectView）
// KeyCaptureView - 捕获键盘输入事件
// GlobalMouseTrackingArea - 全局鼠标跟踪
```

#### Shapes（自定义形状）

```swift
// ConditionallyConcentricRectangle
// 条件性同心矩形（用于特殊 UI 效果）
```

---

## 📦 第三方依赖

项目在 `project.yml` 中定义的 Swift Package Dependencies：

| 依赖 | 版本 | 用途 |
|------|------|------|
| **Sparkle** | 2.6.0+ | 自动更新框架 |
| **Inject** | 1.5.2+ | 热重载支持（调试时实时代码更新） |
| **FavIconFinder** | 5.1.5 | 网站 favicon 获取 |
| **WebKit** (系统库) | - | 网页渲染引擎 |
| **SwiftData** (系统库) | - | 数据持久化 |
| **SwiftUI** (系统库) | - | UI 框架 |

---

## 🎯 关键工作流

### 应用启动流程

```
main.swift (App Delegate)
    ↓
OraApp (根应用)
    ↓
OraRoot (根视图) - 初始化所有服务和状态
    ├─ AppState
    ├─ TabManager → 加载保存的标签页
    ├─ HistoryManager → 加载历史记录
    ├─ DownloadManager
    ├─ SearchEngineService → 加载搜索引擎配置
    ├─ AppearanceManager → 应用主题
    ├─ MediaController
    └─ 其他服务...
    ↓
BrowserView (主浏览器视图)
    ├─ SidebarView (左侧标签列表)
    └─ BrowserContentContainer (中间内容)
        └─ WebView (WebKit)
```

### 打开新标签页流程

```
用户操作: Cmd+T / 点击"新标签"按钮
    ↓
NewTabButton.swift 触发事件
    ↓
TabManager.createNewTab()
    ├─ 创建新 Tab 对象
    ├─ 设置默认 URL（主页）
    └─ 保存到 SwiftData
    ↓
UI 自动更新 (ObservableObject 响应式更新)
    ├─ SidebarView 中新增标签列表项
    └─ BrowserContentContainer 显示新页面
```

### 地址栏导航流程

```
用户输入 URL / 搜索词
    ↓
URLBar.swift 捕获回车
    ↓
解析输入（URL 还是搜索词）
    ├─ 若是搜索词 → SearchEngineService.buildSearchURL()
    └─ 若是 URL → 直接使用
    ↓
TabManager.updateTab(url:)
    ↓
WebView 加载 URL
    ↓
页面加载后 → 提取标题、favicon
    ├─ FaviconService.fetchFavicon()
    └─ Tab 属性自动更新
    ↓
UI 显示更新后的标签信息
```

### 容器切换流程

```
用户在侧边栏选择容器
    ↓
ContainerSwitcher / ContainerView 更新选中容器
    ↓
TabManager.switchContainer(_ container: TabContainer)
    ├─ 加载容器内的所有标签页
    └─ 恢复容器的上一次焦点标签页
    ↓
SidebarView 更新标签列表
BrowserView 更新显示内容
```

---

## 📊 数据流架构

### MVVM + Combine 架构

```
┌─────────────────────────────────────────────────┐
│           View Layer (SwiftUI)                  │
│  - BrowserView, SidebarView, URLBar...          │
│  @StateObject: 观察 ViewModel                   │
└──────────────────┬──────────────────────────────┘
                   │ @Published / ObservableObject
                   ↓
┌─────────────────────────────────────────────────┐
│         ViewModel/Services Layer                │
│  - TabManager, HistoryManager, Services...      │
│  @Published var: 数据源                         │
│  ObservableObject: 响应式更新                   │
└──────────────────┬──────────────────────────────┘
                   │ 调用/保存
                   ↓
┌─────────────────────────────────────────────────┐
│          Model Layer (SwiftData)                │
│  - Tab, TabContainer, History, Download...      │
│  @Model: 自动 CRUD 操作                         │
└─────────────────────────────────────────────────┘
```

### 环境对象（Environment Objects）

这些对象在 OraRoot 中初始化，并通过 `.environmentObject()` 注入到视图树：

```swift
@StateObject var tabManager: TabManager
@StateObject var historyManager: HistoryManager
@StateObject var downloadManager: DownloadManager
@StateObject var dialogManager: DialogManager
@StateObject var sidebarManager: SidebarManager
// ... 其他全局状态

// 在 BrowserView 及其子视图中可直接访问：
@EnvironmentObject var tabManager: TabManager
```

---

## 🔌 WebKit 集成

### WebView 组件

**文件**: `ora/UI/WebView.swift`

```swift
struct WebView: NSViewRepresentable {
    @ObservedObject var model: Tab
    @EnvironmentObject var tabManager: TabManager

    func makeNSView(context: Context) -> WKWebView {
        // 配置 WKWebView
        // 注入脚本处理器
        // 设置委托
    }

    func updateNSView(_ webView: WKWebView, context: Context) {
        // 监听 URL 变更
        // 处理页面加载事件
    }
}
```

### JavaScript 通信

**文件**: `ora/Services/TabScriptHandler.swift`

```swift
class TabScriptHandler: NSObject, WKScriptMessageHandler {
    func userContentController(
        _ userContentController: WKUserContentController,
        didReceive message: WKScriptMessage
    ) {
        // 处理来自网页的消息
        // 如：获取页面信息、执行脚本指令
    }
}
```

### 内容阻止（Content Blocking）

```swift
// PrivacyService 配置 WKContentRuleList
// 阻止追踪脚本、广告 URL 等
```

---

## 🎨 UI 组件库

### 通用组件

| 组件 | 位置 | 说明 |
|------|------|------|
| **OraButton** | `UI/Buttons/` | 自定义按钮样式 |
| **URLBarButton** | `UI/Buttons/` | 地址栏按钮 |
| **Toast** | `UI/Toast/` | 消息提示 |
| **Dialog** | `UI/Dialog/` | 对话框系统 |
| **TabItem** | `UI/TabItem.swift` | 标签项样式 |
| **FavTabItem** | `UI/FavTabItem.swift` | 收藏标签样式 |
| **DownloadProgressView** | `UI/` | 下载进度显示 |
| **StatusPageView** | `UI/` | 错误/空页面 |
| **VerticalScrollView** | `UI/` | 自定义滚动视图 |
| **LinkPreview** | `UI/` | 链接预览 |
| **CopiedURLOverlay** | `UI/` | 复制 URL 提示 |
| **WindowControls** | `UI/` | 窗口控制按钮 |

### 修饰符（Modifiers）

```swift
// UI/Modifiers/
GradientAnimatingBorder   // 渐变动画边框
// ... 其他自定义修饰符

// View+Modifiers.swift
extension View {
    func oraButton() -> some View
    func oraTextField() -> some View
    // ...
}
```

---

## 🔐 隐私与安全

### 沙盒权限配置

**文件**: `ora/ora.entitlements` (发布) 和 `ora/ora-debug.entitlements` (调试)

```xml
<!-- 网络访问 -->
<key>com.apple.security.network.client</key>
<true/>

<!-- 文件访问 -->
<key>com.apple.security.files.downloads.read-write</key>
<true/>
<key>com.apple.security.files.user-selected.read-write</key>
<true/>

<!-- 摄像头和麦克风 -->
<key>com.apple.security.device.camera</key>
<true/>
<key>com.apple.security.device.audio-input</key>
<true/>

<!-- 其他权限... -->
```

### 隐私浏览模式

```swift
// PrivacyMode 类 (OraRoot.swift)
class PrivacyMode: ObservableObject {
    @Published var isPrivate: Bool
}

// 私密模式下：
// - 使用内存数据库（ModelConfiguration.createOraContainer(isPrivate: true)）
// - 不保存历史记录
// - 不保存下载记录
```

### 追踪防护

```swift
// PrivacyService.swift
// 使用 WKContentRuleList 阻止追踪脚本
// 配置广告/追踪 URL 黑名单
```

---

## 🏗️ 构建和部署

### 构建配置

**文件**: `project.yml`

```yaml
targets:
  ora:
    type: application
    platform: macOS
    deploymentTarget: "15.0"

    settings:
      base:
        SWIFT_VERSION: 5.9
        CODE_SIGN_STYLE: Automatic
        MARKETING_VERSION: 0.2.11
        PRODUCT_BUNDLE_IDENTIFIER: com.orabrowser.app
```

### 构建脚本

| 脚本 | 功能 |
|------|------|
| **setup.sh** | 安装开发工具（xcodegen, swiftlint, xcbeautify, lefthook） |
| **build.sh** | 编译 Release 版本并复制到 `./build/Ora.app` |
| **build-and-install.sh** | 编译并安装到 `/Applications` |
| **install-app.sh** | 将已编译的应用安装到 `/Applications` |

### 自动更新配置

```swift
// project.yml - Sparkle 配置
SUFeedURL: "https://the-ora.github.io/browser/appcast.xml"
SUPublicEDKey: "Ozj+rezzbJAD76RfajtfQ7rFojJbpFSCl/0DcFSBCTI="
SUEnableAutomaticChecks: YES
```

---

## 🧪 测试

### 测试框架
- **单元测试**: XCTest
- **测试文件**: `oraTests/oraTests.swift`

### 运行测试
```bash
# 在 Xcode 中
Cmd + U

# 命令行
xcodebuild test -scheme ora -destination "platform=macOS"
```

---

## 🔄 开发工作流

### 代码风格

- **SwiftLint**: 自动代码检查（`.swiftlint.yml`）
- **SwiftFormat**: 代码自动格式化（`.swiftformat`）
- **Git Hooks**: 提交前自动运行检查（`.lefthook.yml`）

### 常见开发任务

#### 添加新功能模块

1. **创建模块目录**: `ora/Modules/YourFeature/`
2. **创建主视图**: `YourFeatureView.swift`
3. **添加数据模型**: `ora/Models/YourModel.swift`
4. **创建服务层**: `ora/Services/YourService.swift`
5. **集成到主视图**: 在 `OraRoot.swift` 中初始化服务，在 `BrowserView.swift` 中集成 UI

#### 添加新快捷键

1. **编辑**: `ora/Common/Constants/KeyboardShortcuts.swift`
2. **绑定处理**: 在对应视图中使用 `.keyboardShortcut()`
3. **或通过**: `CustomKeyboardShortcutManager` 实现自定义快捷键

#### 修改数据模型

1. **编辑**: `ora/Models/*.swift`
2. **修改 `@Model` 属性**时需考虑迁移策略
3. **SwiftData 自动处理简单更改**（添加可选属性等）

#### 添加设置项

1. **UI**: `ora/Modules/Settings/Sections/YourSettingsView.swift`
2. **存储**: `ora/Common/Utils/SettingsStore.swift` 中添加属性
3. **集成**: 在 `SettingsContentView.swift` 中添加选项卡

---

## 📈 项目进度

### 当前阶段: Alpha（活跃开发）

#### 已完成功能 ✅
- 核心浏览功能（标签页、导航）
- 侧边栏和竖标签布局
- 容器/分组（Spaces）
- 标签固定和重排
- 浮动标签切换器
- 页面内查找
- 历史记录
- 私密浏览模式
- 开发者工具
- 快捷键定义
- 自动更新（Sparkle）

#### 计划实现（Beta 1）📋
- 自动填充（iCloud Keychain）
- 书签管理和文件夹
- 下载管理器（暂停/恢复）
- Peek（预览网页）
- 分屏视图
- 权限管理
- 广告阻止器
- 指纹防护

#### 长期规划 🚀
- 完整扩展生态（Safari + Chrome 扩展）
- 主题系统
- 性能优化
- UI 完善

详见: `ROADMAP.md`

---

## 🤝 贡献指南

### 贡献流程

1. **Fork** 项目
2. **创建特性分支**: `git checkout -b feature/your-feature`
3. **提交代码**: 遵循代码规范（自动通过 Git hooks 检查）
4. **推送**: `git push origin feature/your-feature`
5. **提交 PR**: 填写 PR 模板

### 代码审查清单

- [ ] 遵循 Swift 命名约定
- [ ] 代码通过 SwiftLint 检查
- [ ] 添加了必要的注释
- [ ] 处理了错误情况
- [ ] 添加了相关测试
- [ ] 更新了 ROADMAP（如果影响功能）

详见: `CONTRIBUTING.md`

---

## 🔗 相关资源

### 官方链接
- **网站**: https://www.orabrowser.com
- **GitHub**: https://github.com/the-ora/browser
- **Wiki**: https://github.com/the-ora/browser/wiki
- **Discord**: https://discord.gg/9aZWH52Zjm

### 技术文档
- [SwiftUI 官方文档](https://developer.apple.com/xcode/swiftui/)
- [WebKit 文档](https://webkit.org/)
- [SwiftData 指南](https://developer.apple.com/xcode/swiftdata/)
- [Sparkle 更新框架](https://sparkle-project.org/)

---

## 📝 快速参考

### 常用命令

```bash
# 开发环境初始化
./scripts/setup.sh

# 打开项目
open Ora.xcodeproj

# 编译 Debug 版本
xcodebuild -project Ora.xcodeproj -scheme ora -configuration Debug build

# 编译 Release 版本
xcodebuild -project Ora.xcodeproj -scheme ora -configuration Release build

# 运行测试
xcodebuild test -scheme ora -destination "platform=macOS"

# 重新生成 Xcode 项目（修改 project.yml 后）
xcodegen
```

### 常用文件导航

| 任务 | 文件位置 |
|------|---------|
| 修改应用入口 | `ora/OraRoot.swift` |
| 添加全局命令 | `ora/OraCommands.swift` |
| 修改主浏览器 UI | `ora/Modules/Browser/*.swift` |
| 添加快捷键 | `ora/Common/Constants/KeyboardShortcuts.swift` |
| 修改数据模型 | `ora/Models/*.swift` |
| 添加业务逻辑 | `ora/Services/*.swift` |
| 修改应用配置 | `project.yml` |
| 修改权限 | `ora/ora.entitlements` |

---

## 🐛 调试技巧

### 启用日志输出
```swift
// 在相关服务中使用
print("[TabManager] Creating new tab: \(tab.id)")
```

### 使用 Xcode 调试器
- **设置断点**: Cmd + \
- **查看变量**: 右侧调试面板
- **执行表达式**: Cmd + Shift + Y

### 测试私密浏览
```swift
// 在 OraRoot.swift 中修改初始化
init(isPrivate: true)  // 启用私密模式
```

### 热重载（Inject 框架）
- 修改代码后自动重新编译（Debug 版本）
- 无需重启应用查看 UI 变化

---

## 📚 文件大小和行数统计

```
总 Swift 文件: ~134 个
代码规模: 中等规模项目（正在积极开发中）

主要模块行数分布:
- Modules/: ~40% 代码（UI 组件和视图）
- Services/: ~30% 代码（业务逻辑）
- UI/: ~15% 代码（通用组件）
- Models/: ~8% 代码（数据定义）
- Common/: ~7% 代码（工具和扩展）
```

---

## 🎓 学习路径建议

### 初学者
1. 阅读 `README.md` 和 `CONTRIBUTING.md`
2. 运行 `./scripts/setup.sh` 完成开发环境设置
3. 在 Xcode 中打开项目，编译 Debug 版本
4. 尝试修改简单的 UI（如按钮文字）
5. 学习基本的 SwiftUI 知识

### 中级开发者
1. 理解 `OraRoot.swift` 中的应用结构
2. 阅读一个完整的功能模块（如 `Modules/Sidebar/`）
3. 添加一个新的快捷键功能
4. 修改搜索引擎选择逻辑
5. 贡献一个简单的 Bug 修复

### 高级开发者
1. 设计新的功能模块（参考现有模块结构）
2. 优化 WebView 性能或集成新的 WebKit 功能
3. 实现复杂的 UI 动画或交互
4. 处理数据迁移和兼容性问题
5. 提交架构相关的 PR

---

**最后更新**: 2026-03-03
**Ora Browser 版本**: 0.2.11
**项目阶段**: Alpha (活跃开发)
