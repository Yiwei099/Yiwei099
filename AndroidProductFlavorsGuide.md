# Android 多风味（Product Flavors）配置指南

## 📖 概述

本文档介绍如何通过 Android Gradle 的 `Product Flavors` 功能，使用**一份源代码**配置出**多个不同包名、版本、签名、图标、主题色**的独立 APK。

---

## 🎯 实现目标

| 风味           | 包名                     | 应用名 | 签名文件        | 主题色 |
|--------------|------------------------|-----|-------------|--------|
| **banana** | `com.eiviayw.banana` | 香蕉  | `banana.keystore` | #2169F9 |
| **apple** | `com.eiviayw.apple` | 苹果  | `apple.keystore` | E89DB9 |

---

## 📁 项目目录结构

```
app/
├── src/
│   ├── main/
│   │   ├── java/.../           # 公共代码
│   │   ├── res/                # 公共资源（默认图标、字符串等）
│   │   │   ├── values/
│   │   │   │   └── colors.xml  # 默认颜色
│   │   │   └── ...
│   │   └── AndroidManifest.xml # 公共配置
│   │
│   ├── banana/                # banana 风味
│   │   ├── res/
│   │   │   ├── mipmap-hdpi/    # 风味专用图标
│   │   │   │   └── logo.png
│   │   │   └── values/
│   │   │       ├── colors.xml  # 覆盖主题色
│   │   │       └── strings.xml # 覆盖应用名
│   │   └── java/.../           # 风味专属代码（可选）
│   │
│   └── apple/                  # apple 风味
│       ├── res/
│       │   ├── mipmap-hdpi/
│       │   │   └── logo.png
│       │   └── values/
│       │       ├── colors.xml
│       │       └── strings.xml
│       └── java/.../
│
├── keystore/                   # 签名文件目录
│   ├── banana.keystore
│   └── apple.keystore
│
└── build.gradle.kts            # 模块级构建配置
```

---

## ⚙️ Gradle 完整配置

### `app/build.gradle.kts`

```kotlin
android {
    namespace = "com.eiviayw.banana"
    compileSdk = 36

    // ==================== URL 常量定义 ====================
    // 正式环境节点
    val BANANA_RELEASE_URL = listOf(
        "url1",
        "url2",
        // ... 更多 URL
    )

    // 测试环境节点
    val BANANA_DEBUG_URL = listOf(
        "url1",
        "url1",
    )

    // Apple 正式环境节点
    val APPLE_RELEASE_URL = listOf("url1", "url2")
    // Apple 测试环境节点
    val APPLE_DEBUG_URL = listOf("url1","url2")

    fun formatStringArray(list: List<String>): String {
        return list.joinToString(prefix = "{", separator = ", ", postfix = "}") { "\"$it\"" }
    }

    // ==================== 签名配置 ====================
    signingConfigs {
        create("banana") {
            storeFile = file("../keystore/banana.keystore")
            storePassword = "your_banana_password"
            keyAlias = "Key0"
            keyPassword = "your_banana_key_password"
        }

        create("apple") {
            storeFile = file("../keystore/apple.keystore")
            storePassword = "your_apple_password"
            keyAlias = "Key0"
            keyPassword = "your_apple_key_password"
        }
    }

    // ==================== 默认配置 ====================
    defaultConfig {
        applicationId = "com.eiviayw.banana"
        minSdk = 24
        targetSdk = 36
        versionCode = 1
        versionName = "1.0.0"

        testInstrumentationRunner = "androidx.test.runner.AndroidJUnitRunner"
        
        // 默认 BuildConfig 字段（让 IDE 识别）
        // 开发时生效，可以依据业务需要修改不同的节点集合，打包时会被其它脚本覆盖
        buildConfigField("String[]", "NODE_ARR", formatStringArray(BANANA_RELEASE_URL))
    }

    // ==================== 产品风味配置 ====================
    flavorDimensions += "appType"
    
    productFlavors {
        create("banana") {
            dimension = "appType"
            applicationId = "com.eiviayw.banana"
            versionCode = 1
            versionName = "1.0.0"
            signingConfig = signingConfigs.getByName("banana")
        }

        create("apple") {
            dimension = "appType"
            applicationId = "com.eiviayw.apple"
            versionCode = 100001
            versionName = "1.1.0"
            signingConfig = signingConfigs.getByName("apple")
        }
    }

    // ==================== 构建类型 ====================
    buildTypes {
        release {
            isMinifyEnabled = true
            proguardFiles(
                getDefaultProguardFile("proguard-android-optimize.txt"),
                "proguard-rules.pro"
            )
        }
        debug {
            isMinifyEnabled = false
        }
    }

    // ==================== APK 输出配置 ====================
    applicationVariants.all { variant ->
        val variantName = variant.name
        
        when {
            variantName == "bananaDebug" -> {
                variant.buildConfigField("String[]", "NODE_ARR", formatStringArray(BANANA_DEBUG_URL))
                variant.outputs.all {
                    if (this is BaseVariantOutputImpl) {
                        outputFileName = "香蕉_v${variant.versionName}_debug.apk"
                    }
                }
            }
            variantName == "appleDebug" -> {
                variant.buildConfigField("String[]", "NODE_ARR", formatStringArray(APPLE_DEBUG_URL))
                variant.outputs.all {
                    if (this is BaseVariantOutputImpl) {
                        outputFileName = "苹果_v${variant.versionName}_debug.apk"
                    }
                }
            }
            variantName == "bananaRelease" -> {
                variant.buildConfigField("String[]", "NODE_ARR", formatStringArray(BANANA_RELEASE_URL))
                variant.outputs.all {
                    if (this is BaseVariantOutputImpl) {
                        outputFileName = "香蕉_v${variant.versionName}.apk"
                    }
                }
            }
            variantName == "appleRelease" -> {
                variant.buildConfigField("String[]", "NODE_ARR", formatStringArray(APPLE_RELEASE_URL))
                variant.outputs.all {
                    if (this is BaseVariantOutputImpl) {
                        outputFileName = "苹果_v${variant.versionName}.apk"
                    }
                }
            }
        }
    }

    // ==================== 其他配置 ====================
    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_17
        targetCompatibility = JavaVersion.VERSION_17
    }
    
    kotlinOptions {
        jvmTarget = "17"
    }
    
    buildFeatures {
        compose = true
        //启用
        buildConfig = true
    }
    
    hilt {
        enableAggregatingTask = false
    }
}
```

---

## 🎨 资源覆盖配置

### 1. 应用名称（`strings.xml`）

**`src/main/res/values/strings.xml`**（默认）：
```xml
<resources>
    <string name="app_name">香蕉</string>
</resources>
```

**`src/banana/res/values/strings.xml`**：
```xml
<resources>
    <string name="app_name">香蕉</string>
</resources>
```

**`src/apple/res/values/strings.xml`**：
```xml
<resources>
    <string name="app_name">苹果</string>
</resources>
```

### 2. 主题颜色（`colors.xml`）

**`src/main/res/values/colors.xml`**（默认）：
```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <color name="purple_500">#FF6200EE</color>
    <color name="purple_700">#FF3700B3</color>
    <color name="teal_200">#FF03DAC5</color>
</resources>
```

**`src/banana/res/values/colors.xml`**：
```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <color name="purple_500">#6200EE</color>
    <color name="purple_700">#3700B3</color>
    <color name="teal_200">#03DAC6</color>
</resources>
```

**`src/apple/res/values/colors.xml`**：
```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <color name="purple_500">#FF6B6B</color>
    <color name="purple_700">#E25555</color>
    <color name="teal_200">#FFE66D</color>
</resources>
```

### 3. Compose 主题中读取颜色

```kotlin
// Theme.kt
import androidx.compose.ui.res.colorResource

@Composable
fun AppTheme(
    darkTheme: Boolean = isSystemInDarkTheme(),
    content: @Composable () -> Unit
) {
    val colorScheme = if (darkTheme) {
        darkColorScheme(
            primary = colorResource(R.color.purple_500),
            primaryContainer = colorResource(R.color.purple_700),
            secondary = colorResource(R.color.teal_200),
        )
    } else {
        lightColorScheme(
            primary = colorResource(R.color.purple_500),
            primaryContainer = colorResource(R.color.purple_700),
            secondary = colorResource(R.color.teal_200),
        )
    }
    
    MaterialTheme(
        colorScheme = colorScheme,
        content = content
    )
}
```

---

## 🖼️ 图标资源

为每个风味准备不同的图标，放置在对应目录：

```
src/banana/res/
├── mipmap-hdpi/logo.png      # 72x72
├── mipmap-mdpi/logo.png      # 48x48
├── mipmap-xhdpi/logo.png     # 96x96
├── mipmap-xxhdpi/logo.png    # 144x144
└── mipmap-xxxhdpi/logo.png   # 192x192
```

`AndroidManifest.xml` 中引用：
```xml
<application
    android:icon="@mipmap/logo"
    android:roundIcon="@mipmap/logo" />
```

---

## 🔧 构建命令

```bash
# 构建所有 Debug 版本
./gradlew assembleDebug

# 构建所有 Release 版本
./gradlew assembleRelease

# 构建特定风味
./gradlew assembleFengyueRelease
./gradlew assembleHuayuRelease

# 清理后重新构建
./gradlew clean assembleRelease
```

---

## 📦 APK 输出位置

```
app/build/outputs/apk/
├── banana/
│   └── release/
│       └── 香蕉_v1.0.0.apk
│   └── debug/
│       └── 香蕉_v1.0.0_debug.apk
└── apple/
    └── release/
        └── 苹果_v1.1.0.apk
    └── debug/
        └── 苹果_v1.1.0_debug.apk
```

---

## 💡 注意事项

| 要点 | 说明 |
|------|------|
| **versionCode** | 不同风味可以独立设置版本号 |
| **包名** | 必须唯一，用于应用商店区分 |
| **签名** | 建议每个风味使用独立签名文件 |
| **资源覆盖** | 风味目录下的资源会自动覆盖 main 目录 |
| **sourceSets** | 通常不需要配置，默认合并规则已足够 |
| **BuildConfig** | 需要在 `defaultConfig` 中预定义，让 IDE 识别 |

---

## 🎯 常见问题

### Q: BuildConfig 字段找不到？

**A**: 在 `defaultConfig` 中添加默认定义，让 IDE 能识别：
```kotlin
defaultConfig {
    buildConfigField("String[]", "NODE_ARR", formatStringArray(BANANA_RELEASE_URL))
}
```

### Q: 资源没有生效？

**A**: 检查目录结构是否正确，删除手动配置的 `sourceSets`，使用默认自动合并。

### Q: 图标没有替换？

**A**: 确保图标文件名相同（如 `logo.png`），且放在正确的 `mipmap-*dpi` 目录下。

---

## 📚 参考

- [Android Product Flavors 官方文档](https://developer.android.com/studio/build/build-variants#product-flavors)
- [Gradle 构建配置](https://developer.android.com/studio/build)