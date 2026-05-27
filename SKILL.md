---
name: "web-to-apk"
description: "Packages any single-page web app into an Android APK with immersive status bar. Invoke when user wants to convert/pack a web app to Android APK or build a mobile app from HTML."
---

# Web 应用打包为 Android APK

将任意单页 Web 应用打包为适配沉浸式状态栏的 Android APK。使用时只需提供 HTML 文件路径、应用名、包名即可，其余全部自动完成。

---

## 用户需提供的信息

- HTML 文件路径
- 应用名称
- 包名（如 com.example.myapp）

## 自动完成的全部步骤

---

### 一、环境准备（仅在首次使用时执行）

检查本机是否已安装以下工具，若缺失则自动下载安装：

1. **JDK 17** — 下载 Adoptium JDK 17 并解压到 `{工作目录}/android-build/jdk/`
2. **Android SDK 命令行工具** — 下载 `commandlinetools-{平台}`（Windows 为 `win`，Mac 为 `mac`，Linux 为 `linux`）并解压到 `{工作目录}/android-build/android-sdk/`，然后：
   - 重命名 `cmdline-tools/cmdline-tools` 为 `cmdline-tools/latest`
   - 预创建 `android-sdk/licenses/` 目录，写入以下两个文件以自动接受许可：
     - `android-sdk-license`：内容为 `24333f8a63b6825ea9c5514f83c2829b004d1fee` 和 `d56f5187479451eabf01fb78af6dfcb131a6481e`
     - `android-sdk-preview-license`：内容为 `84831b9409646a918e30573bab4c9c91346d8abd`
   - 运行 `sdkmanager --sdk_root={SDK路径} "platform-tools" "platforms;android-34" "build-tools;34.0.0"`
3. **Gradle 8.4** — 下载 `gradle-8.4-bin.zip` 并解压到 `{工作目录}/android-build/gradle/`

安装完成后记录三个路径，后续构建复用：
- `JDK_HOME` = JDK 解压目录
- `ANDROID_HOME` = Android SDK 目录
- `GRADLE_HOME` = Gradle 解压目录

---

### 二、创建 Android 项目

在 HTML 文件同级目录下创建 `{应用名}-android/` 项目，结构如下：

```
{应用名}-android/
├── build.gradle
├── settings.gradle
├── gradle.properties
├── gradlew.bat
├── gradle/wrapper/gradle-wrapper.properties
└── app/
    ├── build.gradle
    ├── proguard-rules.pro
    └── src/main/
        ├── AndroidManifest.xml
        ├── assets/www/index.html          ← 复制 Web 应用文件到此处
        ├── java/{包名路径}/MainActivity.java
        └── res/
            ├── layout/activity_main.xml
            ├── drawable/ic_launcher_background.xml
            ├── drawable/ic_launcher_foreground.xml
            ├── mipmap-anydpi-v26/ic_launcher.xml
            └── values/
                ├── strings.xml
                ├── colors.xml
                └── styles.xml
```

#### 各文件内容

**settings.gradle**
```groovy
pluginManagement {
    repositories {
        google()
        mavenCentral()
        gradlePluginPortal()
    }
}
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
    }
}
rootProject.name = "{应用名}"
include ':app'
```

**根 build.gradle**
```groovy
plugins {
    id 'com.android.application' version '8.1.0' apply false
}
```

**app/build.gradle**
```groovy
plugins {
    id 'com.android.application'
}
android {
    namespace '{包名}'
    compileSdk 34
    defaultConfig {
        applicationId "{包名}"
        minSdk 24
        targetSdk 34
        versionCode 1
        versionName "1.0"
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_17
        targetCompatibility JavaVersion.VERSION_17
    }
}
dependencies {
    implementation 'androidx.appcompat:appcompat:1.6.1'
    implementation 'androidx.webkit:webkit:1.8.0'
}
```

**gradle.properties**
```properties
android.useAndroidX=true
android.enableJetifier=true
android.suppressUnsupportedCompileSdk=34
org.gradle.jvmargs=-Xmx2048m
```

**gradle-wrapper.properties**
```properties
distributionBase=GRADLE_USER_HOME
distributionPath=wrapper/dists
distributionUrl=https\://services.gradle.org/distributions/gradle-8.4-bin.zip
networkTimeout=10000
validateDistributionUrl=true
zipStoreBase=GRADLE_USER_HOME
zipStorePath=wrapper/dists
```

**AndroidManifest.xml**
```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.CAMERA" />
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    <application
        android:allowBackup="true"
        android:icon="@drawable/ic_launcher_foreground"
        android:label="@string/app_name"
        android:roundIcon="@drawable/ic_launcher_foreground"
        android:supportsRtl="true"
        android:theme="@style/AppTheme"
        android:usesCleartextTraffic="true">
        <activity
            android:name=".MainActivity"
            android:exported="true"
            android:configChanges="orientation|screenSize|keyboardHidden"
            android:windowSoftInputMode="adjustResize">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>
</manifest>
```

**MainActivity.java**（⚠️ 关键文件，沉浸式状态栏 + 密度转换）
```java
package {包名};

import android.app.Activity;
import android.os.Build;
import android.os.Bundle;
import android.view.KeyEvent;
import android.view.View;
import android.view.WindowInsets;
import android.webkit.WebChromeClient;
import android.webkit.WebSettings;
import android.webkit.WebView;
import android.webkit.WebViewClient;

public class MainActivity extends Activity {
    private WebView webView;
    private int statusBarInset = 0;
    private int navBarInset = 0;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.R) {
            getWindow().setDecorFitsSystemWindows(false);
        } else {
            getWindow().getDecorView().setSystemUiVisibility(
                View.SYSTEM_UI_FLAG_LAYOUT_STABLE
                | View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN
            );
        }

        getWindow().setStatusBarColor(android.graphics.Color.TRANSPARENT);
        getWindow().setNavigationBarColor(android.graphics.Color.TRANSPARENT);

        setContentView(R.layout.activity_main);

        View decorView = getWindow().getDecorView();
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.R) {
            decorView.setOnApplyWindowInsetsListener((v, insets) -> {
                statusBarInset = insets.getInsets(WindowInsets.Type.systemBars()).top;
                navBarInset = insets.getInsets(WindowInsets.Type.systemBars()).bottom;
                injectInsets();
                return insets;
            });
        } else {
            decorView.setOnApplyWindowInsetsListener(new View.OnApplyWindowInsetsListener() {
                @Override
                public WindowInsets onApplyWindowInsets(View v, WindowInsets insets) {
                    statusBarInset = insets.getSystemWindowInsetTop();
                    navBarInset = insets.getSystemWindowInsetBottom();
                    injectInsets();
                    return insets;
                }
            });
        }

        webView = findViewById(R.id.webview);
        WebSettings settings = webView.getSettings();
        settings.setJavaScriptEnabled(true);
        settings.setDomStorageEnabled(true);
        settings.setAllowFileAccess(true);
        settings.setAllowContentAccess(true);
        settings.setMediaPlaybackRequiresUserGesture(false);
        settings.setBuiltInZoomControls(false);
        settings.setDisplayZoomControls(false);
        settings.setUseWideViewPort(true);
        settings.setLoadWithOverviewMode(true);
        settings.setSupportZoom(false);
        settings.setCacheMode(WebSettings.LOAD_DEFAULT);
        settings.setMixedContentMode(WebSettings.MIXED_CONTENT_ALWAYS_ALLOW);

        webView.setWebViewClient(new WebViewClient() {
            @Override
            public void onPageFinished(WebView view, String url) {
                injectInsets();
            }
        });
        webView.setWebChromeClient(new WebChromeClient());
        webView.setOverScrollMode(View.OVER_SCROLL_NEVER);
        webView.setScrollBarStyle(View.SCROLLBARS_INSIDE_OVERLAY);
        webView.loadUrl("file:///android_asset/www/index.html");
    }

    // ⚠️ 关键：物理像素必须除以 density 转换为 CSS 像素
    // WindowInsets 返回的是物理像素，而 WebView 中 1px = 1dp
    // 不做转换会导致安全区在高密度屏幕上被放大 density 倍
    private void injectInsets() {
        if (webView != null) {
            float density = getResources().getDisplayMetrics().density;
            int sbDp = Math.round(statusBarInset / density);
            int nbDp = Math.round(navBarInset / density);
            String js = "document.documentElement.style.setProperty('--status-bar-height','" + sbDp + "px');"
                + "document.documentElement.style.setProperty('--nav-bar-height','" + nbDp + "px');";
            webView.evaluateJavascript(js, null);
        }
    }

    @Override
    public boolean onKeyDown(int keyCode, KeyEvent event) {
        if (keyCode == KeyEvent.KEYCODE_BACK && webView.canGoBack()) {
            webView.goBack();
            return true;
        }
        return super.onKeyDown(keyCode, event);
    }

    @Override
    protected void onResume() {
        super.onResume();
        if (webView != null) webView.onResume();
    }

    @Override
    protected void onPause() {
        super.onPause();
        if (webView != null) webView.onPause();
    }

    @Override
    protected void onDestroy() {
        if (webView != null) {
            webView.destroy();
            webView = null;
        }
        super.onDestroy();
    }
}
```

**activity_main.xml**
```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <WebView
        android:id="@+id/webview"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />
</RelativeLayout>
```

**styles.xml**
```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <style name="AppTheme" parent="android:Theme.Material.Light.NoActionBar">
        <item name="android:colorPrimary">@color/colorPrimary</item>
        <item name="android:colorPrimaryDark">@color/colorPrimaryDark</item>
        <item name="android:colorAccent">@color/colorAccent</item>
        <item name="android:statusBarColor">@android:color/transparent</item>
        <item name="android:navigationBarColor">@android:color/transparent</item>
        <item name="android:windowTranslucentStatus">false</item>
        <item name="android:windowTranslucentNavigation">false</item>
        <item name="android:windowDrawsSystemBarBackgrounds">true</item>
    </style>
</resources>
```

**strings.xml**
```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="app_name">{应用名}</string>
</resources>
```

**colors.xml** — 根据应用主题色自动生成，读取 HTML 中 CSS 变量或主色调
```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <color name="colorPrimary">{从HTML中提取的主题色}</color>
    <color name="colorPrimaryDark">{主题色加深版}</color>
    <color name="colorAccent">{主题色}</color>
</resources>
```

**ic_launcher_background.xml** — 图标背景使用主题色
```xml
<?xml version="1.0" encoding="utf-8"?>
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="108dp"
    android:height="108dp"
    android:viewportWidth="108"
    android:viewportHeight="108">
    <path
        android:fillColor="{主题色}"
        android:pathData="M0,0h108v108h-108z"/>
</vector>
```

**ic_launcher_foreground.xml** — 图标前景，根据应用名首字母生成白色文字 path
```xml
<?xml version="1.0" encoding="utf-8"?>
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="108dp"
    android:height="108dp"
    android:viewportWidth="108"
    android:viewportHeight="108">
    <path
        android:fillColor="#ffffff"
        android:pathData="{根据应用名首字母生成的矢量路径}"/>
</vector>
```

**mipmap-anydpi-v26/ic_launcher.xml**
```xml
<?xml version="1.0" encoding="utf-8"?>
<adaptive-icon xmlns:android="http://schemas.android.com/apk/res/android">
    <background android:drawable="@drawable/ic_launcher_background"/>
    <foreground android:drawable="@drawable/ic_launcher_foreground"/>
</adaptive-icon>
```

---

### 三、HTML 沉浸式状态栏适配

MainActivity 通过 JS 注入两个 CSS 变量到 `:root`：
- `--status-bar-height`：状态栏实际高度（已做密度转换，单位等同于 dp）
- `--nav-bar-height`：底部导航栏实际高度（已做密度转换，单位等同于 dp）

**必须执行以下适配：**

1. 在 CSS `:root` 中声明默认值（浏览器预览时为 0，APK 中由原生代码覆盖）：
```css
:root {
  --status-bar-height: 0px;
  --nav-bar-height: 0px;
}
```

2. **分析该 Web 应用的实际 DOM 结构**，找到以下元素并适配：
   - **顶部固定区域**（header/toolbar/titlebar 等）：`padding-top` 增加 `var(--status-bar-height)`，`height` 也相应增加
   - **底部固定区域**（bottom-nav/tabbar/footer 等）：`height` 增加 `var(--nav-bar-height)`，内部内容加 `padding-bottom: var(--nav-bar-height)`
   - **可滚动内容区**：`padding-bottom` 增加 `var(--nav-bar-height)`，确保底部内容不被固定导航栏遮挡

3. 适配时只需加安全区高度，**不要再额外加多余的 padding**，状态栏/导航栏本身已提供视觉间距

**适配示例（不同布局结构）：**

```css
/* 场景 A：顶部固定 header + 底部固定 nav + 中间滚动 */
.fixed-header {
  padding-top: calc(var(--status-bar-height) + 2px);
  height: calc(原header高度 + var(--status-bar-height));
}
.fixed-bottom-nav {
  height: calc(原nav高度 + var(--nav-bar-height));
  padding-bottom: var(--nav-bar-height);
}
.scrollable-content {
  padding-bottom: calc(原nav高度 + var(--nav-bar-height) + 16px);
}

/* 场景 B：全屏滚动，无固定 header/nav */
.scrollable-content {
  padding-top: var(--status-bar-height);
  padding-bottom: var(--nav-bar-height);
}

/* 场景 C：顶部固定 header，无底部 nav */
.fixed-header {
  padding-top: calc(var(--status-bar-height) + 2px);
  height: calc(原header高度 + var(--status-bar-height));
}
.scrollable-content {
  padding-bottom: var(--nav-bar-height);
}
```

---

### 四、构建 APK

```powershell
# 设置环境变量（路径来自第一步安装记录）
$env:JAVA_HOME = "{JDK_HOME}"
$env:ANDROID_HOME = "{ANDROID_HOME}"
$env:ANDROID_SDK_ROOT = $env:ANDROID_HOME
$env:Path = "$env:JAVA_HOME\bin;{GRADLE_HOME}\bin;$env:ANDROID_HOME\platform-tools;$env:Path"

# 清理旧构建（首次或修改 Java 文件后必须执行）
Remove-Item "{项目目录}\app\build" -Recurse -Force -ErrorAction SilentlyContinue

# 构建（使用 --no-daemon 避免 AAPT2 daemon 启动失败）
& "{GRADLE_HOME}\bin\gradle.bat" -p "{项目目录}" assembleDebug --no-daemon
```

APK 输出：`{项目目录}\app\build\outputs\apk\debug\app-debug.apk`

---

### 五、已知问题与解决方案

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 安全区被放大 density 倍 | WindowInsets 返回物理像素，CSS px = dp | `Math.round(inset / density)` 转换后再注入 |
| 底部安全区过大 | `navigation_bar_height` 资源永远返回 48dp | 用 WindowInsets API 获取实际插边，不要读 dimen 资源 |
| 顶部安全区过大 | padding-top 额外加了多余 padding | 只加 `var(--status-bar-height) + 2px` |
| AAPT2 daemon 启动失败 | daemon 缓存冲突 | 构建时加 `--no-daemon`，或删除 `.gradle` 和 `app\build` |
| Gradle 缓存不重新编译 | Java 文件改动但输出 UP-TO-DATE | 先删除 `app\build` 目录再构建 |
| SDK 许可证问题 | sdkmanager 需要交互确认 | 预创建 `licenses/` 目录写入哈希值 |

---

### 六、AOSP 官方系统栏尺寸参考

来源：[frameworks/base/core/res/res/values/dimens.xml](https://android.googlesource.com/platform/frameworks/base/+/refs/heads/main/core/res/res/values/dimens.xml)

| 资源 | 默认值 | 说明 |
|------|--------|------|
| `status_bar_height` | 24dp | 竖屏状态栏高度，OEM 可覆盖（刘海屏可能 25~36dp） |
| `navigation_bar_height` | 48dp | 三键导航栏高度 |
| 手势导航底部插边 | ~20px | WindowInsets 实际返回值，远小于 48dp |

**重要：不要硬编码这些值，始终通过 WindowInsets API 动态获取。**
