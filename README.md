# NETUM Printer SDK for Android

开发预览 **`0.3.0-preview.20`**，native **`0.3.0`**、C ABI **`3/r3`**。
支持 Android API 21+、Java 8，AAR 包含 `arm64-v8a`、`armeabi-v7a`、`x86_64`。
本版使用统一 `PrinterListener`，全部16个回调都是必需方法，没有默认空实现。Java／Kotlin 调用者必须显式实现每个方法，未使用的动作或观察也应在 App 中明确留空；移动调用方须迁移并重新编译。
Android 包装和 native 库应成套使用同一分发版本。许可和第三方声明见 [LICENSE](LICENSE) 和 [LICENSE-NOTICE.md](LICENSE-NOTICE.md)。

[客户指南](docs/README.md) 提供双平台接入、生命周期、迁移、错误码和完整公开 API。
冻结候选本身不表示已上传，接入时核对精确版本、制品 SHA-256 与实际运行库身份。

## 依赖

实际发布后使用精确 Maven 坐标；发布前使用已验证的本地制品：

```kotlin
dependencyResolutionManagement {
    repositories { google(); mavenCentral() }
}
dependencies {
    implementation("com.netumscan:printer-sdk-android:0.3.0-preview.20")
}
```

SDK 静态链接私有 C++ 运行库并提供 consumer R8 规则，不分发私有 SDK 实现源码。

## 统一接收者与实时扫描

由稳定的 App owner 完整实现 `PrinterListener`，先注册，再提交请求。编译器拒绝空实现或遗漏任何一个方法；以下完整示例明确保留未使用动作的空方法。操作立即返回 `RequestId`，
按动作分别交付 `ManagerResult`；结果保留原始请求 ID、outcome、code、message、completion 和 `outputMayHaveOccurred`。
以下示例只有用户调用方法才扫描、连接或打印，候选发现不会自动连接：

```java
import android.content.Context;
import com.netumscan.printersdk.*;
import java.util.ArrayList;
import java.util.List;

final class PrinterSession implements PrinterListener {
    // Unused callbacks are explicit: this owner does not submit or observe these actions.
    @Override public void onBondedDevicesCompleted(ManagerResult<java.util.List<DiscoveredDevice>> result) { }
    @Override public void onModelConfigurationCompleted(ManagerResult<ConnectionInfo> result) { }
    @Override public void onPaperTypeCompleted(ManagerResult<Void> result) { }
    @Override public void onStatusQueryCompleted(ManagerResult<StatusResult> result) { }
    @Override public void onLabelPositioningCompleted(ManagerResult<Void> result) { }
    @Override public void onRawCompleted(ManagerResult<Void> result) { }
    @Override public void onCapabilitiesCompleted(ManagerResult<PrinterCapabilities> result) { }
    @Override public void onSdkInfoCompleted(ManagerResult<SdkInfo> result) { }
    @Override public void onDisconnectCompleted(ManagerResult<Void> result) { }

    private final PrinterManager manager;
    private RequestId scan;
    private List<DiscoveredDevice> devices = new ArrayList<>();

    PrinterSession(Context context) {
        manager = new PrinterManager(context);
        manager.setListener(this);
    }
    void scan() {
        if (scan != null) return;
        devices.clear();
        scan = manager.scanBle();
    }
    void stopScan() { if (scan != null) manager.stopScan(scan); }
    @Override public void onDeviceDiscovered(RequestId scanId, DiscoveredDevice device) {
        if (scan != scanId) return;
        devices.add(device); // Update the visible list while scanning.
    }
    @Override public void onScanCompleted(ManagerResult<List<DiscoveredDevice>> result) {
        if (scan != result.requestId) return;
        scan = null;
        if (result.outcome == ManagerResult.Outcome.SUCCESS && result.value != null) {
            devices = new ArrayList<>(result.value); // Final merged names and RSSI.
        }
        // Display cancellation/failure separately; do not report partial candidates as success.
    }
    RequestId connect(DiscoveredDevice device) {
        // Call after scan completion; confirm the physical model before choosing LT_M90.
        return manager.connectBle(device, PrinterModel.LT_M90);
    }
    @Override public void onConnectionCompleted(ManagerResult<ConnectionInfo> result) {
        // Keep the original result; printerInfo is optional.
    }
    RequestId print(byte[] pngJpeg) { return manager.printEncodedImage(pngJpeg, 50, 50); }
    @Override public void onPrintCompleted(ManagerResult<Void> result) {
        // DATA_SENT confirms transfer, not physical paper completion.
    }
    @Override public void onStatusChanged(StatusResult status) {
        // Unknown optional values remain unknown.
    }
    @Override public void onDisconnected(ManagerResult<Void> result) {
        // Associate an unexpected loss with result.requestId, the connection request ID.
    }
    RequestId close() { return manager.close(); }
    @Override public void onCloseCompleted(ManagerResult<Void> result) {
        // The App may release this owner after cleanup.
    }
}
```

App 保存同一个 owner、Manager 和请求 ID，所有方法及通知均按公共 API 的主线程约定使用。
`onDeviceDiscovered` 按本轮扫描 ID 通知首次唯一设备，`onScanCompleted` 提供最终合并列表。
App 可边扫边展示候选。正常停止使用 `stopScan(scanId)`，由原 `onScanCompleted` 成功返回最新合并列表（可为空），仍用原请求 ID；`cancel(scanId)` 保持取消且无列表。重复、外部、已完成或非扫描 ID 无操作。选择、连接和打印由用户分别触发。
连接不会自动停止扫描。Android 经典蓝牙使用 `bondedDevices` 读取已配对设备，再显式 `connectSpp`；
权限、系统配对和用户选择由 App 处理，SDK 不暴露平台句柄。

Manager 强持有当前 Listener；`setListener(null)` 解除，替换或解除使旧注册尚未交付的通知失效。
通知只向仍有效接收者至多交付一次，不补交历史；解除不取消任务，已进入的通知可以结束。
要观察关闭结果，必须保留 owner 和接收者直到 `onCloseCompleted`；关闭终态交付后 Manager 释放接收者。
不要在应用线程同步等待通知。

## 图片、设备与生命周期

`printImage(bitmap, widthMm, heightMm)` 和 `printEncodedImage(pngJpeg, widthMm, heightMm)`
直接接收图片和页面毫米宽高，不传逐调用回调，也不要求先生成预览。编码入口接收完整 PNG/JPEG，
不接收 Base64 或裸 RGBA。Bitmap／byte[] 在返回前取得独立副本，复制期间不得并发修改；返回后可释放 App 副本。
解码、方向修正、透明白底和等比缩小居中补白在后台执行；不放大、不裁剪，固定一份、阈值 128、不反色。
打印不会隐式设置纸型、FORM、定位或切纸；`setPaperType` 和 `locateNextLabel` 是独立请求。

设备名称可推荐内置 `PrinterModel`；已确认型号的改名设备可显式指定，未知型号不猜通信参数。
每次连接自动读取可选 `ConnectionInfo.printerInfo`；`printerInfoReadStatus` 区分 `NOT_ATTEMPTED`／`SUCCEEDED`／`FAILED`。仅失败时 `printerInfoReadCode` 和 `printerInfoReadMessage` 有值，保留本次稳定错误和原因；未尝试与成功时为空。普通读取失败保留可用连接和打印；查询隔离策略不变，后续查询以自身结果为准，不自动重试。换型号保留诊断，重连重新读取并清空旧诊断。
`getStatus()` 的 `onStatusQueryCompleted` 返回本次 ManagerResult<StatusResult>，成功 value 来自原生已校验响应；独立状态变化仍经 `onStatusChanged` 更新；
四项状态均可未知，LT-M90 的开盖／缺纸可由有效查询或主动上报确认，纸将尽／切刀错误仍未知。
查询写出后的超时、取消或坏回复保守断开以隔离迟到回复，App 显式重连，不自动重试。

所有打印、Raw、纸型、走纸和查询共用同一设备 FIFO。`cancel(id)` 表达取消意图，仍等待真实终态；
`disconnect()` 清理后可显式重连，`close()` 永久结束 Manager。SDK 不自动重连、续打或重放；
失败、取消及 `DATA_SENT` 均不能证明机械打印完成。
Flutter 使用标准 codec 将完整 `Uint8List` 转为 `byte[]`，无需 Base64 或插件重复解码。
软件验证不等于真实无线、纸面或峰值内存验收，`hardware_verified=false`。

