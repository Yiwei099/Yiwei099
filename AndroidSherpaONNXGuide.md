# Sherpa-ONNX 语音识别与标点恢复 Android 集成文档

## 一、概述

本文档介绍如何在 Android 应用中集成基于 Sherpa-ONNX 的离线语音识别（ASR）和标点恢复功能。该方案完全离线运行，无需网络连接。

### 技术栈

| 组件          | 版本/规格                                        | 说明                                            |
|-------------|----------------------------------------------|-----------------------------------------------|
| Sherpa-ONNX | 1.13.2                                       | 核心推理框架，需要使用静态链接版本(aar内不包含 libonnxruntime.so ) | 
| ASR 模型      | streaming-zipformer-zh-int8 (2025-06-30)     | 流式中文语音识别                                      |
| 标点模型        | punct-ct-transformer-zh-en-int8 (2024-04-12) | 中英文标点恢复                                       |

### 最终 APK 体积

- **基础 APK**：~42 MB（不含模型文件）
- **Sherpa 引擎**：~21 MB（静态链接版 AAR）
- **总增量**：~63 MB

> 模型文件（~75 MB）采用动态下载方案，不打包进 APK。

---

## 二、环境要求

| 项目             | 要求                    |
|----------------|-----------------------|
| Android Studio | Hedgehog \| 2023.1.1+ |
| minSdk         | 26 (Android 8.0)      |
| targetSdk      | 34                    |
| NDK            | 已配置（用于 ABI 过滤）        |
| 架构支持           | arm64-v8a（主流手机）       |

---

## 三、集成步骤

### 3.1 添加 AAR 依赖

1. 从 [GitHub Releases](https://github.com/k2-fsa/sherpa-onnx/releases) 下载静态链接版 AAR：
    - 文件名示例：`sherpa-onnx-static-link-onnxruntime-1.13.2.aar`
    - 体积约 21-25 MB（不含独立 ONNX Runtime）

2. 将 AAR 文件放入 `app/libs/` 目录

3. 配置 `app/build.gradle.kts`：

```kotlin
android {
    defaultConfig {
        ndk {
            // 只打包 arm64-v8a 架构
            abiFilters.add("arm64-v8a")
        }
    }
}

dependencies {
    // 本地 AAR 依赖
    implementation(fileTree(mapOf("dir" to "libs", "include" to listOf("*.aar"))))
}
```

4. 在 `AndroidManifest.xml` 中添加权限：

```xml
<uses-permission android:name="android.permission.RECORD_AUDIO" />
<!--可选-->
<uses-feature android:name="android.hardware.microphone" android:required="false" />
```

---

## 四、模型文件配置

### 4.1 模型文件清单

| 功能       | 文件名                 | 大小     | 来源                                                                         |
|----------|---------------------|--------|----------------------------------------------------------------------------|
| ASR 编码器  | `encoder.int8.onnx` | ~7 MB  | sherpa-onnx-streaming-zipformer-zh-int8-2025-06-30.tar.bz2                 |
| ASR 解码器  | `decoder.onnx`      | ~4 MB  | sherpa-onnx-streaming-zipformer-zh-int8-2025-06-30.tar.bz2                 |
| ASR 联合网络 | `joiner.int8.onnx`  | ~4 MB  | sherpa-onnx-streaming-zipformer-zh-int8-2025-06-30.tar.bz2                 |
| ASR 词表   | `tokens.txt`        | ~1 MB  | sherpa-onnx-streaming-zipformer-zh-int8-2025-06-30.tar.bz2                 |
| 标点模型     | `model.int8.onnx`   | ~73 MB | sherpa-onnx-punct-ct-transformer-zh-en-vocab272727-2024-04-12-int8.tar.bz2 |

> **模型选型说明**：1.选 int8；2.选日期较新

### 4.2 模型文件放置（本地测试）

如需要本地测试，将模型文件放入 `app/src/main/assets/models/` 目录：

```
app/src/main/assets/
└── models/
    ├── encoder.int8.onnx
    ├── decoder.onnx
    ├── joiner.int8.onnx
    └── tokens.txt
└── punctuation/
    └── model.int8.onnx      # 标点模型
```

> **生产环境建议**：将模型文件托管到服务器，用户首次使用时动态下载。

---

## 五、参考链接
- [在这里获取官方最新发布的 aar](https://github.com/k2-fsa/sherpa-onnx)
- [在这里获取官方所有构建好的资料(demo/预构建模型等)](https://pkg.go.dev/github.com/k2-fsa/sherpa-onnx)
- [在这里获取官方最新发布的ASR预训练模型](https://github.com/k2-fsa/sherpa-onnx/releases/tag/asr-models)
- [在这里获取官方最新发布的标点恢复模型](https://github.com/k2-fsa/sherpa-onnx/releases/tag/punctuation-models)

---