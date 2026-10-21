# 迁移到 Manager

[返回指南](README.md) · 适用版本 `0.3.0-preview.8`

当前移动 API 以 `PrinterManager` 作为唯一控制入口。这是 Java/Swift API 的不兼容迁移，
C ABI 仍为 native `0.3.0` / ABI `3/r2`；既有 r1/r2 C 消费者不会因移动类型隐藏而失去导出入口。
旧在线制品、公开样例、候选快照和历史证明继续保留。

## Android

1. 由业务 owner 创建独立 PrinterManager，使用 `ConnectionOptions.spp(device)`／`ble(device)`
   保留标准设备名。连接成功读取 ConnectionInfo；SDK 自动匹配 Profile，未知或需更正时只在连接后空闲
   调用 `configurePrintProfile`。移除连接前 Profile、单次覆盖与旧显式 Profile 操作重载。
2. 将旧 `print(profile, preparation, ...)` 改为 `print(PrintOptions, Callback)`。
   `PrintOptions.labelMm`／`labelDots` 或小票工厂固定布局、图像、空白、份数、超时与介质要求；
   自定义解码迁至 `ImageSource.deferred`，也可采用 PixelImage 或 AndroidImages 的资源／缓存文件入口。
   逐图缩放、阈值／抖动与反色使用 ImageRender，loader 检查 ImageLoadContext 取消及资源预算。
3. 已知 Profile 的 DPI、最大宽度和精确点密度不再逐次传入；mm 换算及预览标尺统一使用 Profile。
   LT-M90 的居中由 SDK 计算，移除 App 重复计算的横向起点；传标签宽高与可选校准，旧绝对 x 不转换为校准。
4. Profile 配置与实际介质初始化分开；配置成功后按纸型调用 setMedia，必要时以 MediaRequirement
   绑定打印。配置成功后介质失败保留新 Profile，可单独重试初始化；不要把连接成功当作介质已就绪。
5. 每请求一个 Callback 汇入 owner，保存 RequestId 集合并在终态移除；Activity 观察状态，
   连接与意外断连单独处理。ImageSource 的 App 引用由 owner 关闭，PrintOptions 本身不需要 close。
6. 从选图／导入开始保护换机与配置，SDK 从请求接纳起继续保护排队、准备和执行。
   Activity 重建不重放请求，真正结束独占 owner 时异步 close；正常换机等待 disconnect 完成后再 connect。

## iOS 与共用规则

iOS 保持逐操作 Profile 与 preparation → PreparedPrint；后台准备检查 CancellationContext，
主线程完成和断连闭包保持 `@MainActor @Sendable`。不采用 Android 的连接默认 Profile 或 PrintOptions。

两端均以请求标识和完成回调管理每次操作，读取 ManagerResult 的 outcome/code/completion。
取消调用 `manager.cancel(requestID)`；断开和永久关闭使用各自异步完成回调，不同步等待主线程。
页面退出只解除自己的 UI 绑定，共享 Manager 由应用级 owner 决定何时关闭。
删除应用自己的 SDK 任务轮询和 IO 队列，不公开或传递 native image/job/operation 句柄。
区分 dataSent 与 deviceConfirmed，硬件状态未验证时显示未知；遇到断连、超时或取消不自动重放。

Raw、介质、走纸和查询同样经过 Manager 进入 native 设备队列，不增加旁路 IO。
完整编译声明与数据 DTO 见 [Java API](android-java-api.md) 和 [Swift API](ios-swift-api.md)，
经过编译检查的应用会话代码见[接入](integration.md)。
