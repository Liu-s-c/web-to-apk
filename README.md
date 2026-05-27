<p align="center">
  <h1 align="center">📱 web-to-apk</h1>
  <p align="center">
    <b>Turn any HTML webapp into a native Android APK — in one conversation.</b>
  </p>
</p>

---

## Why?

Vibe coding tools generate beautiful, functional web apps in seconds. But there's a catch: they're trapped in the browser. You can't install them, share them as an app, or use them on your phone the way you use every other app.

**web-to-apk** closes that gap.

It's an AI skill that takes a single HTML file and produces a real Android APK — with proper immersive status bar, native WebView wrapper, and correct density handling so it looks native, not like a janky website in a frame.

> You vibe-code a web app → You vibe-ship it to Android. Done.

### Why This Matters Specifically for Vibe Coding

**Output format match.** Vibe coding produces an `index.html` (plus optional CSS/JS/images). This skill consumes exactly that — drop in your HTML, get an APK. No conversion, no restructuring, no framework lock-in.

**Eliminates the install barrier.** A normal user has no idea what to do with an HTML file. But everyone knows what an APK is. One tap to install, icon on the home screen, works like any other app. Your AI-generated prototype becomes a phone experience instantly.

**Safe-area handling — vibe coding's blind spot.** AI-generated web apps rarely account for system bars (status bar, gesture bar). Throw them into a bare WebView and you get either content hidden behind bars or ugly black dead zones. This skill auto-injects CSS variables (`--status-bar-height`, `--nav-bar-height`) and provides adaptation templates — solving a professional-grade layout problem that vibe coding tools don't even think about.

**Zero environment friction.** Most vibe coding users are frontend devs or AI enthusiasts who don't have JDK, Android SDK, or Gradle installed. This skill detects missing tools and installs them automatically. They just bring the HTML — the skill handles the rest.

## What It Does

| Step | Description |
|------|-------------|
| **1. Environment** | Auto-detects and installs JDK 17, Android SDK CLI, and Gradle 8.4 if missing |
| **2. Project** | Creates a full Android project skeleton with WebView wrapper |
| **3. Immersive UI** | Adapts the HTML with CSS custom properties for status bar & navigation bar insets |
| **4. Build** | Compiles a debug APK you can install immediately — no Android Studio required |

## Immersive Status Bar Done Right

Most "web to APK" tools do this wrong. They either leave a black bar or over-compensate with massive padding.

This skill:

- Uses `WindowInsets` API to read the **actual** system bar heights at runtime
- Converts physical pixels to CSS pixels via `density` division — preventing 3× oversize on high-DPI screens
- Injects CSS variables (`--status-bar-height`, `--nav-bar-height`) your web app can use for pixel-perfect layout
- Handles gesture navigation naturally (bottom inset is ~20px, not the 48dp hardcoded in `dimens.xml`)

## Use Cases

| Scenario | Before | After |
|----------|--------|-------|
| Tool dashboard from Bolt/Lovable/v0 | Only works in browser | Installed APK, works offline |
| Prototype you want to show investors | "Open this URL on your phone" | "Here's the APK, install it" |
| Internal company tool | Always needs internet + browser | Push an APK to devices via MDM |
| Personal utility app | Can't put it on home screen | One-tap launch like any app |
| Student project / hackathon submission | Demo-gated by laptop | Judges install the APK directly |

## Quick Start

Just describe what you need to any AI assistant with the skill installed:

> *"Pack this web app into an APK. HTML is at `./index.html`, name it 'TaskFlow', package `com.mycompany.taskflow`."*

Three inputs. That's it. The skill handles everything else.

## Files

```
.
├── SKILL.md       # The AI skill — all instructions for the assistant
├── README.md      # You're reading it
└── LICENSE        # MIT
```

## How It Works (for the curious)

1. **WebView wrapper** — The APK embeds a native Android `WebView` that loads your HTML from local assets. No remote server needed.
2. **CSS adaptation** — Before building, the skill analyzes your HTML's DOM structure and adds safe-area padding to fixed headers, bottom navs, and scrollable content areas.
3. **Density-aware insets** — `MainActivity.java` reads system insets via `WindowInsets` API, divides by screen density, then injects the result into `:root` CSS variables.
4. **Gradle build** — Compiles with `compileSdk 34`, `minSdk 24`, supports Android 7.0+. Output is `app-debug.apk`.

## Permissions

The APK requests:
- `INTERNET` — for any external resources your app loads
- `CAMERA` — if your web app uses camera input
- `READ/WRITE_EXTERNAL_STORAGE` — for file uploads/downloads

## AOSP Reference

System bar dimensions per [AOSP `dimens.xml`](https://android.googlesource.com/platform/frameworks/base/+/refs/heads/main/core/res/res/values/dimens.xml):

| Resource | Default | Notes |
|----------|---------|-------|
| `status_bar_height` | 24dp | OEMs may override (notch phones: 25–36dp) |
| `navigation_bar_height` | 48dp | 3-button nav only; gesture nav is ~20px |
| Gesture nav inset | ~20px | From `WindowInsets` at runtime — never hardcode |

**Rule: never hardcode these values. Always use `WindowInsets` API.**

---

# 🇨🇳 中文

<p align="center">
  <b>将任意 HTML 网页应用一键打包为原生 Android APK。</b>
</p>

## 为什么需要它？

Vibe coding（氛围编程）工具能在几秒内生成漂亮的网页应用。但有个问题：它们被困在浏览器里。你没法像使用其他 App 那样安装到手机上、分享给朋友、放在桌面随时打开。

**web-to-apk** 填上了这个缺口。

它是一个 AI 技能，接收一个 HTML 文件，产出真正的 Android APK —— 自带沉浸式状态栏、原生 WebView 包装、正确的屏幕密度处理，装在手机上看起来就是原生 App，而不是一个套壳的网页。

> Vibe 写出一个网页 → Vibe 装进手机里。搞定。

### 为什么在 Vibe Coding 场景下特别有价值

**输出形式完美匹配。** Vibe coding 的产物就是一个 `index.html`（加上可能引用的 CSS/JS/图片）。这个技能直接吃这个文件，吐出来一个 APK。无需转换、无需重构、不绑定任何框架。

**消除安装门槛。** 普通用户拿到一个 HTML 文件不知道该怎么装到手机上，但 APK 他们认识。一键安装，桌面出现图标，跟普通 App 一样操作。你的 AI 生成原型瞬间变成手机上的真实体验。

**安全区适配 —— vibe coding 的盲区。** AI 生成的网页通常不考虑系统栏（状态栏、手势条），直接塞进 WebView 要么内容被遮挡，要么留出难看的黑边。这个技能自动注入 CSS 变量（`--status-bar-height`、`--nav-bar-height`），并提供适配模板 —— 解决了 vibe coding 工具根本不会去想的专业级布局问题。

**零环境依赖。** Vibe coding 的用户大多是前端开发者或 AI 爱好者，不一定装了 JDK、Android SDK、Gradle。这个技能自动检测缺失的工具并一键安装。他们只需要提供 HTML 文件路径，其余全自动。

## 流程

| 步骤 | 说明 |
|------|------|
| **1. 环境准备** | 自动检测并安装 JDK 17、Android SDK CLI、Gradle 8.4（缺失时自动补全） |
| **2. 创建项目** | 生成完整的 Android 项目骨架，内含 WebView 容器 |
| **3. 沉浸式适配** | 分析 HTML 的 DOM 结构，使用 CSS 自定义属性适配状态栏和导航栏 |
| **4. 构建 APK** | 编译 debug APK，立即可安装 —— 无需 Android Studio |

## 沉浸式状态栏的正确做法

市面上的"网页转 APK"工具大多没做对 —— 要么留下一片黑条，要么补偿过度、留出巨大空白。

本技能的做法：

- 用 `WindowInsets` API 在运行时读取**真实的**系统栏高度
- 将物理像素除以 `density` 转换为 CSS 像素 —— 避免高 DPI 屏幕上 3 倍放大
- 注入 CSS 变量（`--status-bar-height`、`--nav-bar-height`）让你的网页精准布局
- 自然适配全面屏手势（底部安全区约 20px，而非 `dimens.xml` 里写死的 48dp）

## 使用场景

| 场景 | 之前 | 之后 |
|------|------|------|
| 用 Bolt/Lovable/v0 等搭建的工具面板 | 只能在浏览器打开 | 安装为 APK，支持离线 |
| 给投资人展示原型 | "手机打开这个网址" | "这是安装包，直接装" |
| 公司内部工具 | 必须联网 + 浏览器 | 通过 MDM 下发 APK |
| 个人工具 App | 没法放桌面 | 像普通 App 一样点开就用 |
| 学生项目/黑客松展示 | 评委围在笔记本前看 | 评委直接安装 APK 到手机 |

## 快速开始

只需向安装了该技能的 AI 助手描述需求：

> *"把这个网页打包成 APK，HTML 路径是 `./index.html`，名叫「任务通」，包名 `com.mycompany.taskflow`"*

三项输入就够了，其余全部自动完成。

## 文件结构

```
.
├── SKILL.md       # AI 技能定义 —— 助手的完整指令
├── README.md      # 你正在看
└── LICENSE        # MIT
```

## 原理

1. **WebView 包装** — APK 内嵌原生 `WebView`，从本地 assets 加载 HTML，不需要远程服务器。
2. **CSS 适配** — 构建前分析 HTML 的 DOM 结构，为固定头部、底部导航、可滚动内容区添加安全区间距。
3. **密度感知** — `MainActivity.java` 通过 `WindowInsets` API 读取系统插边，除以屏幕密度后注入到 `:root` CSS 变量。
4. **Gradle 构建** — `compileSdk 34`，`minSdk 24`，兼容 Android 7.0+。输出 `app-debug.apk`。

## 权限

APK 会申请以下权限：
- `INTERNET` — 应用加载外部资源
- `CAMERA` — 应用使用相机
- `READ/WRITE_EXTERNAL_STORAGE` — 文件上传/下载

## AOSP 参考

系统栏尺寸，源自 [AOSP `dimens.xml`](https://android.googlesource.com/platform/frameworks/base/+/refs/heads/main/core/res/res/values/dimens.xml)：

| 资源 | 默认值 | 说明 |
|------|--------|------|
| `status_bar_height` | 24dp | OEM 可覆盖（刘海屏 25~36dp） |
| `navigation_bar_height` | 48dp | 仅三键导航；手势导航约 20px |
| 手势导航底部插边 | ~20px | 运行时通过 `WindowInsets` 获取 —— 切勿硬编码 |

**原则：永远不要硬编码这些值，始终使用 `WindowInsets` API 动态获取。**

## 许可证

MIT © [Liu-s-c](https://github.com/Liu-s-c)
