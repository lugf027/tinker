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
