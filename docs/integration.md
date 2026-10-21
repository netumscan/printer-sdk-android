# Android/iOS App 接入

[返回指南](README.md) · 适用版本 `0.3.0-preview.8` · native `0.3.0` · ABI `3/r2`

Android 最低 API 21、Java 8；iOS 最低 iOS 15。当前未发布开发版使用本地 SDK 工程或本地制品，
不要用线上旧版本替代。只有对应版本实际发布后，才可使用精确 Maven 坐标或 SwiftPM exact version；
SDK 的分发身份、校验和与运行库身份必须相符。

App 负责权限、选择设备及必要的配对；iOS 配置 `NSBluetoothAlwaysUsageDescription`。
Android 可由 SDK 按扫描得到的标准名称匹配 BLE 参数和打印 Profile；未知 BLE 名称仍需明确通信配置。
名称匹配不构成硬件认证，纸型与标签尺寸由 App 按实际耗材选择。
iOS 保持显式 BLE 参数及逐次操作 Profile，不采用 Android 的连接默认 Profile。

以下代码直接取自持续编译检查的 `LabelSession.java` 和 `AppPrinter.swift`。
Android 与当前 SDK 按 Java 8 编译，iOS 与新编译的 SDK module 按 iOS 15、Swift 5/6 类型检查。
编译验证不表示真机打印通过。

## Android 会话

将代码保存为 App 源码 `LabelSession.java`，按工程添加 package，由业务 owner 持有。
构造时提供应用 Context、ConnectionOptions、连接超时与 Ui；`spp(device)` 或 `ble(device)`
保留设备原始名称用于 SDK 匹配。连接成功后读取 `ConnectionInfo.printProfile` 与 configurationSource；
未知 Profile 时允许通信和 Raw，结构化打印须先通过 `configureProfile` 补充配置。
仅连接后空闲时可配置 Profile，成功后须按实际介质重新初始化；失败时保留旧配置。
连接与配置都不自动设置纸型，示例通过 `initializeMedia` 调用 `setMedia`。

普通打印调用 `print(PrintOptions)`；宽高可用 `labelMm` 或 `labelDots`，不再逐次传 Profile、
DPI 或最大打印宽度。示例 `printLabelMm` 使用 GAP 介质要求和下一标签定位，调用前等待介质初始化成功。
每请求独立 Callback 汇入 owner，Activity 只观察 owner 状态；连接结果与意外断连分别走
`Ui.connection` 和 `Ui.disconnected`，打印终态走 `Ui.completed`，无需 SDK 任务句柄或订阅。

```java
import android.content.Context;
import com.netumscan.printersdk.*;

/** App-only notification target. All actual control stays on the Manager. */
public final class LabelSession implements AutoCloseable {
    public interface Ui {
        void connection(ManagerResult<ConnectionInfo> result);
        void configuration(ManagerResult<ConnectionInfo> result);
        void media(ManagerResult<Void> result);
        void disconnected(ManagerResult<Void> result);
        void completed(ManagerResult<Void> result);
    }
    private final PrinterManager manager;
    private volatile Ui ui;
    private RequestId active;
    /** Pass spp(device) or ble(device) to use SDK name matching. */
    public LabelSession(Context context, ConnectionOptions connection, int timeoutMs, Ui ui) {
        manager = new PrinterManager(context); this.ui = ui;
        manager.connect(connection, timeoutMs,
            result -> { Ui target = this.ui; if (target != null) target.connection(result); },
            result -> { Ui target = this.ui; if (target != null) target.disconnected(result); });
    }
    /** Only after connection, while idle. Success invalidates previous media evidence. */
    public void configureProfile(PrinterProfile profile) {
        manager.configurePrintProfile(profile, result -> {
            Ui target = ui; if (target != null) target.configuration(result);
        });
    }
    /** The app chooses physical media; configuring a Profile performs no printer IO. */
    public void initializeMedia(MediaSettings settings, int timeoutMs) {
        manager.setMedia(settings, timeoutMs, result -> {
            Ui target = ui; if (target != null) target.media(result);
        });
    }
    /** ImageSource.deferred loaders run in the SDK background coordinator. */
    public void print(PrintOptions options) {
        active = manager.print(options, result -> {
            Ui target = ui; if (target != null) target.completed(result);
        });
    }
    /** GAP label: the adopted Profile supplies physical density and label placement. */
    public void printLabelMm(ImageSource source, double widthMm, double heightMm, int calibrationDots) {
        print(PrintOptions.labelMm(widthMm, heightMm)
            .horizontalCalibrationDots(calibrationDots)
            .addImage(source, 0, 0, ImageRender.threshold(128).containBoxMm(widthMm, heightMm))
            .nextLabel(true)
            .mediaRequirement(new MediaRequirement(MediaSettings.tracking(MediaMode.GAP),
                MediaRequirement.ALLOW_LAST_SENT))
            .build());
    }
    public void cancel() { manager.cancel(active); }
    @Override public void close() { ui = null; manager.close(result -> { }); }
}
```

图像使用 `ImageSource.pixels(PixelImage.copyOf(...))`、`AndroidImages.fromResource`、
`AndroidImages.takeFile` 或 `ImageSource.deferred`。PNG/JPEG 与 EXIF 处理由内置适配器在 SDK 后台执行；
自定义 loader 检查 `ImageLoadContext` 取消与资源预算。文件方式只接管专用、稳定的 App 缓存副本，
URI 权限与复制由 App 负责，接管后不再修改或删除该文件。
`PrintOptions` 不拥有来源的关闭责任；owner 在不再需要新提交时关闭自己的 ImageSource，
已接收请求和实际读取退出后才会释放其引用。一个请求复用同一来源只加载一次，动态 loader 可在不同提交返回不同内容。

此精简示例的 `cancel()` 只取消最近一次打印；多请求 owner 保存 RequestId 集合并按终态移除。
从选图开始到导入、提交及设备请求结束，App 应禁止换机和修改 Profile；取消、断开和关闭仍可使用。
正常换机须等待 `disconnect` 清理完成后再连接。独占 owner 结束调用 close；共享 owner 的页面只解绑观察，
Activity 重建不重放连接、初始化或打印。后台策略由 App 明确实施，回前台不自动重连或补印。

## iOS 会话

由 ViewModel 持有 `AppPrinter`，在 MainActor 上设置 showConnection、showResult 与 disconnected。
`connect` 和 `printGray8` 立即返回 RequestID；连接、打印、断开和关闭通过 completion 报告，
不使用 async/await 控制重载。示例直接 `import PrinterSDK`，`@Sendable` 回调通过 MainActor 更新 UI。

```swift
import Foundation
import PrinterSDK

// An App/ViewModel owner. Only the Manager controls the printer or owns SDK work.
@MainActor
final class AppPrinter {
    private let manager: PrinterManager
    private var requests: Set<RequestID> = []
    var showConnection: (ManagerResult<Void>) -> Void = { _ in }
    var showResult: (ManagerResult<Void>) -> Void = { _ in }
    var disconnected: (ManagerResult<Void>) -> Void = { _ in }

    init(manager: PrinterManager = PrinterManager()) { self.manager = manager }

    @discardableResult
    func connect(identifier: UUID, options: BLEOptions) -> RequestID {
        manager.connect(bleIdentifier: identifier, options: options, completion: { [weak self] result in
            self?.showConnection(result)
        }, onDisconnected: { [weak self] result in
            self?.disconnected(result)
        })
    }

    @discardableResult
    func printGray8(_ pixels: Data, width: UInt32, height: UInt32, profile: CpclProfile) -> RequestID {
        let id = manager.print(profile: profile, preparation: { cancellation in
            try cancellation.checkCancellation()
            let image = Pixels(data: pixels, width: width, height: height)
            return PreparedPrint(label: LabelJobOptions(widthDots: profile.widthDots,
                heightDots: height, dpi: profile.dpi, originXDots: Int32(profile.xOffsetDots),
                form: profile.form), elements: [.image(image)])
        }, completion: { [weak self] result in
            guard let self else { return }
            requests.remove(result.requestID)
            showResult(result) // dataSent is transport completion, not paper completion.
        })
        requests.insert(id)
        return id
    }

    func cancel(_ id: RequestID) { manager.cancel(id) }
    func cancelPrint() { for id in requests { manager.cancel(id) } }
    func disconnect(_ completion: @escaping @MainActor @Sendable (ManagerResult<Void>) -> Void = { _ in }) {
        manager.disconnect(completion: completion)
    }
    func close(_ completion: @escaping @MainActor @Sendable (ManagerResult<Void>) -> Void = { _ in }) {
        manager.close(completion: completion)
    }
}
```

像素为 GRAY8 原始 Data；每次 printGray8 显式提供 Profile。
`cancel(requestID)` 取消指定请求，cancelPrint 取消当前保存的打印请求；disconnect 完成后可显式重连，
close 永久结束独占所有权。不要在主线程同步等待这些完成回调。

共享 Manager 应由应用级所有者持有，页面离开仅解除 UI 绑定，不默认取消共享任务或关闭设备。
`DATA_SENT` 只表示发送完成，取消和失败可能已有部分输出；SDK 不自动重连、续打或重放。
