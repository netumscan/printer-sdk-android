# API 与生命周期

[返回指南](README.md) · 适用版本 `0.3.0-preview.8`

通过构造器创建独立 `PrinterManager`，每个实例至多管理一个连接，不使用共享单例。
Android 从成功连接结果采用默认 Profile，未知或需更正时连接后空闲调用 `configurePrintProfile`；
普通打印使用 `PrintOptions`，不逐次覆盖 Profile。iOS 保持逐操作 Profile 和 `PreparedPrint`。
输入描述不持有可操作的 native 句柄，应用无需释放 image/job/operation 或维护 SDK 订阅。

连接、扫描、打印、控制、查询、断开和关闭立即返回请求标识，并异步在主线程完成一次回调。
参数、状态、准备、准入及执行失败均从同一 `ManagerResult` 通道返回。
数据对象的构造器校验错误仍由应用处理，回调参数须有效；不要阻塞主线程等待完成。

Android 的 ImageSource 加载与 ImageRender 处理在 SDK 后台执行，自定义 loader 检查
`ImageLoadContext` 的取消和资源预算；iOS preparation 检查 `CancellationContext` 并返回纯数据 PreparedPrint。
不要自行创建打印 IO 队列或轮询器。Manager 按请求顺序进行准备和准入，
实际设备 IO 仍由同一个 native FIFO 串行执行。总超时覆盖等待、加载／准备、准入及执行，不在开始发送时重置。
最多接纳 64 个未完成业务请求，超过限制通过完成回调返回 `1008`。

Android 的不可变 PrintOptions 不拥有 ImageSource 的关闭责任。owner 在不再新提交时关闭自己的来源引用；
SDK 已接收的工作保留读取引用，来源真正无人使用后才释放缓存等资源。取消终态不保证 loader 已退出，
退出前继续计入工作集与 loader 数量上限；迟到结果不再准入，也不能绕过预算启动额外线程。

`ManagerResult` 包含请求标识、成功/失败/取消、稳定错误码、消息、完成证据、
`outputMayHaveOccurred` 和可选业务值。`dataSent` 仅证明传输完成；只有协议提供明确证据时才有
`deviceConfirmed`。移动 API 不公开阶段、队列快照、native operation ID、连接代次或资源快照。

每请求一个 Callback，由 owner 根据 RequestId 更新状态和 pending 集合；Activity 观察 owner 快照，
不保管 SDK Callback，也不在 render 或恢复时重新提交。FIFO 不保证被取消请求的终态回调顺序。
连接成功、Profile 配置和介质初始化各自报告；意外断连通过独立通知交付，主动断开只使用对应完成回调。
配置成功只更新本地默认值，介质初始化失败时保留新 Profile，允许单独重试初始化。

| 动作 | 生命周期效果 |
| --- | --- |
| `manager.cancel(requestID)` | 请求取消等待、准备、准入或执行中的本实例请求；外部实例或已完成标识无效。 |
| Android `manager.configurePrintProfile(profile, callback)` | 只在连接后空闲时原子更新本地配置，不做 IO；忙时失败并保留旧值，成功使旧介质证据失效。 |
| `manager.disconnect(completion)` | 结束本会话请求并释放设备；清理完成后可显式连接另一台设备。 |
| `manager.close(completion)` | 幂等地永久关闭 Manager；清理完成后业务请求返回 InvalidState。 |

断开和关闭的完成回调在本会话请求终态安排交付、设备和扫描资源释放后交付。
清理不依赖应用准备函数合作返回；迟到的准备结果不会再次准入。意外断连在旧设备清理和受影响请求终态安排后，
仅通知所属连接；owner 无需再重复 disconnect，旧连接事件不能影响后续连接。
取消只是请求，最终结果仍以 Callback 为准；disconnect／close 不可取消，重复调用各自等待同一清理屏障。
SDK 不自动重连、续打、重试或重放。

SDK 从接纳请求起保护连接与 Profile，App 另从选图／导入开始禁止换机与修改配置，
避免把选图开始前的内容或尺寸交给另一台设备。取消与清理不受此限制；正常换机先等待无选图、导入或设备工作，
再 disconnect 并等待清理完成。sdkInfo 等本地独立查询单独跟踪，不必占用 App 的设备工作锁；
queryStatus 等设备请求仍按设备工作处理。

Swift 完成和断连回调声明为 `@MainActor @Sendable`，MainActor owner 可直接更新其状态；
preparation 仍为后台 `@Sendable` 闭包，不在其中访问 UI。应用无需 `@preconcurrency import`，也不应给未同步的可变对象
添加 `unchecked Sendable` 来压制诊断。

共享会话由应用级所有者持有；页面离开仅解除自己的 UI 回调，不默认取消共享工作或关闭设备。
独占页面结束时由其所有者调用异步 close，避免把自己的清理责任转移给其他页面。
Activity 配置重建保留 owner，进程重建只恢复 App 草稿，不恢复连接、请求或自动补印；
后台断开／导入策略由 App 负责，回前台不隐式重连、初始化或打印。
