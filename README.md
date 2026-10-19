# NETUM Printer SDK for Android

`0.3.0-preview.2` 为开发预览，核心 SDK 0.3.0，C ABI 3/r1。
契约已经冻结；集成时核对精确 Maven Central 版本与运行库身份，文档本身不作为在线发布证明。

[客户指南](docs/README.md) 覆盖权限、API 生命周期、设备配置、错误码、兼容诊断和迁移。
ABI3 不兼容 ABI2；使用新的 Java API 和同一版本 AAR，不能混用旧 native 库。

在应用工程中使用已发布的精确版本：

```kotlin
dependencyResolutionManagement {
    repositories { google(); mavenCentral() }
}
// Application module:
dependencies {
    implementation("com.netumscan:printer-sdk-android:0.3.0-preview.2")
}
```

要求 Android 5.0 / API 21+、Java 8；支持 `arm64-v8a`、`armeabi-v7a`、`x86_64`。
AAR 提供 25 个 C 导出、23 个 JNI 导出及 consumer R8 规则，静态链接私有 C++ 运行库。
应用应保留其他依赖所需的共享运行库。

应用使用 `PrinterManager` 连接、释放和关闭的 completion overload，
使用 `MobilePrinter.observeState` 查看连接、工作和队列状态，
使用 `PrintOperation.onComplete` 接收终态；默认回调派发到主线程。
每次打印显式传入经硬件确认的 profile，不猜测打印宽度、蓝牙 UUID 或完成协议。
`PrinterImage` / `PrintJob` 支持复用像素和多图任务；构建后不可变，使用后需关闭调用方句柄。
`Subscription.close()` 仅注销通知，`PrintOperation.close()` 仅释放访问权；
取消任务必须调用 `PrintOperation.cancel()`。

SPP 使用经典蓝牙 RFCOMM；BLE 需使用设备对应的服务及写特征 UUID。
权限申请、用户选设备和生命周期由应用管理，详细要求见[接入与权限](docs/integration.md)。
硬件状态没有已验证协议时显示未知，状态查询返回不支持且不发送猜测指令。
数据发送完成不证明纸张打印完成；断线后不会自动重连或重放任务。

本目录和 Maven classifier JAR 仅含分发文档与许可信息，不含 SDK 实现源码。
许可范围见 [LICENSE](LICENSE)。预览候选尚未完成全部硬件验收。
