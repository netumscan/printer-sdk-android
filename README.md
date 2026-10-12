# NETUM Printer SDK for Android

NETUM 打印机 Android SDK 的闭源二进制分发入口。当前版本为开发预览版
`0.2.0-preview.3`，核心 SDK 0.2.0，C ABI 2/r10。

[客户指南](docs/README.md) 覆盖权限、API 生命周期、纸张配置、错误码、兼容诊断、迁移和安全反馈。
可运行工程见 [Quick Start / Full Demo](https://github.com/netumscan/printer-sdk-samples)。

在应用工程的仓库配置中加入 Maven Central：

```kotlin
dependencyResolutionManagement {
    repositories { google(); mavenCentral() }
}
```

在应用模块中添加依赖：

```kotlin
dependencies {
    implementation("com.netumscan:printer-sdk-android:0.2.0-preview.3")
}
```

要求 Android API 26+；Java 8；支持 `arm64-v8a`、`armeabi-v7a`、`x86_64`。
AAR 包含 JNI、原生 C 库、`libc++_shared.so` 和必要的 consumer R8 规则。
与其他原生 SDK 同时集成时，应在应用侧统一选择兼容的 C++ 运行库。

最小接入示例（在应用工作线程执行连接与关闭）：

```java
import com.netumscan.printersdk.CpclProfile;
import com.netumscan.printersdk.MobilePrinter;
import com.netumscan.printersdk.PrintOperation;

CpclProfile profile = new CpclProfile(203, 576, 4096, true);
try (MobilePrinter printer = MobilePrinter.spp(context, address, profile)) {
    printer.connect(10000);
    try (PrintOperation operation = printer.submitLabelGray8(
            gray8, width, height, 1, 0, 0, 0, true, 120000)) {
        while (!operation.snapshot().terminal()) {
            Thread.sleep(20);
        }
        // operation.close() 释放结果；不会取消已入队的打印。
    }
}
```

上述几何参数只是配置示例，必须按设备和纸张能力填写。应用负责蓝牙权限、
用户授权和目标设备选择。Android 12+ 连接需要 `BLUETOOTH_CONNECT`，扫描还需要
`BLUETOOTH_SCAN`；SPP 连接会取消系统发现，因此也需要 `BLUETOOTH_SCAN`。
Android 8–11 BLE 扫描需位置权限和相应系统设置。完整策略见[接入与权限](docs/integration.md)。
打印任务入队后异步执行；数据发送完成不代表纸张打印完成。连接中断后不会自动续打。

`MobilePrinter.feature(int)` 返回 `FeatureSupport` 的实现、设备支持和证据三字段；
旧 `implementsFeature(int)` 保持原义。实现非零不证明硬件支持，未知与明确不支持应区分。
详见 [FeatureSupport](docs/api-lifecycle.md#featuresupport)。

预览版尚未完成全部型号、固件、故障和手机峰值内存验收，不承诺生产稳定性。
本仓库和 Maven classifier JAR 只含分发文档及许可信息，不含 SDK 实现源码。
免费商用接入、再分发边界和限制以 [LICENSE](LICENSE) 为准。

发布后，AAR、POM、Gradle module、签名和校验文件由 Maven Central 提供。
本仓库保存接入文档、许可证及 `releases/v0.2.0-preview.3/` 中的版本化 POM，
发布标签为 `v0.2.0-preview.3`。
