# Tinker 接入指南

官方wiki: https://github.com/Tencent/tinker/wiki/Tinker-%E6%8E%A5%E5%85%A5%E6%8C%87%E5%8D%97
IDE: IntelliJ IDEA 2021.3.3

step1: 编译,安装
```shell
./gradlew assembleDebug
adb install ./app/build/bakApk/app-debug-0325-16-06-02.apk
```

step2: 修改源代码，打包、推送
- 修改源代码

- 修改配置
  修改 `./app/build.gradle` 中的配置， `tinkerOldApkPath` 的值改为 step1 中的安装包。

- 编译
  ```shell
  ./gradlew tinkerPatchDebug
  adb push ./app/build/outputs/apk/tinkerPatch/debug/app-debug-patch_signed_7zip.apk /sdcard/Download/patch_signed_7zip.apk
  ```

- 在app中点击loadPatch，选择上一步推送的安装包。等待（注意时间比较久）。

---

## 为让 tinker-demo 运行起来所做的修改

### 1. 修复软件源（`tinker-sample-android/build.gradle`）

原项目使用已下线的 `jcenter()` 作为依赖仓库，导致构建时无法下载依赖。

- 将 `jcenter()` 替换为 `mavenCentral()`
- 新增阿里云国内镜像加速：`maven { url 'https://maven.aliyun.com/repository/public' }`

### 2. 升级编译环境（`app/build.gradle`）

- Java 版本从 `VERSION_1_7` 升级到 `VERSION_1_8`
- `compileSdkVersion` 从 28 升级到 30
- `androidx.appcompat` 从 `1.1.0` 升级到 `1.3.0`
- 新增 `androidx.activity:activity:1.2.0` 依赖（用于文件选择器 API）

### 3. 使用本地安装的 7za（`app/build.gradle`）

原项目通过 Maven 下载 SevenZip 工具包（`com.tencent.mm:SevenZip:1.1.10`），在 macOS Apple Silicon 上无法正常使用。

- 注释掉 `zipArtifact`（Maven 方式）
- 改为指定本地 7za 路径：`path = "/opt/homebrew/bin/7za"`
- macOS 安装方式：`brew install p7zip`

### 4. 使用文件选择器加载补丁包（`MainActivity.java`）

原实现通过硬编码路径（`Environment.getExternalStorageDirectory()`）读取外部存储的 patch 文件，在 Android 10+ 上因存储权限收紧而无法访问。

- 使用 `ActivityResultLauncher<String[]>` + `ActivityResultContracts.OpenDocument` 打开系统文件选择器
- 将用户选择的文件通过 `ContentResolver` 复制到应用私有缓存目录（`getCacheDir()`）后再传给 Tinker，无需申请外部存储权限
- `adb push` 到 `/sdcard/Download/` 后，在手机文件管理器中选择该文件即可触发加载
