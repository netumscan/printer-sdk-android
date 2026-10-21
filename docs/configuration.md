# 连接与打印配置

[返回指南](README.md) · 适用版本 `0.3.0-preview.8`

Android 的连接绑定默认打印 Profile：SDK 按标准设备名匹配，或连接后空闲时由 App
调用 `configurePrintProfile` 补充／更正。连接前不传 Profile，单次打印、介质和定位不覆盖 Profile。
iOS 连接只拥有通信通道，协议 Profile 保持逐次打印、介质设置或走纸时单独传入。
Android SPP 使用经典蓝牙 RFCOMM；已知名称可匹配 BLE 参数，未知名称需显式服务、写特征和流控配置。
iOS 使用 BLE 外设 UUID 和显式通信配置。设备名、扫描名称或开发 preset 都不构成硬件验证。

## 当前移动端支持矩阵

| 能力 | Android | iOS |
| --- | --- | --- |
| 最低编译／部署目标 | API21、Java8 | iOS15、Swift5/6 |
| 经典蓝牙连接 | SPP/RFCOMM | 不提供 |
| 系统已配对蓝牙设备枚举 | `bondedDevices` | 同名方法回调 1004 |
| BLE 连接／扫描 | 名称匹配或显式通信配置、`scanBle` | 显式配置、`scanBLE` |
| 未配对经典设备主动搜索 | Manager 不提供，交由 App／系统配对 | 不提供 |
| TCP／USB／串口 | 无移动 Manager 入口 | 无移动 Manager 入口 |
| CPCL／ESC-POS 光栅及 Raw | 已实现，均进入同一设备 FIFO | 同 Android |
| 介质／下一标签定位 | CPCL tracking 配置与定位；其他字段按能力拒绝 | 同 Android |
| 结构化切纸／硬件状态查询 | 不支持；状态查询回调 1004，不发送猜测指令 | 同 Android |
| 固件、SN、电量读回 | 不提供；sdkInfo 只表示 SDK 身份 | 同 Android |
| 图像解码／来源 | SDK 可处理 PNG/JPEG、像素和自定义 loader；URI 导入归 App | App/plugin 负责 |
| 权限／配对界面 | App/plugin 负责 | App/plugin 负责 |
| 后台打印、自动重连／续打／重放 | 不提供 | 不提供 |

此矩阵描述软件实现，不表示最低系统运行、具体型号、实际 Flutter engine 或峰值内存已验收。
表中 1004 指参数和生命周期检查通过后的能力拒绝，不覆盖未连接或已关闭等错误。
Android `bondedDevices` 不按经典蓝牙／BLE 类型过滤；设备出现在列表中不表示其支持 SPP/RFCOMM。
未知硬件状态保持未知；验收范围随精确版本证据交付，`hardware_verified=false`。

## Android 设备配置与尺寸

以成功连接或配置回调的 `ConnectionInfo.printProfile` 为准，configurationSource 区分
NAME_RECOMMENDATION、EXPLICIT 与 NONE。无 Profile 时可通信和发送 Raw，不能结构化打印。
配置成功仅更新本地默认值，不发送打印机 IO，且使旧介质证据失效；App 重新调用 `setMedia`。
断开后配置清空。每次请求按接纳时的连接与 Profile 快照校验，不能将旧预览当作新设备配置。

PrinterProfile 描述 protocol、名义 dpi、maxPrintWidthDots、精确 dotsPerMm 和 labelAlignment。
已知型号无需外部再次传 DPI 与最大打印宽度；未知型号才由 App 补充已确认规格。
通用 `PrinterProfile.cpcl(dpi, maxWidthDots)` 使用 dpi/25.4 和 LEFT；具有精确规格的设备可用
带 dotsPerMm、labelAlignment 的工厂，不能用其他型号的纸卷宽度猜测点密度。

`PrintOptions.labelMm` 与 `labelDots` 指定实际标签宽高，`receiptDots` 指定小票宽度与整单长度预算；
图像输入为 px，画布、位置与走纸为 dots。mm 按已采用 Profile 计算 round(mm*dotsPerMm)，
包含 `ImageRender.containBoxMm` 与 loader 的目标尺寸，图像文件 DPI 不决定输出尺寸。
App 预览和毫米标尺使用同一 Profile 的 `dotsFromMm`；打印时 SDK 仍重新校验。
LT-M90 使用名义 203 DPI、864 dots、精确 8 dots/mm，50／99／108 mm 对应 400／792／864 dots。

LT-M90 的 CENTERED 规则在 Profile 内，SDK 按整字节补齐的画布宽度自动居中。
50 mm 标签默认起点 X232；App 只传标签宽高与可选 `horizontalCalibrationDots`，默认 0，正向右、负向左。
校准不改变 DPI、点密度或 Profile，也不由 SDK 自动持久化。负起点、右侧越界和溢出均拒绝，不裁切或钳制。
CENTERED 不允许非零绝对 x；通用 LEFT 的旧 `originDots` 非负绝对 x 保留，非零绝对 x 与非零校准互斥。
小票不支持标签校准。标签尺寸不要求 8 的倍数，右边界校验使用补齐后的编码宽度。

`PrintOptions` 固定标签／小票布局、ImageSource 顺序、空白、份数、超时和介质要求；
`ImageRender` 逐图指定阈值、Floyd-Steinberg／Bayer 抖动、fitWidth／containBox 及反色。
来源支持 GRAY8、RGB24、RGBA32 像素、Android resource、专用缓存文件或协作取消 loader；
RGBA 使用白底合成，fitWidth／containBox 只等比缩小，containBox 居中补白、不裁剪。
所有来源加载及校验完成后才能开始输出；CPCL 多图仅 x=0、纵向不重叠。

## iOS 逐次配置与共用约束

iOS Profile 描述协议、DPI、宽度、页高及相关策略，`PreparedPrint` 描述布局、像素与空白；
图像解码由 App 在 Manager 的 preparation 中完成，Swift 使用只读 Data。
同一像素对象可在多个元素中复用，SDK 共享一次 native 导入。
光栅条带与传输分块独立，应用不用保存完整的最终打印命令缓冲区。

`manager.capabilities` 异步返回只读能力值；分别检查 implementation、deviceSupport、evidence、
known 和 geometry 的有效字段。未知不等于支持、就绪或无限。标签页高是单页/单份限制，
小票 maxTotalLength 是所有副本和结束留白的整单预算，不能替换使用。

当前源载荷预算为单图 16MiB、每作业逻辑图片 32MiB、Raw 1MiB、活动与等待载荷 64MiB，
native 未终态 operation 最多 64 个，copies 为 1..100；Manager 另外限制 64 个未完成业务请求。
这些是接纳预算，不是手机 RSS 上限；资源不足应缩减输入或未完成工作，不通过轮询预留容量。
Android 图像准备还限制工作集 64MiB、最多 2 个实际 loader，以及整请求源像素与新增处理结果合计 32MiB。
取消终态不代表仍在读取的 loader 已退出，其资源与来源引用在实际退出后释放。

介质 DTO 可以表达完整字段，但不代表硬件支持。Profile 识别不推断 GAP／连续纸或实际纸张宽高。
当前明确 CPCL Profile 只实现已确认的 tracking
设置，其他字段或持久化要求按能力校验返回不支持，不静默忽略。
Android 介质证据绑定连接和相容 Profile，重连、配置变化或 Raw 保守失效；
请求可携带 MediaRequirement 并在执行前再次检查，ALLOW_LAST_SENT 仍只是曾发送过初始化，不是传感器读回。
当前未经验证的硬件查询通过 Manager 完成回调返回 `1004`，不发送猜测指令。
展示应用自己维护的连接状态与请求结果；不得把缓存连接成功解释为纸张正常、设备就绪或机械打印完成。
