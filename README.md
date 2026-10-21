# NETUM Printer SDK for Android

`0.3.0-preview.8` 为开发预览，核心 SDK 0.3.0，C ABI 3/r2。
当前移动 API 通过独立 `PrinterManager` 管理一个连接，输入与结果都是数据对象。
开发身份不作为在线发布证明；集成时核对精确版本、制品摘要与运行库身份。

[客户指南](docs/README.md) 覆盖权限、请求生命周期、配置、错误码、兼容诊断和迁移。
Manager 移动 API 与旧移动 API 不兼容，需重新编译；C ABI3/r2 仍保留原 native 契约。

当前未发布开发版使用本地 SDK 工程或本地 AAR。对应版本实际发布后，可使用精确坐标：

```kotlin
dependencyResolutionManagement {
    repositories { google(); mavenCentral() }
}
dependencies {
    implementation("com.netumscan:printer-sdk-android:0.3.0-preview.8")
}
```

要求 Android 5.0 / API 21+、Java 8，支持 `arm64-v8a`、`armeabi-v7a`、`x86_64`。
AAR 提供 27 个 C 导出、23 个 JNI 导出及 consumer R8 规则，静态链接私有 C++ 运行库。
应用保留其他依赖所需的共享运行库，不能混用旧 native 库。

应用创建独立 `PrinterManager(context)`，通过 connect、print、sendRaw、setMedia、queryStatus、
disconnect 和 close 发起请求。请求立即返回 RequestId，完成回调异步派发到主线程；
参数、状态、准备、准入与执行失败统一从 `ManagerResult` 返回。

连接使用 `ConnectionOptions.spp(device)` 或 `ble(device)` 保留扫描得到的原始名称，
SDK 为已知名称匹配 Profile；以成功回调的 `ConnectionInfo.printProfile` 为准。
未知或需更正的配置，在连接后空闲时调用 `configurePrintProfile`，不提供连接前 Profile 或单次打印覆盖。
配置成功不发送介质命令；App 选择实际纸型后，显式调用 `setMedia` 初始化。

打印调用 `manager.print(PrintOptions, Callback)`。`PrintOptions` 描述标签／小票宽高、图像、
空白、份数、超时与校准；`ImageSource` 提供像素、资源、稳定缓存文件或后台 loader，
逐图 `ImageRender` 指定缩放、阈值／抖动和反色。已知 Profile 的 DPI 与最大打印宽度无需重复传入。
LT-M90 使用精确 8 dots/mm，50 × 50 mm 对应 400 × 400 dots；SDK 按硬件居中规则计算起点，
App 仅传标签尺寸与可选校准。详情见[连接与打印配置](docs/configuration.md)。

每请求独立 Callback，由 owner 集中更新状态，Activity 观察 owner；连接结果与意外断连单独处理。
取消使用 `manager.cancel(requestId)`，断开和关闭通过各自 Callback 等待清理屏障，
不要同步等待主线程回调。选图／导入期间由 App 禁止换机与修改 Profile；SDK 从请求接纳起保护排队与执行工作。
应用不维护 SDK 轮询器、IO 队列、订阅或 native 任务句柄。

SPP 使用经典蓝牙 RFCOMM。已知名称可匹配 BLE 参数，未知名称需显式提供服务、特征和流控配置。
名称匹配不是硬件身份认证。权限、用户选设备和前后台策略由应用负责，见[接入与权限](docs/integration.md)。
未验证硬件状态显示未知，查询不发送猜测指令；DATA_SENT 不证明实际打印完成。
SDK 不自动重连、续打、重试或重放。

本目录和 Maven classifier JAR 仅含分发文档与许可信息，不含 SDK 实现源码。
许可范围见 [LICENSE](LICENSE)，`hardware_verified=false`。
