# Termora 项目打包与部署指南

本指南详细介绍了如何在 macOS、Windows 和 Linux 三大操作系统上对 Termora 项目进行打包，以及各平台的打包图标（icon）修改路径和注意事项。

---

## 目录
1. [前提条件与注意事项](#一前提条件与注意事项)
2. [多平台通用打包命令](#二多平台通用打包命令)
3. [各操作系统打包详解](#三各操作系统打包详解)
4. [应用图标（Icon）修改路径](#四应用图标icon修改路径)

---

## 一、前提条件与注意事项

在开始打包之前，请确保本地开发环境满足以下要求：

1. **JDK 版本要求**：
   * 本项目使用 Java 25 编译器。请确保您的系统安装了 **JDK 25**（如 Oracle OpenJDK 25 或 JetBrains Runtime 25），并且 `JAVA_HOME` 环境变量已正确指向该 JDK 路径。

2. **Gradle 任务隐式冲突（非常重要）**：
   * 某些外部执行任务（如 `jlink`、`jpackage`）以及插件子模块在并发构建时存在 Gradle 的隐式依赖冲突。
   * **不要在一行命令中直接执行 `clean` 和打包，或者将 `jar` 和 `copy-dependencies` 并发执行**，否则会导致构建失败。
   * 必须采用**分步串行**的方式进行构建（详见[通用打包命令](#二多平台通用打包命令)）。

3. **外部缓存残留问题**：
   * 由于 `jlink` 和 `jpackage` 任务会调用系统的底层打包工具在 `build` 目录下生成外部缓存文件，普通的 `./gradlew clean` 无法将其完全清理。
   * 若打包报错提示 *“directory already exists”* 或 *“temp must be non-existent or empty”*，请在打包前手动在终端运行：
     ```bash
     rm -rf build
     ```

---

## 二、多平台通用打包命令

为避免冲突并确保 100% 打包成功，请在项目根目录下，**严格按照以下顺序**分步执行终端命令：

```bash
# 1. 彻底清除旧构建缓存（包括底层工具生成的物理残留）
rm -rf build

# 2. 编译主程序和子插件，生成主 jar 文件
./gradlew jar

# 3. 拷贝并解压各平台专用的第三方依赖包
./gradlew copy-dependencies

# 4. 生成精简的定制 JRE 运行时环境，并生成安装包以及分发归档
./gradlew jlink jpackage dist
```

执行完毕后，所有的最终安装包和分发文件均会输出在项目的 **`build/distributions/`** 目录中。

---

## 三、各操作系统打包详解

### 1. macOS 打包
* **默认产物**：会在 `build/distributions/` 下生成 `.dmg` 安装镜像以及免安装的 `.zip` 压缩包。
* **依赖工具**：需要 macOS 系统自带的 `ditto`（用于打包 zip 格式）。
* **签名与公证（选填）**：
  若需要官方签名和公证以避免 macOS 系统拦截，请在执行打包前，在终端中配置对应的环境变量：
  ```bash
  export TERMORA_MAC_SIGN=true
  export TERMORA_MAC_SIGN_USER_NAME="您的开发者签名证书用户名"
  export TERMORA_MAC_NOTARY=true
  export TERMORA_MAC_NOTARY_KEYCHAIN_PROFILE="您的公证 Keychain 配置名"
  ```

### 2. Windows 打包
* **默认产物**：默认生成 `app-image` 便携版本，并进一步封装成 `.zip` 归档以及 `.exe` 安装程序。
* **依赖工具**：
  * 系统需安装 **Inno Setup** 编译器（确保 `iscc` 命令可用），用来生成 `.exe` 安装引导。它会自动读取 `src/main/resources/termora.iss` 脚本。
  * 如需生成 UWP/MSIX 格式，需要系统包含 `makeappx` 工具，并通过环境变量配置其路径：
    ```cmd
    set MAKEAPPX_PATH=C:\Program Files (x86)\Windows Kits\10\bin\...\makeappx.exe
    set TERMORA_TYPE=appx
    ```

### 3. Linux 打包
* **默认产物**：默认生成 `tar.gz` 绿色压缩包、`AppImage` 单文件版，以及可供 Debian/Ubuntu 使用的 `.deb` 包。
* **依赖工具**：
  * **AppImage**：首次打包会通过 `wget` 自动在 `.gradle` 目录下下载 `appimagetool`。请确保系统已安装 `wget` 并具备联网权限。
  * **DEB 安装包**：如需打包为 deb 文件，请提前设置环境变量：
    ```bash
    export TERMORA_TYPE=deb
    ```

---

## 四、应用图标（Icon）修改路径

如果您需要更换 Termora 在各个系统打包后的程序图标，请直接替换对应路径下的图片文件：

| 操作系统 | 图标格式 | 项目内文件路径 | 尺寸与说明 |
| :--- | :--- | :--- | :--- |
| **macOS** | `.icns` | [src/main/resources/icons/termora.icns](file:///Users/mba/Projects/termora/src/main/resources/icons/termora.icns) | macOS 专属的多尺寸图标资源 |
| **Windows** | `.ico` | [src/main/resources/icons/termora.ico](file:///Users/mba/Projects/termora/src/main/resources/icons/termora.ico) | Windows 应用程序和桌面快捷图标 |
| **Windows Inno**| `.bmp` | [src/main/resources/icons/termora_128x128.bmp](file:///Users/mba/Projects/termora/src/main/resources/icons/termora_128x128.bmp) | Inno Setup 安装引导窗的位图展示 |
| **Linux** | `.png` | [src/main/resources/icons/termora.png](file:///Users/mba/Projects/termora/src/main/resources/icons/termora.png) | 桌面应用快捷图标（基本尺寸） |
| **Linux HD** | `.png` | [src/main/resources/icons/termora_256x256.png](file:///Users/mba/Projects/termora/src/main/resources/icons/termora_256x256.png) | AppImage 高清打包图标 (256x256 像素) |

> 💡 **提示**：替换上述文件时，请保持文件名与文件格式完全一致，随后重新执行通用打包命令即可。
