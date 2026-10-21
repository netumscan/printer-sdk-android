# Android Java API 参考

发布候选版本 **0.3.0-preview.8**；原生 SDK **0.3.0**；C ABI **3/r2**。

本页逐项覆盖审定的编译器公共 API 基线，共 **336** 个文档条目。
声明、JVM descriptor、常量、枚举和编译器关系均来自基线，说明按稳定文档 ID 维护。
机器可读的完整基线与说明见 [api-reference.json](api-reference.json)；运行
`python3 tools/check-api-reference.py` 可检查覆盖范围、审定 SHA-256 和本页字节一致性。

基线 SHA-256：`4dcc42d4069e03b42c9922027e4d44811d07cf9362d1dc7a62c17ecc09350ded`

运行约束与错误处理见 [API 与生命周期](api-lifecycle.md) 和 [错误码与排障](errors-troubleshooting.md)。
本页是未发布的发布候选契约，不代表发布制品或公开历史证明。硬件验收仍暂缓，`hardware_verified=false`。

## 类型导航

- [AndroidImages](#api-6cc14f79c939373ebad3)
- [BleOptions](#api-168ab2e3c48dfc7424a0)
- [BleOptions.CreditFlowControl](#api-7b31b50111b4bf370803)
- [CancellationContext](#api-b6cc8aebaaca337eddbc)
- [ConnectionInfo](#api-bf113bf34fa7893fb410)
- [ConnectionInfo.ConfigurationSource](#api-618e1e74889703ec2387)
- [ConnectionOptions](#api-305c75156578a769bb1d)
- [CpclProfile](#api-d96de71f64e0601eb399)
- [DiscoveredDevice](#api-b790baf5c321454d18b7)
- [EscPosProfile](#api-961de0862e0cb78fdffe)
- [FeatureSupport](#api-1713e3bca4852cbf74f4)
- [ImageLoadContext](#api-33ef44e6a0d34ed58fc9)
- [ImageLoader](#api-d2a820217deb7df1eae0)
- [ImageRender](#api-c7ebed100a5f88acba1d)
- [ImageSource](#api-8283abbc7b9b40cd9caf)
- [LTM90Profile](#api-cc7a220484daaf56ad23)
- [ManagerResult](#api-98f169a3e4c7b889744f)
- [ManagerResult.Completion](#api-1721ae5f9f8202edb30d)
- [ManagerResult.Outcome](#api-2784cdf665d09aa3045c)
- [MediaMode](#api-4b74ae3f0e52d0db0f65)
- [MediaRequirement](#api-2f5be4c6d23e94746056)
- [MediaSettings](#api-e5824d9e8f16babb6dab)
- [PixelImage](#api-a4466c8912ce3d2b0c5b)
- [PreparedPrint](#api-5cf4c76272c2df7a9a90)
- [PreparedPrint.Builder](#api-9acaf968b2de3c0d2d3d)
- [PreparedPrint.Pixels](#api-52e18138e529bfa1a76c)
- [PrintOptions](#api-4dc5d455340db9937535)
- [PrintOptions.Builder](#api-0bdf506a98412f3151e5)
- [PrinterCapabilities](#api-a36171733f6c01585161)
- [PrinterManager](#api-bfc786be5b911ef74d19)
- [PrinterManager.Callback](#api-a8eff59edaee0576f508)
- [PrinterManager.DisconnectedListener](#api-9149b393dfb876ce6b8e)
- [PrinterManager.Preparation](#api-926b9d0552ac918f6e21)
- [PrinterProfile](#api-73cf28c112dad2127298)
- [PrinterProfile.LabelAlignment](#api-044125a48fcbc00942aa)
- [PrinterProfile.Protocol](#api-a6f2b17ef78a597d41ca)
- [PrinterProfiles](#api-c4cd867a0c3c5e8c017c)
- [RequestId](#api-764c5a4147cfdf641cc3)
- [ScalarConstraint](#api-b49ec93e3a988bef134a)
- [SdkInfo](#api-e68d6e49886d528a891c)
- [StatusQueryConfig](#api-31d51fc34c8d04a03b44)
- [StatusResult](#api-346a10d907279a636fde)

Java 签名保留编译器记录的二进制嵌套名 `$`；源码导入和类型访问使用 `.`。
方法参数名不在 `javap` 基线中，参数顺序与类型以声明和 descriptor 为准。

<a id="api-6cc14f79c939373ebad3"></a>

## AndroidImages

Android有界PNG/JPEG图片适配。

```java
public final class com.netumscan.printersdk.AndroidImages
```

工厂不在UI线程解码；SDK准备阶段读取、检查文件和EXIF，采用有界采样、白底合成及灰度输出。系统选图、URI授权、导入缓存复制和后台策略归App。

文档 ID：`android:type:com.netumscan.printersdk.AndroidImages`

<a id="api-992c07bfaa5684625114"></a>

### public static com.netumscan.printersdk.ImageSource fromResource(android.content.Context, int)

创建延迟解码的Android资源来源。

```java
public static com.netumscan.printersdk.ImageSource fromResource(android.content.Context, int);
```

只保存application Context；resourceId不能为0。准备时有界复制到缓存、检查格式、解码且禁用density缩放，临时文件在真实读取退出路径清理。 损坏／不支持格式或本地读取失败经原Callback报告1001、NONE且无输出；资源超限仍为1008，错误消息不包含文件路径。

文档 ID：`android:member:com.netumscan.printersdk.AndroidImages:cfea0f23da17d923`

JVM descriptor：`(Landroid/content/Context;I)Lcom/netumscan/printersdk/ImageSource;`

编译器注解：`@androidx.annotation.NonNull`。

参数 0（从 0 起）注解：`@androidx.annotation.NonNull`。

<a id="api-7ae91ed03b30df9d22c7"></a>

### public static com.netumscan.printersdk.ImageSource takeFile(java.io.File)

接管专用稳定导入缓存副本。

```java
public static com.netumscan.printersdk.ImageSource takeFile(java.io.File);
```

成功后App不得修改或删除该文件。SDK在后台检查32MiB文件、6400万源像素、32768单边及32MiB位图等边界并解码PNG/JPEG、EXIF1..8；最后读取退出后后台删除。工厂不保证文件已有效，失败经打印Callback交付。 损坏／不支持格式或本地读取失败经原Callback报告1001、NONE且无输出；资源超限仍为1008，错误消息不包含文件路径。

文档 ID：`android:member:com.netumscan.printersdk.AndroidImages:9007555b1e89fae5`

JVM descriptor：`(Ljava/io/File;)Lcom/netumscan/printersdk/ImageSource;`

编译器注解：`@androidx.annotation.NonNull`。

参数 0（从 0 起）注解：`@androidx.annotation.NonNull`。

<a id="api-168ab2e3c48dfc7424a0"></a>

## BleOptions

不可变的 BLE 服务、写入及可选信用流控配置。

```java
public final class com.netumscan.printersdk.BleOptions
```

应用须显式提供服务与特征 UUID、MTU 请求值及写入上限；没有自动识别型号或猜测 UUID 的行为。构造只校验配置，不连接设备。`withResponse` 选择 GATT 写入方式，不能作为打印机执行确认；实际单包长度受协商 MTU、配置上限和启用流控时固件通知的 `packetLimit` 限制。Credit 单独控制剩余发送次数，每发送一包消耗 1 个，不表示可发送字节数。字段均为 `final`，可在多个设备创建请求间复用。

文档 ID：`android:type:com.netumscan.printersdk.BleOptions`

<a id="api-06c89b8949bb2d2f4f47"></a>

### public com.netumscan.printersdk.BleOptions(java.util.UUID, java.util.UUID, boolean, int, int)

创建不启用信用流控的 BLE 配置。

```java
public com.netumscan.printersdk.BleOptions(java.util.UUID, java.util.UUID, boolean, int, int);
```

参数 `service`、`writeCharacteristic` 为非 null 的服务和写特征 UUID；`withResponse` 为 true 时使用带响应写入，false 时使用无响应写入；`requestedMtu` 的范围为 23..517 字节，`maxWriteBytes` 的范围为 1..512 字节。等价于六参数构造器传入 `creditFlowControl=null`。配置非法抛出 `IllegalArgumentException`。同步返回新配置，不执行蓝牙 IO；没有隐式 MTU 或分包默认值。

文档 ID：`android:member:com.netumscan.printersdk.BleOptions:c88f3fcaa589a6a9`

JVM descriptor：`(Ljava/util/UUID;Ljava/util/UUID;ZII)V`

<a id="api-62616d2dd2d65d913d01"></a>

### public com.netumscan.printersdk.BleOptions(java.util.UUID, java.util.UUID, boolean, int, int, com.netumscan.printersdk.BleOptions$CreditFlowControl)

创建带可选信用流控的 BLE 配置。

```java
public com.netumscan.printersdk.BleOptions(java.util.UUID, java.util.UUID, boolean, int, int, com.netumscan.printersdk.BleOptions$CreditFlowControl);
```

参数 `service`、`writeCharacteristic` 不可为 null；`requestedMtu` 为 23..517 字节，`maxWriteBytes` 为 1..512 字节。`creditFlowControl` 可为 null；非 null 时 `withResponse` 必须为 false，且写特征必须不同于 `dataOut` 和 `dataFC`。该对象已保证两种通知特征互不相同。违反条件抛出 `IllegalArgumentException`。同步创建配置，不发起连接或订阅；GATT 属性和通知能力在 `connect()` 时验证。

文档 ID：`android:member:com.netumscan.printersdk.BleOptions:7cd4a0c376a401a5`

JVM descriptor：`(Ljava/util/UUID;Ljava/util/UUID;ZIILcom/netumscan/printersdk/BleOptions$CreditFlowControl;)V`

<a id="api-cb046178c975767ab6e0"></a>

### public final boolean withResponse

是否使用带响应的 GATT 特征写入。

```java
public final boolean withResponse;
```

由构造参数显式指定，true 对应 `WRITE_TYPE_DEFAULT`，false 对应 `WRITE_TYPE_NO_RESPONSE`；启用信用流控时必须为 false。GATT 写回调或响应不表示纸面打印完成。没有可变 setter。

文档 ID：`android:member:com.netumscan.printersdk.BleOptions:9349d0eb41799510`

JVM descriptor：`Z`

<a id="api-7dd3ed5c4225dba40584"></a>

### public final com.netumscan.printersdk.BleOptions$CreditFlowControl creditFlowControl

可选的 Data Transmission V1.2 信用流控配置。

```java
public final com.netumscan.printersdk.BleOptions$CreditFlowControl creditFlowControl;
```

值为 null 表示不启用该流控；五参数构造器固定采用 null。非 null 时连接会订阅两种通知特征，并等待初始信用信息后才就绪；不能用它代替通用状态查询或打印完成确认。

文档 ID：`android:member:com.netumscan.printersdk.BleOptions:5020202ee255a725`

JVM descriptor：`Lcom/netumscan/printersdk/BleOptions$CreditFlowControl;`

<a id="api-3e70e0378b25fbbd5251"></a>

### public final int maxWriteBytes

单次 BLE 写入的数据长度上限，单位为字节。

```java
public final int maxWriteBytes;
```

合法范围为 1..512，由构造参数显式指定。实际单包长度不超过此值与协商 MTU 减 3 的较小值；启用信用流控时，还受固件通知的独立单包字节上限 `packetLimit` 限制。Credit 表示剩余发送次数，每发送一包消耗 1 个；额度为零时等待新的额度通知，不用额度数值限制包长。这是传输分包上限，不是光栅条带高度或完整作业大小限制。

文档 ID：`android:member:com.netumscan.printersdk.BleOptions:f96a0e16f66d9c02`

JVM descriptor：`I`

<a id="api-329424eb072add7be305"></a>

### public final int requestedMtu

连接时请求的 ATT MTU，单位为字节。

```java
public final int requestedMtu;
```

合法范围为 23..517。23 表示沿用初始 MTU；大于 23 时请求 MTU 协商。请求值不保证等于协商结果，实际写包还受 `maxWriteBytes` 限制；没有自动将 MTU 请求值当成可用载荷大小的行为。

文档 ID：`android:member:com.netumscan.printersdk.BleOptions:11c9b2aff8de43b4`

JVM descriptor：`I`

<a id="api-3a483f5c69ca680c6989"></a>

### public final java.util.UUID service

目标 BLE GATT 服务 UUID。

```java
public final java.util.UUID service;
```

构造时必须非 null。连接后在该服务中查找写特征和可选通知特征；未知服务不会自动回退到其他服务。UUID 本身不证明设备型号或协议兼容性。

文档 ID：`android:member:com.netumscan.printersdk.BleOptions:d1aa937cc2ea3c07`

JVM descriptor：`Ljava/util/UUID;`

<a id="api-4790d77a0327dac499e3"></a>

### public final java.util.UUID writeCharacteristic

目标服务内用于发送打印数据的特征 UUID。

```java
public final java.util.UUID writeCharacteristic;
```

构造时必须非 null；连接时验证其具备所选的带响应或无响应写属性。启用信用流控时不得与两种通知特征 UUID 相同。特征不支持时连接失败，通常报告 `PrinterException` code 1004。

文档 ID：`android:member:com.netumscan.printersdk.BleOptions:0d32a05b9d9ca5ae`

JVM descriptor：`Ljava/util/UUID;`

<a id="api-7b31b50111b4bf370803"></a>

## BleOptions.CreditFlowControl

Data Transmission V1.2 的两种通知特征配置。

```java
public final class com.netumscan.printersdk.BleOptions$CreditFlowControl
```

不可变的 `BleOptions` 嵌套值类型。`dataOut` 与 `dataFC` 均位于配置选定的服务；SDK 使用通知建立流控会话，其中信用通知决定可发送额度。只适用于明确支持该固件协议的设备，配置本身不升级硬件证据等级，也不公开平台 GATT 句柄。

文档 ID：`android:type:com.netumscan.printersdk.BleOptions$CreditFlowControl`

<a id="api-6657cb40e72e0c97a9e3"></a>

### public com.netumscan.printersdk.BleOptions$CreditFlowControl(java.util.UUID, java.util.UUID)

声明互不相同的数据通知与信用通知 UUID。

```java
public com.netumscan.printersdk.BleOptions$CreditFlowControl(java.util.UUID, java.util.UUID);
```

参数 `dataOut`、`dataFC` 均不可为 null，且两者不得相等，否则同步抛出 `IllegalArgumentException`。返回新配置，不进行订阅。作为 `BleOptions.creditFlowControl` 使用时，还要求写特征不同于这两个 UUID，并使用无响应写入。

文档 ID：`android:member:com.netumscan.printersdk.BleOptions$CreditFlowControl:33abf9a6aa732f1e`

JVM descriptor：`(Ljava/util/UUID;Ljava/util/UUID;)V`

<a id="api-ab18289d478138156f5d"></a>

### public final java.util.UUID dataFC

Data Transmission V1.2 的信用流控通知特征 UUID。

```java
public final java.util.UUID dataFC;
```

非 null，且不同于 `dataOut`。连接时订阅此特征，解析其信用信息并限制后续发送；等待信用也占用操作超时预算。通知不表示纸面打印完成；流控错误不会自动重放已发送数据。

文档 ID：`android:member:com.netumscan.printersdk.BleOptions$CreditFlowControl:1c434c73eb8d8d08`

JVM descriptor：`Ljava/util/UUID;`

<a id="api-ba50413f8eac6399d66f"></a>

### public final java.util.UUID dataOut

Data Transmission V1.2 的数据通知特征 UUID。

```java
public final java.util.UUID dataOut;
```

非 null，且不同于 `dataFC`。连接时须能在目标服务中找到该特征、NOTIFY 属性与 CCCD；SDK 订阅该特征以建立此固件要求的通知通道。当前公共 Java API 不提供数据通知回调，不能据此读取通用打印机状态。

文档 ID：`android:member:com.netumscan.printersdk.BleOptions$CreditFlowControl:e8f4989c9668b7e5`

JVM descriptor：`Ljava/util/UUID;`

<a id="api-b6cc8aebaaca337eddbc"></a>

## CancellationContext

准备阶段的协作取消上下文。

```java
public final class com.netumscan.printersdk.CancellationContext
```

应用在耗时像素准备中检查取消并尽快返回；该对象不持有可操作的native资源。请求取消或清理后，迟到的准备结果不能重新提交打印。

文档 ID：`android:type:com.netumscan.printersdk.CancellationContext`

<a id="api-2e00f14c155bf147c36d"></a>

### public boolean isCancelled()

检查准备是否已被取消。

```java
public boolean isCancelled();
```

应用在耗时准备的分块边界检查该值并尽早结束；仅用于协作取消，不提供native设备操作。

文档 ID：`android:member:com.netumscan.printersdk.CancellationContext:f1f8e4ff91b49c6d`

JVM descriptor：`()Z`

<a id="api-e1c2075542c094fbbc54"></a>

### public long remainingTimeMs()

取得本次请求的剩余时间。

```java
public long remainingTimeMs();
```

毫秒预算包含Manager排队与准备，归零表示预算已耗尽；不能用新的完整超时重置原请求截止时间。

文档 ID：`android:member:com.netumscan.printersdk.CancellationContext:e0a63064191a9f55`

JVM descriptor：`()J`

<a id="api-0f816b14b5645dfb7250"></a>

### public void throwIfCancelled()

在准备函数中检查取消或总超时。

```java
public void throwIfCancelled();
```

取消时抛出CancellationException，超时通过稳定Timeout错误结束；应用应在耗时准备的边界调用该方法。

文档 ID：`android:member:com.netumscan.printersdk.CancellationContext:871ea18d6bc6f0cb`

JVM descriptor：`()V`

<a id="api-bf113bf34fa7893fb410"></a>

## ConnectionInfo

本次连接采用的本地配置。

```java
public final class com.netumscan.printersdk.ConnectionInfo
```

只返回地址、可空原始名称、可空打印Profile与配置来源，不包含连接控制或原生句柄。名称推荐不构成型号认证，未知硬件信息不补造。

文档 ID：`android:type:com.netumscan.printersdk.ConnectionInfo`

<a id="api-d9e9a591203ff9a82278"></a>

### public final com.netumscan.printersdk.ConnectionInfo$ConfigurationSource configurationSource

当前连接打印配置的采用来源。

```java
public final com.netumscan.printersdk.ConnectionInfo$ConfigurationSource configurationSource;
```

NAME_RECOMMENDATION来自连接时自动匹配；EXPLICIT仅来自连接成功后的configurePrintProfile；没有Profile时为NONE。名称匹配和手动配置均不提升硬件证据。

文档 ID：`android:member:com.netumscan.printersdk.ConnectionInfo:e6bc5db30f85806e`

JVM descriptor：`Lcom/netumscan/printersdk/ConnectionInfo$ConfigurationSource;`

编译器注解：`@androidx.annotation.NonNull`。

<a id="api-aecead461c9a9e157c0a"></a>

### public final com.netumscan.printersdk.PrinterProfile printProfile

可空的连接默认打印配置。

```java
public final com.netumscan.printersdk.PrinterProfile printProfile;
```

未知为null；允许通信与Raw，但普通结构化打印需要明确配置。每请求提交捕获快照，不保存全局最近Profile。

文档 ID：`android:member:com.netumscan.printersdk.ConnectionInfo:4d4c3e3356b1b1c7`

JVM descriptor：`Lcom/netumscan/printersdk/PrinterProfile;`

编译器注解：`@androidx.annotation.Nullable`。

<a id="api-4c028dbba5fe7fd68071"></a>

### public final java.lang.String address

目标设备地址。

```java
public final java.lang.String address;
```

来自连接选项，不作为硬件型号、在线状态或认证证据。连接成功结果保留本次采用的目标值。

文档 ID：`android:member:com.netumscan.printersdk.ConnectionInfo:b1aeb5a5c56be66f`

JVM descriptor：`Ljava/lang/String;`

编译器注解：`@androidx.annotation.NonNull`。

<a id="api-32b6ddd89b5a8b314325"></a>

### public final java.lang.String deviceName

可空的原始设备名称。

```java
public final java.lang.String deviceName;
```

只用于展示与集中推荐解析；未知为null，不使用N/A，也不接收带地址或UUID的UI标题。

文档 ID：`android:member:com.netumscan.printersdk.ConnectionInfo:f4705af91bfff61e`

JVM descriptor：`Ljava/lang/String;`

编译器注解：`@androidx.annotation.Nullable`。

<a id="api-618e1e74889703ec2387"></a>

## ConnectionInfo.ConfigurationSource

打印配置的采用来源。

```java
public final class com.netumscan.printersdk.ConnectionInfo$ConfigurationSource extends java.lang.Enum<com.netumscan.printersdk.ConnectionInfo$ConfigurationSource>
```

EXPLICIT表示连接后手动配置，NAME_RECOMMENDATION表示连接时标准名称匹配，NONE表示没有打印配置；均不代表硬件能力已回读。

文档 ID：`android:type:com.netumscan.printersdk.ConnectionInfo$ConfigurationSource`

枚举声明顺序（`ordinal()` 从 0 起；不作为持久化协议值）：

| ordinal | 枚举项 |
| --- | --- |
| 0 | `EXPLICIT` |
| 1 | `NAME_RECOMMENDATION` |
| 2 | `NONE` |

<a id="api-78319ffce1ff9116bc89"></a>

### public static com.netumscan.printersdk.ConnectionInfo$ConfigurationSource valueOf(java.lang.String)

按精确枚举名查找。

```java
public static com.netumscan.printersdk.ConnectionInfo$ConfigurationSource valueOf(java.lang.String);
```

Java编译器生成的方法，名称必须精确匹配；null或未知名称使用Java标准异常，不执行设备IO。

文档 ID：`android:member:com.netumscan.printersdk.ConnectionInfo$ConfigurationSource:61f127470d12356d`

JVM descriptor：`(Ljava/lang/String;)Lcom/netumscan/printersdk/ConnectionInfo$ConfigurationSource;`

这是 Java 编译器为枚举生成的公共方法。

<a id="api-513820916fd742c3f11e"></a>

### public static com.netumscan.printersdk.ConnectionInfo$ConfigurationSource[] values()

取得按声明顺序排列的枚举值。

```java
public static com.netumscan.printersdk.ConnectionInfo$ConfigurationSource[] values();
```

Java编译器生成的新数组，修改数组不修改枚举定义或SDK配置。

文档 ID：`android:member:com.netumscan.printersdk.ConnectionInfo$ConfigurationSource:023e9d3b5cc22838`

JVM descriptor：`()[Lcom/netumscan/printersdk/ConnectionInfo$ConfigurationSource;`

这是 Java 编译器为枚举生成的公共方法。

<a id="api-8c3680bd767a4ca270f2"></a>

### public static final com.netumscan.printersdk.ConnectionInfo$ConfigurationSource EXPLICIT

连接后显式提供配置。

```java
public static final com.netumscan.printersdk.ConnectionInfo$ConfigurationSource EXPLICIT;
```

仅由configurePrintProfile成功采用，后续请求使用该连接配置；不是硬件回读或型号认证。

文档 ID：`android:member:com.netumscan.printersdk.ConnectionInfo$ConfigurationSource:b61d66f3034533f8`

JVM descriptor：`Lcom/netumscan/printersdk/ConnectionInfo$ConfigurationSource;`

<a id="api-abfd32f5357a91d3e7e2"></a>

### public static final com.netumscan.printersdk.ConnectionInfo$ConfigurationSource NAME_RECOMMENDATION

采用原始名称的推荐配置。

```java
public static final com.netumscan.printersdk.ConnectionInfo$ConfigurationSource NAME_RECOMMENDATION;
```

名称可修改，不把推荐升级为确认能力。

文档 ID：`android:member:com.netumscan.printersdk.ConnectionInfo$ConfigurationSource:da136cf06148ff4f`

JVM descriptor：`Lcom/netumscan/printersdk/ConnectionInfo$ConfigurationSource;`

<a id="api-fa58c0b36ae902363695"></a>

### public static final com.netumscan.printersdk.ConnectionInfo$ConfigurationSource NONE

没有有效默认打印配置。

```java
public static final com.netumscan.printersdk.ConnectionInfo$ConfigurationSource NONE;
```

仍可通信和Raw，普通结构化打印需先明确配置。

文档 ID：`android:member:com.netumscan.printersdk.ConnectionInfo$ConfigurationSource:42d1d1b1218c4065`

JVM descriptor：`Lcom/netumscan/printersdk/ConnectionInfo$ConfigurationSource;`

<a id="api-305c75156578a769bb1d"></a>

## ConnectionOptions

不可变的通信目标与传输参数。

```java
public final class com.netumscan.printersdk.ConnectionOptions
```

只保留地址、可空原始设备名称与BLE参数，不接受打印Profile。Manager在连接时按原始名称自动匹配；未知打印配置需连接后configurePrintProfile。未知BLE配置允许纯值创建，但connect异步拒绝1001，不降级SPP。工厂不连接或解码。

文档 ID：`android:type:com.netumscan.printersdk.ConnectionOptions`

<a id="api-85d0172e48787683a475"></a>

### public com.netumscan.printersdk.ConnectionOptions withBleOptions(com.netumscan.printersdk.BleOptions)

返回采用显式BLE参数的新选项。

```java
public com.netumscan.printersdk.ConnectionOptions withBleOptions(com.netumscan.printersdk.BleOptions);
```

参数非空，保留地址和原始名称，并明确选择BLE路径。旧值不修改，不发IO；通信参数改变需重新连接。打印Profile由Manager在连接时匹配，不能在这里指定。

文档 ID：`android:member:com.netumscan.printersdk.ConnectionOptions:719ff3dbf677fdda`

JVM descriptor：`(Lcom/netumscan/printersdk/BleOptions;)Lcom/netumscan/printersdk/ConnectionOptions;`

编译器注解：`@androidx.annotation.NonNull`。

参数 0（从 0 起）注解：`@androidx.annotation.NonNull`。

<a id="api-3e5b97334c8317067cf0"></a>

### public final com.netumscan.printersdk.BleOptions ble

可空的BLE通信配置。

```java
public final com.netumscan.printersdk.BleOptions ble;
```

SPP为空；未知名称的BLE纯值也可能暂为空，此时须withBleOptions后再连接。不能仅根据本字段是否为空推断传输路径。

文档 ID：`android:member:com.netumscan.printersdk.ConnectionOptions:aba718e6092d58ce`

JVM descriptor：`Lcom/netumscan/printersdk/BleOptions;`

编译器注解：`@androidx.annotation.Nullable`。

<a id="api-c19d55fa15e17d1a5d0c"></a>

### public final java.lang.String address

蓝牙设备地址。

```java
public final java.lang.String address;
```

连接时需要明确有效地址；发现值不包含可直接操作的平台Bluetooth句柄。

文档 ID：`android:member:com.netumscan.printersdk.ConnectionOptions:b1aeb5a5c56be66f`

JVM descriptor：`Ljava/lang/String;`

编译器注解：`@androidx.annotation.NonNull`。

<a id="api-9ce6f96faa7d274ed135"></a>

### public final java.lang.String deviceName

可空的原始设备名称。

```java
public final java.lang.String deviceName;
```

只用于展示与集中推荐解析；未知为null，不使用N/A，也不接收带地址或UUID的UI标题。

文档 ID：`android:member:com.netumscan.printersdk.ConnectionOptions:f4705af91bfff61e`

JVM descriptor：`Ljava/lang/String;`

编译器注解：`@androidx.annotation.Nullable`。

<a id="api-e1aefbaaa015449dd92b"></a>

### public static com.netumscan.printersdk.ConnectionOptions ble(com.netumscan.printersdk.DiscoveredDevice)

从发现设备创建BLE连接选项。

```java
public static com.netumscan.printersdk.ConnectionOptions ble(com.netumscan.printersdk.DiscoveredDevice);
```

保留地址和原始名称，按标准名称推荐BLE通信参数。未知名称不猜测BLE参数，connect回调1001；需要时使用带BleOptions工厂。打印Profile由Manager连接时独立匹配，本工厂不采用打印配置或执行IO。

文档 ID：`android:member:com.netumscan.printersdk.ConnectionOptions:14e6959e5230145e`

JVM descriptor：`(Lcom/netumscan/printersdk/DiscoveredDevice;)Lcom/netumscan/printersdk/ConnectionOptions;`

编译器注解：`@androidx.annotation.NonNull`。

参数 0（从 0 起）注解：`@androidx.annotation.NonNull`。

<a id="api-bc83dc6de2caa4b82ca6"></a>

### public static com.netumscan.printersdk.ConnectionOptions ble(com.netumscan.printersdk.DiscoveredDevice, com.netumscan.printersdk.BleOptions)

从发现设备和显式BLE参数创建连接选项。

```java
public static com.netumscan.printersdk.ConnectionOptions ble(com.netumscan.printersdk.DiscoveredDevice, com.netumscan.printersdk.BleOptions);
```

保留地址、原始名称与非空BleOptions，不按名称覆盖显式通信参数。Manager在连接时独立匹配打印Profile；未知打印配置需连接后手动补充。本工厂不执行IO。

文档 ID：`android:member:com.netumscan.printersdk.ConnectionOptions:8096b8a33a8e961e`

JVM descriptor：`(Lcom/netumscan/printersdk/DiscoveredDevice;Lcom/netumscan/printersdk/BleOptions;)Lcom/netumscan/printersdk/ConnectionOptions;`

编译器注解：`@androidx.annotation.NonNull`。

参数 0（从 0 起）注解：`@androidx.annotation.NonNull`。

参数 1（从 0 起）注解：`@androidx.annotation.NonNull`。

<a id="api-d856f066b9e2be032eb1"></a>

### public static com.netumscan.printersdk.ConnectionOptions ble(java.lang.String, com.netumscan.printersdk.BleOptions)

构造BLE连接目标。

```java
public static com.netumscan.printersdk.ConnectionOptions ble(java.lang.String, com.netumscan.printersdk.BleOptions);
```

传入明确地址及BleOptions服务/特征配置；DTO不打开连接、不执行扫描，也不暴露平台设备句柄。

文档 ID：`android:member:com.netumscan.printersdk.ConnectionOptions:040f19a651bedc97`

JVM descriptor：`(Ljava/lang/String;Lcom/netumscan/printersdk/BleOptions;)Lcom/netumscan/printersdk/ConnectionOptions;`

编译器注解：`@androidx.annotation.NonNull`。

参数 0（从 0 起）注解：`@androidx.annotation.NonNull`。

参数 1（从 0 起）注解：`@androidx.annotation.NonNull`。

<a id="api-82b72027b7d1828bd9eb"></a>

### public static com.netumscan.printersdk.ConnectionOptions spp(com.netumscan.printersdk.DiscoveredDevice)

从发现设备创建SPP连接选项。

```java
public static com.netumscan.printersdk.ConnectionOptions spp(com.netumscan.printersdk.DiscoveredDevice);
```

保留地址和可空原始名称，Manager在连接时按完整名称自动匹配打印Profile。没有连接前Profile指定入口。系统配对和权限由App负责，工厂不连接。

文档 ID：`android:member:com.netumscan.printersdk.ConnectionOptions:64be4e66f59c4f6b`

JVM descriptor：`(Lcom/netumscan/printersdk/DiscoveredDevice;)Lcom/netumscan/printersdk/ConnectionOptions;`

编译器注解：`@androidx.annotation.NonNull`。

参数 0（从 0 起）注解：`@androidx.annotation.NonNull`。

<a id="api-c2f74fa8ad3dc7549046"></a>

### public static com.netumscan.printersdk.ConnectionOptions spp(java.lang.String)

构造经典蓝牙SPP连接目标。

```java
public static com.netumscan.printersdk.ConnectionOptions spp(java.lang.String);
```

传入明确设备地址；Manager.connect负责异步参数校验和连接，协议profile仍由每个打印请求独立选择。

文档 ID：`android:member:com.netumscan.printersdk.ConnectionOptions:c8b71af4f566db64`

JVM descriptor：`(Ljava/lang/String;)Lcom/netumscan/printersdk/ConnectionOptions;`

编译器注解：`@androidx.annotation.NonNull`。

参数 0（从 0 起）注解：`@androidx.annotation.NonNull`。

<a id="api-d96de71f64e0601eb399"></a>

## CpclProfile

声明 CPCL 打印的显式光栅几何及兼容便捷入口默认值。

```java
public final class com.netumscan.printersdk.CpclProfile
```

不可变配置，可在每次 CPCL 操作提交时提供；旧 CPCL 设备工厂也可绑定它。点数为打印机 dots，DPI 与纸张毫米数需由调用方结合实际设备换算，SDK 不猜测型号。`widthDots` 是设备最大可打印宽度，不是输入图片固定宽度。`xOffsetDots` 和 `form` 仅为 `submitGray8()` 便捷入口的默认值；显式 `submitLabelGray8()` 使用自己的原点和结束策略，Raw 字节不受这些值改写。配置不是硬件验收证明。

文档 ID：`android:type:com.netumscan.printersdk.CpclProfile`

<a id="api-d9243ca54e436527f6fc"></a>

### public com.netumscan.printersdk.CpclProfile(int, int, int, boolean)

创建水平原点为零的 CPCL 配置。

```java
public com.netumscan.printersdk.CpclProfile(int, int, int, boolean);
```

参数 `dpi` 为正整数；`widthDots` 为 8..7992 dots，必须为 8 的倍数；`maxLabelHeightDots` 为 1..65535 dots；`form` 指定兼容 `submitGray8()` 的下一标签开始结束策略。等价于五参数构造器传入 `xOffsetDots=0`。非法几何同步抛出 `IllegalArgumentException`；不执行 IO，不自动推断纸宽。

文档 ID：`android:member:com.netumscan.printersdk.CpclProfile:7002d48a5c80a84b`

JVM descriptor：`(IIIZ)V`

<a id="api-acf193df3ed9c8f9d898"></a>

### public com.netumscan.printersdk.CpclProfile(int, int, int, boolean, int)

创建带水平光栅原点的 CPCL 配置。

```java
public com.netumscan.printersdk.CpclProfile(int, int, int, boolean, int);
```

参数 `dpi` 为正整数；`widthDots` 为 8..7992 dots 且为 8 的倍数；`maxLabelHeightDots` 为 1..65535 dots；`xOffsetDots` 为 0..`widthDots-1`；`form` 指定兼容图像提交的结束策略。非法参数同步抛出 `IllegalArgumentException`。偏移不会扩大可打印宽度或裁剪图片：提交时偏移加按整字节补齐的图宽仍须落在设备宽度内。显式标签提交可另设原点/结束策略；Raw 命令不受此配置改写。

文档 ID：`android:member:com.netumscan.printersdk.CpclProfile:8f3cfa075f5b2a1c`

JVM descriptor：`(IIIZI)V`

<a id="api-b21b6b4649d00efca4d5"></a>

### public final boolean form

兼容图像入口默认采用的标签结束策略。

```java
public final boolean form;
```

true 使 `submitGray8()` 使用下一标签开始策略（CPCL FORM）；false 使用内容结束策略。它不选择 GAP/BLACK_MARK/CONTINUOUS、不校准传感器，也不验证纸张实际位置。`submitLabelGray8()` 的 `form` 参数独立覆盖该默认值，Raw 不受影响。

文档 ID：`android:member:com.netumscan.printersdk.CpclProfile:6df5f7cd5045db8d`

JVM descriptor：`Z`

<a id="api-dc0ed4d1751e21a8bf5b"></a>

### public final int dpi

显式声明的打印密度，单位为 dots per inch。

```java
public final int dpi;
```

由构造器要求为正整数，没有型号默认值。Java 包装将其用于标签作业的 X/Y DPI 和几何校验；它不自动改变打印机物理分辨率，必须与实际设备匹配。

文档 ID：`android:member:com.netumscan.printersdk.CpclProfile:a54ea9eab4fa22b3`

JVM descriptor：`I`

<a id="api-62ac8ccfc717fd318b32"></a>

### public final int maxLabelHeightDots

一张标签的最大页高，单位为 dots。

```java
public final int maxLabelHeightDots;
```

范围为 1..65535，由调用方按设备约束声明。显式标签提交的 `originY + height + trailingBlankDots` 须不超过它；此值不乘 copies，也不是整单累计长度或手机内存承诺。

文档 ID：`android:member:com.netumscan.printersdk.CpclProfile:442cf4821daf3658`

JVM descriptor：`I`

<a id="api-9fc785d8c552c1b98cd0"></a>

### public final int widthDots

设备可打印的最大光栅宽度，单位为 dots。

```java
public final int widthDots;
```

范围为 8..7992，必须为 8 的倍数。输入图片可窄于该值；SDK 不自动缩放，实际图宽按 8 点补齐后再加水平原点不得超过此上限。纸张毫米宽度或蓝牙包长均不能直接代入。

文档 ID：`android:member:com.netumscan.printersdk.CpclProfile:d1f158b9ba2a99a4`

JVM descriptor：`I`

<a id="api-5180a79db48655575a6e"></a>

### public final int xOffsetDots

兼容图像入口默认的非负水平原点，单位为 dots。

```java
public final int xOffsetDots;
```

四参数构造器默认值为 0；显式值必须小于 `widthDots`。仅 `submitGray8()` 继承此值，`submitLabelGray8()` 使用自己的 `originX`。原点不改变输入图尺寸或 Raw 字节，超界不自动裁剪。

文档 ID：`android:member:com.netumscan.printersdk.CpclProfile:af181a7757d57091`

JVM descriptor：`I`

<a id="api-b790baf5c321454d18b7"></a>

## DiscoveredDevice

蓝牙发现或配对枚举的只读设备信息。

```java
public final class com.netumscan.printersdk.DiscoveredDevice
```

包含地址、名称、可选RSSI与广播服务标识；后续连接使用设备标识和显式BLE选项，不暴露平台设备句柄。

文档 ID：`android:type:com.netumscan.printersdk.DiscoveredDevice`

<a id="api-5d1582cb157eea406b6d"></a>

### public final java.lang.Integer rssi

扫描接收信号强度。

```java
public final java.lang.Integer rssi;
```

单位为dBm；没有扫描测量值时为空，不使用特殊数值伪造有效测量。

文档 ID：`android:member:com.netumscan.printersdk.DiscoveredDevice:4911150c511b5cd7`

JVM descriptor：`Ljava/lang/Integer;`

<a id="api-862732bae26f04800853"></a>

### public final java.lang.String address

蓝牙设备地址。

```java
public final java.lang.String address;
```

连接时需要明确有效地址；发现值不包含可直接操作的平台Bluetooth句柄。

文档 ID：`android:member:com.netumscan.printersdk.DiscoveredDevice:b1aeb5a5c56be66f`

JVM descriptor：`Ljava/lang/String;`

<a id="api-9b7190a400caf05e5d3d"></a>

### public final java.lang.String name

设备广播或系统提供的名称。

```java
public final java.lang.String name;
```

未知时使用可空值；该名称用于显示，不能替代明确的连接标识。

文档 ID：`android:member:com.netumscan.printersdk.DiscoveredDevice:77409202b35ac1a5`

JVM descriptor：`Ljava/lang/String;`

<a id="api-969a356a5a3af09831f5"></a>

### public final java.util.List<java.lang.String> advertisedServices

广播的服务标识列表。

```java
public final java.util.List<java.lang.String> advertisedServices;
```

只读数据；不提供BluetoothGatt或平台扫描对象，连接仍需要明确配置。

文档 ID：`android:member:com.netumscan.printersdk.DiscoveredDevice:278808084302b914`

JVM descriptor：`Ljava/util/List;`

<a id="api-961de0862e0cb78fdffe"></a>

## EscPosProfile

声明 ESC/POS 小票光栅的显式 DPI 和可打印宽度。

```java
public final class com.netumscan.printersdk.EscPosProfile
```

不可变配置，用于 ESC/POS 设备工厂。必须与实际设备的密度和可打印点宽匹配，不从纸宽或名称推断型号。该类型不包含切刀或状态支持声明；当前标准后端不实现状态查询及切纸。整张小票须由调用方渲染为 Gray8 图片，再用 `submitReceiptGray8()` 提交。

文档 ID：`android:type:com.netumscan.printersdk.EscPosProfile`

<a id="api-a389a27bb00caa90a626"></a>

### public com.netumscan.printersdk.EscPosProfile(int, int)

创建 ESC/POS 光栅几何配置。

```java
public com.netumscan.printersdk.EscPosProfile(int, int);
```

参数 `dpi` 为 1..255 dots per inch，`widthDots` 为 8..7992 dots 且为 8 的倍数。超出范围同步抛出 `IllegalArgumentException`。两值都必须显式提供，没有型号默认值；构造不连接或设置硬件。

文档 ID：`android:member:com.netumscan.printersdk.EscPosProfile:13dfdc2812e6b642`

JVM descriptor：`(II)V`

<a id="api-b663d0e9cf4b797b927b"></a>

### public final int dpi

ESC/POS 作业声明的打印密度，单位为 dots per inch。

```java
public final int dpi;
```

范围为 1..255，由构造参数指定；用于与设备配置一致的 X/Y DPI。它不测量打印头、不自动改变硬件分辨率，配置值须经调用方核实。

文档 ID：`android:member:com.netumscan.printersdk.EscPosProfile:a54ea9eab4fa22b3`

JVM descriptor：`I`

<a id="api-73fb6b16df56ec320fd7"></a>

### public final int widthDots

ESC/POS 设备可打印的最大光栅宽度，单位为 dots。

```java
public final int widthDots;
```

范围为 8..7992，必须为 8 的倍数。整张小票图片可窄于该值；按 8 点补齐后的图宽必须在此范围内，不自动缩放或裁剪，也不等同于纸张毫米宽度。

文档 ID：`android:member:com.netumscan.printersdk.EscPosProfile:d1f158b9ba2a99a4`

JVM descriptor：`I`

<a id="api-1713e3bca4852cbf74f4"></a>

## FeatureSupport

分别记录实现可用性、设备支持与证据的不可变快照。

```java
public final class com.netumscan.printersdk.FeatureSupport
```

通过 `PrinterCapabilities.feature()` 获取，不提供公共构造器。`implementation` 描述 SDK 路径是否实现，`deviceSupport` 描述设备支持是否已知，`evidence` 是证据位掩码；三个维度不能相互替代。当前显式 CPCL/ESC-POS 配置的已知功能可能仅有 PROFILE_DECLARED 且设备支持为 UNKNOWN，因此 `isImplemented()` 可为 true、`isSupported()` 仍为 false。未知功能 ID 返回未实现、未知设备支持和 evidence=0。

文档 ID：`android:type:com.netumscan.printersdk.FeatureSupport`

<a id="api-5adf34cff79f8a28b5fc"></a>

### public boolean isImplemented()

判断 SDK 是否存在该功能的实现路径。

```java
public boolean isImplemented();
```

无参数，返回 `implementation != NOT_IMPLEMENTED`。NATIVE 与 HOST_RENDERED 都返回 true；不读取设备、不执行 IO。该结果不检查 `deviceSupport` 或证据，不是当前硬件可用或打印已验收的保证。

文档 ID：`android:member:com.netumscan.printersdk.FeatureSupport:9f723c20b86e9048`

JVM descriptor：`()Z`

<a id="api-9648a867f14b13663a1a"></a>

### public boolean isSupported()

判断实现可用且设备支持已明确为支持。

```java
public boolean isSupported();
```

无参数，仅在 `isImplemented()` 为 true 且 `deviceSupport == SUPPORTED` 时返回 true。UNKNOWN 与 UNSUPPORTED 都返回 false，不能用 false 反推设备必定不支持。此判断不要求 HARDWARE_VERIFIED 位，仍应单独检查证据。

文档 ID：`android:member:com.netumscan.printersdk.FeatureSupport:751796123f4e2c80`

JVM descriptor：`()Z`

<a id="api-b69d65a56f8413e395f5"></a>

### public final int deviceSupport

该功能在设备上的支持知识状态。

```java
public final int deviceSupport;
```

当前定义为 UNKNOWN=0、UNSUPPORTED=1、SUPPORTED=2。UNKNOWN 表示没有足够证据，不等于 UNSUPPORTED；显式打印参数和协议编码存在也不能推导为 SUPPORTED。字段只读，配合 `implementation` 与 `evidence` 使用。

文档 ID：`android:member:com.netumscan.printersdk.FeatureSupport:6dfcca999e954a11`

JVM descriptor：`I`

<a id="api-0034bebc335217764927"></a>

### public final int evidence

支持声明的证据位掩码。

```java
public final int evidence;
```

PROFILE_DECLARED=1、DEVICE_REPORTED=2、HARDWARE_VERIFIED=4 可按位组合，0 表示没有对应证据。应使用按位与测试目标位；配置声明或设备自报均不能替代硬件验证。读取快照不会触发新的查询或验收。

文档 ID：`android:member:com.netumscan.printersdk.FeatureSupport:958eb37813714ec1`

JVM descriptor：`I`

<a id="api-99fa3cc41f8df96a5868"></a>

### public final int implementation

该功能的 SDK 实现类别整数。

```java
public final int implementation;
```

当前定义为 NOT_IMPLEMENTED=0、NATIVE=1、HOST_RENDERED=2。它描述实现路径而非设备能力；使用 `isImplemented()` 可保留对将来非零实现值的保守兼容。字段只读，读取不执行 IO。

文档 ID：`android:member:com.netumscan.printersdk.FeatureSupport:9ad48e1c07766934`

JVM descriptor：`I`

<a id="api-5ede70a7e57b8e7202c9"></a>

### public final int knownFields

功能knownFields位集合。

```java
public final int knownFields;
```

VALUES_KNOWN=1、REQUIRED_FIELDS_KNOWN=2、SUPPORTED_FIELDS_KNOWN=4，先检查此位再解释对应集合，未知不等于空。

文档 ID：`android:member:com.netumscan.printersdk.FeatureSupport:700caffac055e8b2`

JVM descriptor：`I`

<a id="api-c5a9c98790c8414e72cf"></a>

### public final int requiredFields

功能requiredFields位集合。

```java
public final int requiredFields;
```

介质设置必需字段mask，仅REQUIRED_FIELDS_KNOWN有效时解释；当前CPCL仅tracking。

文档 ID：`android:member:com.netumscan.printersdk.FeatureSupport:7da7b5176ab9cfb2`

JVM descriptor：`I`

<a id="api-09357b4d94c68c8db5dc"></a>

### public final int supportedFields

功能supportedFields位集合。

```java
public final int supportedFields;
```

介质允许字段mask，仅SUPPORTED_FIELDS_KNOWN有效时解释；未知不能推断全部可用。

文档 ID：`android:member:com.netumscan.printersdk.FeatureSupport:6a1a916a859f4e9f`

JVM descriptor：`I`

<a id="api-0eeee30ce1ebd05e100f"></a>

### public final int supportedValues

功能supportedValues位集合。

```java
public final int supportedValues;
```

功能允许的枚举值mask，仅VALUES_KNOWN有效时解释；CUT按FULL/PARTIAL，IMAGE按算法，MEDIA按tracking值。

文档 ID：`android:member:com.netumscan.printersdk.FeatureSupport:8e1bfa54fc237043`

JVM descriptor：`I`

<a id="api-2711890ae200c6e98573"></a>

### public static final int DEVICE_REPORTED = 2

DEVICE_REPORTED常量，值2。

```java
public static final int DEVICE_REPORTED = 2;
```

表示DEVICE_REPORTED对应的已知/存在字段mask；先检查known或present位再解释相关字段。 不执行IO，不表示硬件已验证支持。

文档 ID：`android:member:com.netumscan.printersdk.FeatureSupport:7384e00424860bf1`

JVM descriptor：`I`

<a id="api-29c5a143635d71a155f4"></a>

### public static final int HARDWARE_VERIFIED = 4

HARDWARE_VERIFIED常量，值4。

```java
public static final int HARDWARE_VERIFIED = 4;
```

表示HARDWARE_VERIFIED对应的已知/存在字段mask；先检查known或present位再解释相关字段。 不执行IO，不表示硬件已验证支持。

文档 ID：`android:member:com.netumscan.printersdk.FeatureSupport:a41dc2f2bd9542ff`

JVM descriptor：`I`

<a id="api-333b4ee110fff9669c75"></a>

### public static final int HOST_RENDERED = 2

HOST_RENDERED常量，值2。

```java
public static final int HOST_RENDERED = 2;
```

表示HOST_RENDERED对应的已知/存在字段mask；先检查known或present位再解释相关字段。 不执行IO，不表示硬件已验证支持。

文档 ID：`android:member:com.netumscan.printersdk.FeatureSupport:6e6ae85ee79f4469`

JVM descriptor：`I`

<a id="api-36b7b3a7bdf8d0253599"></a>

### public static final int NATIVE = 1

NATIVE常量，值1。

```java
public static final int NATIVE = 1;
```

表示NATIVE对应的已知/存在字段mask；先检查known或present位再解释相关字段。 不执行IO，不表示硬件已验证支持。

文档 ID：`android:member:com.netumscan.printersdk.FeatureSupport:9c09323ac8e6906f`

JVM descriptor：`I`

<a id="api-8f674851f75479a82ebf"></a>

### public static final int NOT_IMPLEMENTED = 0

NOT_IMPLEMENTED常量，值0。

```java
public static final int NOT_IMPLEMENTED = 0;
```

表示NOT_IMPLEMENTED对应的已知/存在字段mask；先检查known或present位再解释相关字段。 不执行IO，不表示硬件已验证支持。

文档 ID：`android:member:com.netumscan.printersdk.FeatureSupport:258ec4eedffc58f1`

JVM descriptor：`I`

<a id="api-7cb192a56c0db341863a"></a>

### public static final int PROFILE_DECLARED = 1

PROFILE_DECLARED常量，值1。

```java
public static final int PROFILE_DECLARED = 1;
```

表示PROFILE_DECLARED对应的已知/存在字段mask；先检查known或present位再解释相关字段。 不执行IO，不表示硬件已验证支持。

文档 ID：`android:member:com.netumscan.printersdk.FeatureSupport:296a3a40ae4f87ce`

JVM descriptor：`I`

<a id="api-1a137e5ddc2239a97711"></a>

### public static final int REQUIRED_FIELDS_KNOWN = 2

REQUIRED_FIELDS_KNOWN常量，值2。

```java
public static final int REQUIRED_FIELDS_KNOWN = 2;
```

表示REQUIRED_FIELDS_KNOWN对应的已知/存在字段mask；先检查known或present位再解释相关字段。 不执行IO，不表示硬件已验证支持。

文档 ID：`android:member:com.netumscan.printersdk.FeatureSupport:67c7e976e8b80178`

JVM descriptor：`I`

<a id="api-d39842a85f93d06ffa3a"></a>

### public static final int SUPPORTED = 2

SUPPORTED常量，值2。

```java
public static final int SUPPORTED = 2;
```

表示SUPPORTED对应的已知/存在字段mask；先检查known或present位再解释相关字段。 不执行IO，不表示硬件已验证支持。

文档 ID：`android:member:com.netumscan.printersdk.FeatureSupport:2660f6d171c11793`

JVM descriptor：`I`

<a id="api-96b5ed030acd084cf86a"></a>

### public static final int SUPPORTED_FIELDS_KNOWN = 4

SUPPORTED_FIELDS_KNOWN常量，值4。

```java
public static final int SUPPORTED_FIELDS_KNOWN = 4;
```

表示SUPPORTED_FIELDS_KNOWN对应的已知/存在字段mask；先检查known或present位再解释相关字段。 不执行IO，不表示硬件已验证支持。

文档 ID：`android:member:com.netumscan.printersdk.FeatureSupport:8b212cb35baa50a9`

JVM descriptor：`I`

<a id="api-b402a3b6e7c5f8404721"></a>

### public static final int UNKNOWN = 0

UNKNOWN常量，值0。

```java
public static final int UNKNOWN = 0;
```

表示UNKNOWN对应的已知/存在字段mask；先检查known或present位再解释相关字段。 不执行IO，不表示硬件已验证支持。

文档 ID：`android:member:com.netumscan.printersdk.FeatureSupport:12fee7f020c6e236`

JVM descriptor：`I`

<a id="api-d1ae2781a5eb8768e313"></a>

### public static final int UNSUPPORTED = 1

UNSUPPORTED常量，值1。

```java
public static final int UNSUPPORTED = 1;
```

表示UNSUPPORTED对应的已知/存在字段mask；先检查known或present位再解释相关字段。 不执行IO，不表示硬件已验证支持。

文档 ID：`android:member:com.netumscan.printersdk.FeatureSupport:cc394616ff0eb36c`

JVM descriptor：`I`

<a id="api-5f63e368a44431300b29"></a>

### public static final int VALUES_KNOWN = 1

VALUES_KNOWN常量，值1。

```java
public static final int VALUES_KNOWN = 1;
```

表示VALUES_KNOWN对应的已知/存在字段mask；先检查known或present位再解释相关字段。 不执行IO，不表示硬件已验证支持。

文档 ID：`android:member:com.netumscan.printersdk.FeatureSupport:2a8cd771661a8b86`

JVM descriptor：`I`

<a id="api-33ef44e6a0d34ed58fc9"></a>

## ImageLoadContext

后台加载的取消、时间与输出边界。

```java
public final class com.netumscan.printersdk.ImageLoadContext
```

仅在本次ImageLoader调用期间使用，不保留到其他请求。目标尺寸汇总同源所有用途，0表示该方向无采样约束，不是零大小；不能据此操作设备。

文档 ID：`android:type:com.netumscan.printersdk.ImageLoadContext`

<a id="api-308362fe482069af28cf"></a>

### public boolean isCancelled()

读取本次准备是否已失效。

```java
public boolean isCancelled();
```

用于合作退出，取消、超时和清理均可使准备失效；false不是保证随后不会取消。

文档 ID：`android:member:com.netumscan.printersdk.ImageLoadContext:f1f8e4ff91b49c6d`

JVM descriptor：`()Z`

<a id="api-5e88e32869e802448a4f"></a>

### public int targetHeightDots()

取得同源用途汇总的目标高度。

```java
public int targetHeightDots();
```

dots单位；0表示高度未限定或需要原始用途。该提示用于有界解码，最终画布与布局仍由ImageRender和作业校验。

文档 ID：`android:member:com.netumscan.printersdk.ImageLoadContext:1657af22be394aa2`

JVM descriptor：`()I`

<a id="api-7b2f9bd270bc5a8ad382"></a>

### public int targetWidthDots()

取得同源用途汇总的目标宽度。

```java
public int targetWidthDots();
```

dots单位；0表示至少一个用途需要原始宽度，禁止因此下采样。多次使用同源时取能满足全部用途的保守边界。

文档 ID：`android:member:com.netumscan.printersdk.ImageLoadContext:e6650f9427341e52`

JVM descriptor：`()I`

<a id="api-f80bd52f29f286c3e716"></a>

### public long maxOutputBytes()

取得单图输出像素字节上限。

```java
public long maxOutputBytes();
```

当前为16MiB，此外仍受32MiB整请求及64MiBManager准备工作集限制；不能只按一个输出上限估计临时解码峰值。

文档 ID：`android:member:com.netumscan.printersdk.ImageLoadContext:e6dff6d342cc6ff9`

JVM descriptor：`()J`

<a id="api-1e4d0ae0f61b7c4d02d0"></a>

### public long remainingTimeMs()

取得本次请求的剩余时间。

```java
public long remainingTimeMs();
```

毫秒预算包含Manager排队与准备，归零表示预算已耗尽；不能用新的完整超时重置原请求截止时间。

文档 ID：`android:member:com.netumscan.printersdk.ImageLoadContext:e0a63064191a9f55`

JVM descriptor：`()J`

<a id="api-0b842c7fc89d45d2829c"></a>

### public void throwIfCancelled()

取消或超时时合作退出。

```java
public void throwIfCancelled();
```

加载循环和阶段边界调用；异常由本次Manager请求分类并经原Callback交付，不把提前终态当作加载器已退出。

文档 ID：`android:member:com.netumscan.printersdk.ImageLoadContext:871ea18d6bc6f0cb`

JVM descriptor：`()V`

<a id="api-d2a820217deb7df1eae0"></a>

## ImageLoader

可协作取消的后台图片加载函数。

```java
public interface com.netumscan.printersdk.ImageLoader
```

每次提交可重新生成内容，必须返回非空不可变PixelImage。自行关闭输入流，定期检查取消；SDK不接管任意闭包资源或保证强行终止不合作代码。

文档 ID：`android:type:com.netumscan.printersdk.ImageLoader`

编译器注解：`@java.lang.FunctionalInterface`。

<a id="api-59c080042b15843cd9a8"></a>

### public abstract com.netumscan.printersdk.PixelImage load(com.netumscan.printersdk.ImageLoadContext) throws java.lang.Exception

在SDK后台加载一个不可变图像。

```java
public abstract com.netumscan.printersdk.PixelImage load(com.netumscan.printersdk.ImageLoadContext) throws java.lang.Exception;
```

同一提交中同一来源仅调用一次，其他提交可再次调用。返回非空PixelImage，支持协作取消和剩余时间；流及外部资源在finally关闭，错误经原请求Callback分类。

文档 ID：`android:member:com.netumscan.printersdk.ImageLoader:78b56b337e41c7ee`

JVM descriptor：`(Lcom/netumscan/printersdk/ImageLoadContext;)Lcom/netumscan/printersdk/PixelImage;`

编译器注解：`@androidx.annotation.NonNull`、`@androidx.annotation.WorkerThread`。

参数 0（从 0 起）注解：`@androidx.annotation.NonNull`。

<a id="api-c7ebed100a5f88acba1d"></a>

## ImageRender

不可变的逐图处理配置。

```java
public final class com.netumscan.printersdk.ImageRender
```

默认阈值128、不反色不缩放；缩放仅等比缩小，不放大不隐式裁剪。RGBA按白底合成，毫米在提交时按有效Profile固定换算。

文档 ID：`android:type:com.netumscan.printersdk.ImageRender`

<a id="api-0867f5068d8571d254bb"></a>

### public boolean equals(java.lang.Object)

比较完整图像处理值。

```java
public boolean equals(java.lang.Object);
```

比较算法、阈值、反色、缩放模式及尺寸单位；用于同源同处理复用，不按来源内容或设备身份比较。

文档 ID：`android:member:com.netumscan.printersdk.ImageRender:3a3dac16122e0202`

JVM descriptor：`(Ljava/lang/Object;)Z`

<a id="api-b91377be5c14ddbeab7d"></a>

### public com.netumscan.printersdk.ImageRender containBoxDots(int, int)

等比缩小到dots画布并居中补白。

```java
public com.netumscan.printersdk.ImageRender containBoxDots(int, int);
```

两边须正数；不放大、不裁剪，最终图像尺寸为指定画布。额外输出分配计入单图、作业及Manager工作集预算。

文档 ID：`android:member:com.netumscan.printersdk.ImageRender:9e559e9202978928`

JVM descriptor：`(II)Lcom/netumscan/printersdk/ImageRender;`

编译器注解：`@androidx.annotation.NonNull`。

<a id="api-329c5075f2d708ecdb53"></a>

### public com.netumscan.printersdk.ImageRender containBoxMm(double, double)

等比缩小到毫米画布并居中补白。

```java
public com.netumscan.printersdk.ImageRender containBoxMm(double, double);
```

参数须正有限数；提交时按固定Profile的dotsPerMm以round(mm*dotsPerMm)换算，与标签尺寸及解码目标保持一致。拒绝归零与溢出，文件DPI不参与；图像在画布内居中与整个标签的设备定位规则独立。

文档 ID：`android:member:com.netumscan.printersdk.ImageRender:fa73ac7039f24573`

JVM descriptor：`(DD)Lcom/netumscan/printersdk/ImageRender;`

编译器注解：`@androidx.annotation.NonNull`。

<a id="api-c1ecb959051435017e19"></a>

### public com.netumscan.printersdk.ImageRender fitWidthDots(int)

按dots宽度等比缩小。

```java
public com.netumscan.printersdk.ImageRender fitWidthDots(int);
```

宽度须正数；来源较小时不放大，不裁剪，最终高度保持比例。输出仍受画布、设备及字节补齐宽度检查。

文档 ID：`android:member:com.netumscan.printersdk.ImageRender:fef5749d3cd4036e`

JVM descriptor：`(I)Lcom/netumscan/printersdk/ImageRender;`

编译器注解：`@androidx.annotation.NonNull`。

<a id="api-4e29d246ac6f3bc182eb"></a>

### public com.netumscan.printersdk.ImageRender invert(boolean)

读取或配置光栅反色。

```java
public com.netumscan.printersdk.ImageRender invert(boolean);
```

fluent方法返回新不可变值，不修改原对象；反色独立于缩放和算法，不修改输入像素或设备状态。

文档 ID：`android:member:com.netumscan.printersdk.ImageRender:7753a9192da3da79`

JVM descriptor：`(Z)Lcom/netumscan/printersdk/ImageRender;`

编译器注解：`@androidx.annotation.NonNull`。

<a id="api-1a4f6bd081b178add338"></a>

### public final boolean invert

读取或配置光栅反色。

```java
public final boolean invert;
```

fluent方法返回新不可变值，不修改原对象；反色独立于缩放和算法，不修改输入像素或设备状态。

文档 ID：`android:member:com.netumscan.printersdk.ImageRender:702f6f1660fce0d8`

JVM descriptor：`Z`

<a id="api-729656cda149cd1a736e"></a>

### public final int algorithm

所采用的二值化算法值。

```java
public final int algorithm;
```

对应既有THRESHOLD、FLOYD_STEINBERG、BAYER_4X4或BAYER_8X8常量；不表示缩放、文件格式或传输分块。

文档 ID：`android:member:com.netumscan.printersdk.ImageRender:6d8363603cc9fb29`

JVM descriptor：`I`

<a id="api-7cfd1079fb5185ec687c"></a>

### public final int threshold

采用固定阈值进行二值化。

```java
public final int threshold;
```

工厂参数为0..255，否则同步参数异常；默认128。字段表示所采用阈值；其他抖动工厂使用128占位但按所选算法处理。

文档 ID：`android:member:com.netumscan.printersdk.ImageRender:b0f72a281f2c8781`

JVM descriptor：`I`

<a id="api-1b8224d9161cde6d29aa"></a>

### public int hashCode()

取得处理值的稳定哈希。

```java
public int hashCode();
```

与equals包含相同处理字段，适用于值键；不是图像内容摘要或请求身份。

文档 ID：`android:member:com.netumscan.printersdk.ImageRender:5646c2b8b79202af`

JVM descriptor：`()I`

<a id="api-d729335332ca31232382"></a>

### public static com.netumscan.printersdk.ImageRender bayer4x4()

采用4×4 Bayer有序抖动。

```java
public static com.netumscan.printersdk.ImageRender bayer4x4();
```

返回不可变处理值，适合对照同一来源的算法效果；不触发解码或打印。

文档 ID：`android:member:com.netumscan.printersdk.ImageRender:2ab26b4e554bdc59`

JVM descriptor：`()Lcom/netumscan/printersdk/ImageRender;`

编译器注解：`@androidx.annotation.NonNull`。

<a id="api-96049babf8b0116bb138"></a>

### public static com.netumscan.printersdk.ImageRender bayer8x8()

采用8×8 Bayer有序抖动。

```java
public static com.netumscan.printersdk.ImageRender bayer8x8();
```

返回不可变处理值，保留独立的缩放和反色参数，不触发解码或打印。

文档 ID：`android:member:com.netumscan.printersdk.ImageRender:06b70f7f0128456a`

JVM descriptor：`()Lcom/netumscan/printersdk/ImageRender;`

编译器注解：`@androidx.annotation.NonNull`。

<a id="api-9663eca444c497a6dae4"></a>

### public static com.netumscan.printersdk.ImageRender floydSteinberg()

采用Floyd-Steinberg误差扩散。

```java
public static com.netumscan.printersdk.ImageRender floydSteinberg();
```

返回不可变处理值，不在工厂计算图片；后续逐条带光栅处理仍独立于协议与传输。

文档 ID：`android:member:com.netumscan.printersdk.ImageRender:edbc8d849bbb61eb`

JVM descriptor：`()Lcom/netumscan/printersdk/ImageRender;`

编译器注解：`@androidx.annotation.NonNull`。

<a id="api-1544a80aef88c6a6d182"></a>

### public static com.netumscan.printersdk.ImageRender threshold(int)

采用固定阈值进行二值化。

```java
public static com.netumscan.printersdk.ImageRender threshold(int);
```

工厂参数为0..255，否则同步参数异常；默认128。字段表示所采用阈值；其他抖动工厂使用128占位但按所选算法处理。

文档 ID：`android:member:com.netumscan.printersdk.ImageRender:959ef4b290b88f40`

JVM descriptor：`(I)Lcom/netumscan/printersdk/ImageRender;`

编译器注解：`@androidx.annotation.NonNull`。

<a id="api-8283abbc7b9b40cd9caf"></a>

## ImageSource

有读取引用保护的只读图片来源。

```java
public final class com.netumscan.printersdk.ImageSource implements java.lang.AutoCloseable
```

关闭App引用幂等，已接收请求继续持有各自读取引用。最后真实读取退出才安排一次后台释放hook；close、终态和文件删除不是同一时间点。

文档 ID：`android:type:com.netumscan.printersdk.ImageSource`

<a id="api-baf094271cfce3ff40a5"></a>

### public static com.netumscan.printersdk.ImageSource deferred(com.netumscan.printersdk.ImageLoader)

创建可重复后台加载的来源。

```java
public static com.netumscan.printersdk.ImageSource deferred(com.netumscan.printersdk.ImageLoader);
```

loader非空，每次提交可生成不同内容。带onReleased时显式转移一个注册租约，最后真实读取退出后在专用release worker锁外异步执行一次；不关闭任意闭包捕获资源，close不保证hook已运行。

文档 ID：`android:member:com.netumscan.printersdk.ImageSource:388cb9cef72153b9`

JVM descriptor：`(Lcom/netumscan/printersdk/ImageLoader;)Lcom/netumscan/printersdk/ImageSource;`

编译器注解：`@androidx.annotation.NonNull`。

参数 0（从 0 起）注解：`@androidx.annotation.NonNull`。

<a id="api-5651dea9d584d40b3e90"></a>

### public static com.netumscan.printersdk.ImageSource deferred(com.netumscan.printersdk.ImageLoader, java.lang.Runnable)

创建可重复后台加载的来源。

```java
public static com.netumscan.printersdk.ImageSource deferred(com.netumscan.printersdk.ImageLoader, java.lang.Runnable);
```

loader非空，每次提交可生成不同内容。带onReleased时显式转移一个注册租约，最后真实读取退出后在专用release worker锁外异步执行一次；不关闭任意闭包捕获资源，close不保证hook已运行。

文档 ID：`android:member:com.netumscan.printersdk.ImageSource:d6cc07ec540bee65`

JVM descriptor：`(Lcom/netumscan/printersdk/ImageLoader;Ljava/lang/Runnable;)Lcom/netumscan/printersdk/ImageSource;`

编译器注解：`@androidx.annotation.NonNull`。

参数 0（从 0 起）注解：`@androidx.annotation.NonNull`。

参数 1（从 0 起）注解：`@androidx.annotation.NonNull`。

<a id="api-12e88f848fd82d43bece"></a>

### public static com.netumscan.printersdk.ImageSource pixels(com.netumscan.printersdk.PixelImage)

以不可变PixelImage创建可复用来源。

```java
public static com.netumscan.printersdk.ImageSource pixels(com.netumscan.printersdk.PixelImage);
```

不再次复制像素；提交后保留读取引用，App关闭来源不会取消已接收请求，来源关闭后不能新提交。

文档 ID：`android:member:com.netumscan.printersdk.ImageSource:5e717ee7902958d4`

JVM descriptor：`(Lcom/netumscan/printersdk/PixelImage;)Lcom/netumscan/printersdk/ImageSource;`

编译器注解：`@androidx.annotation.NonNull`。

参数 0（从 0 起）注解：`@androidx.annotation.NonNull`。

<a id="api-61ce8c0f4ec55e293b2f"></a>

### public void close()

幂等释放App持有的来源引用。

```java
public void close();
```

不取消已接收工作，也不立即删除正在读的文件。最后真实读取释放后才异步运行hook；不会在UI或SDK状态锁中执行客户释放代码。

文档 ID：`android:member:com.netumscan.printersdk.ImageSource:71f6e6626cb069d2`

JVM descriptor：`()V`

<a id="api-cc7a220484daaf56ad23"></a>

## LTM90Profile

提供 LT-M90 的 BLE 与 CPCL 配置工厂。

```java
public final class com.netumscan.printersdk.LTM90Profile
```

静态工厂命名空间，不提供可创建的实例。BLE 采用固件 V1.2 文档中的 FF00／FF02 和 FF01／FF03 无响应写入及 Credit；CPCL 固定已确认的 203 DPI、864 dots 和水平原点 0。最大标签页高与 FORM 由调用方显式提供，介质和业务布局继续逐操作传入。工厂本身不进行型号识别、连接或 IO，也不代表硬件验收通过。

文档 ID：`android:type:com.netumscan.printersdk.LTM90Profile`

<a id="api-a2f1ce0fbd9ddee3aaed"></a>

### public static com.netumscan.printersdk.BleOptions bleOptions()

创建 LT-M90 默认 BLE 连接配置。

```java
public static com.netumscan.printersdk.BleOptions bleOptions();
```

等价于 `bleOptions(517, 512)`：MTU 请求 517 字节、写入上限 512 字节，FF00 服务、FF02 写入、FF01 数据通知和 FF03 Credit 通知，使用标准 Bluetooth Base UUID。无响应写入并启用 Credit；实际包长仍受平台协商、固件公告和应用上限约束。同步创建新对象，不进行连接、订阅或 IO，不证明硬件支持。

文档 ID：`android:member:com.netumscan.printersdk.LTM90Profile:be72c618d4be961d`

JVM descriptor：`()Lcom/netumscan/printersdk/BleOptions;`

<a id="api-685fbcf42b2b384fffc1"></a>

### public static com.netumscan.printersdk.BleOptions bleOptions(int, int)

创建 LT-M90 V1.2 无响应写入与信用流控配置。

```java
public static com.netumscan.printersdk.BleOptions bleOptions(int, int);
```

参数 `requestedMtu` 为 23..517 字节，`maxWriteBytes` 为 1..512 字节；非法值由 `BleOptions` 抛出 `IllegalArgumentException`。返回非 null `BleOptions`：服务 `0000ff00-0000-1000-8000-00805f9b34fb`、写特征 FF02、数据通知 FF01、信用通知 FF03（后三者使用相同蓝牙基础 UUID），`withResponse=false`。同步创建，不连接设备；预设仅依据固件文档，不证明硬件支持。

文档 ID：`android:member:com.netumscan.printersdk.LTM90Profile:fadbe287c9213756`

JVM descriptor：`(II)Lcom/netumscan/printersdk/BleOptions;`

<a id="api-538a47feb0d8919ec7d0"></a>

### public static com.netumscan.printersdk.CpclProfile cpclProfile(int, boolean)

创建 LT-M90 的 CPCL 打印预设。

```java
public static com.netumscan.printersdk.CpclProfile cpclProfile(int, boolean);
```

固定 DPI 203、设备最大打印宽度 864 dots、水平原点 0；`maxLabelHeightDots` 与 `form` 均由调用方传入，不把未知安全最大页高写成型号默认值。页高接受 1..65535 点，否则由 `CpclProfile` 抛 `IllegalArgumentException`。返回新对象，不执行 IO、介质设置、自动居中或布局；实际图像宽度及纸面适配仍由应用确定。

文档 ID：`android:member:com.netumscan.printersdk.LTM90Profile:c371b4cd4d81f218`

JVM descriptor：`(IZ)Lcom/netumscan/printersdk/CpclProfile;`

<a id="api-d54c33ceee2b4b4753fc"></a>

### public static com.netumscan.printersdk.PrinterProfile printProfile()

取得LT-M90高层打印配置。

```java
public static com.netumscan.printersdk.PrinterProfile printProfile();
```

返回CPCL、名义203DPI、最大864dots、精确8dots/mm和CENTERED。依据用户提供的有效打印宽度108mm／864dots及必须居中装纸结构；标签尺寸和校准逐请求提供。工厂无IO；旧cpclProfile纯值兼容工厂保持原点0，不能替代Manager的高层配置。

文档 ID：`android:member:com.netumscan.printersdk.LTM90Profile:5c1b37703fdd46d7`

JVM descriptor：`()Lcom/netumscan/printersdk/PrinterProfile;`

<a id="api-98f169a3e4c7b889744f"></a>

## ManagerResult

单次请求的不可变完成结果。

```java
public final class com.netumscan.printersdk.ManagerResult<T>
```

包含请求标识、成功/失败/取消、稳定错误码、辅助消息、完成证据、可能已输出标记及可选业务值。主线程回调恰好完成一次；dataSent仅证明数据发送完成，不证明纸张已打印。 引用字段均声明空值约束：`value` 为 `@Nullable`，其他引用字段为 `@NonNull`；Void 结果或失败结果不要求业务值存在。

文档 ID：`android:type:com.netumscan.printersdk.ManagerResult`

<a id="api-9decaa39a103f3afbb6b"></a>

### public final T value

成功结果的可选业务值。

```java
public final T value;
```

用于设备列表、能力、身份和状态查询；没有返回值的请求为Void/null，失败时不要假定业务值存在。

文档 ID：`android:member:com.netumscan.printersdk.ManagerResult:60258455367224f8`

JVM descriptor：`Ljava/lang/Object;`

编译器注解：`@androidx.annotation.Nullable`。

<a id="api-356bea50d55103f0d6ac"></a>

### public final boolean outputMayHaveOccurred

是否可能已发送了不可撤回的输出。

```java
public final boolean outputMayHaveOccurred;
```

true时设备可能已经收到部分命令或像素；应用不得据此自动重试或重放，需自行决定提示和后续人工操作。

文档 ID：`android:member:com.netumscan.printersdk.ManagerResult:a134b771ae6ec59f`

JVM descriptor：`Z`

<a id="api-ea8d4b6c1a2baff62e9d"></a>

### public final com.netumscan.printersdk.ManagerResult$Completion completion

打印完成证据级别。

```java
public final com.netumscan.printersdk.ManagerResult$Completion completion;
```

none表示没有打印完成证据，dataSent表示传输完成，deviceConfirmed仅在协议提供明确设备确认时使用。

文档 ID：`android:member:com.netumscan.printersdk.ManagerResult:97ce1429538d8a64`

JVM descriptor：`Lcom/netumscan/printersdk/ManagerResult$Completion;`

编译器注解：`@androidx.annotation.NonNull`。

<a id="api-b511db0e6dff799b48b8"></a>

### public final com.netumscan.printersdk.ManagerResult$Outcome outcome

请求成功、失败或取消的分类。

```java
public final com.netumscan.printersdk.ManagerResult$Outcome outcome;
```

与稳定code及完成证据一起读取；取消或失败不自动表示设备没有产生部分输出。

文档 ID：`android:member:com.netumscan.printersdk.ManagerResult:c2dc63bbace485a2`

JVM descriptor：`Lcom/netumscan/printersdk/ManagerResult$Outcome;`

编译器注解：`@androidx.annotation.NonNull`。

<a id="api-e4a0bf43f650f01efaeb"></a>

### public final com.netumscan.printersdk.RequestId requestId

请求标识。

```java
public final com.netumscan.printersdk.RequestId requestId;
```

仅能由创建该标识的Manager识别；用于关联完成结果和发送取消意图。

文档 ID：`android:member:com.netumscan.printersdk.ManagerResult:e82f6b58f9864f0c`

JVM descriptor：`Lcom/netumscan/printersdk/RequestId;`

编译器注解：`@androidx.annotation.NonNull`。

<a id="api-4fe8239624fcc5ce1537"></a>

### public final int code

稳定SDK错误码。

```java
public final int code;
```

0表示成功，其他值按SDK错误契约解释；不要依赖可读message进行程序分支。

文档 ID：`android:member:com.netumscan.printersdk.ManagerResult:c87585a61ac30812`

JVM descriptor：`I`

<a id="api-94755314530c221bb6bc"></a>

### public final java.lang.String message

可读的辅助错误消息。

```java
public final java.lang.String message;
```

用于集成诊断和界面提示；程序逻辑以稳定code为主，不解析该文本。

文档 ID：`android:member:com.netumscan.printersdk.ManagerResult:62f72828fbf07916`

JVM descriptor：`Ljava/lang/String;`

编译器注解：`@androidx.annotation.NonNull`。

<a id="api-1721ae5f9f8202edb30d"></a>

## ManagerResult.Completion

打印完成证据级别。

```java
public final class com.netumscan.printersdk.ManagerResult$Completion extends java.lang.Enum<com.netumscan.printersdk.ManagerResult$Completion>
```

NONE/none表示没有打印完成证据，DATA_SENT/dataSent表示传输完成，DEVICE_CONFIRMED/deviceConfirmed仅在协议提供明确设备确认时使用。

文档 ID：`android:type:com.netumscan.printersdk.ManagerResult$Completion`

枚举声明顺序（`ordinal()` 从 0 起；不作为持久化协议值）：

| ordinal | 枚举项 |
| --- | --- |
| 0 | `NONE` |
| 1 | `DATA_SENT` |
| 2 | `DEVICE_CONFIRMED` |

<a id="api-6d97921c294a639254f4"></a>

### public static com.netumscan.printersdk.ManagerResult$Completion valueOf(java.lang.String)

按精确名称读取枚举常量。

```java
public static com.netumscan.printersdk.ManagerResult$Completion valueOf(java.lang.String);
```

由Java编译器生成；名称不匹配时遵循Java标准枚举异常行为。

文档 ID：`android:member:com.netumscan.printersdk.ManagerResult$Completion:b1c46fd2ef782e48`

JVM descriptor：`(Ljava/lang/String;)Lcom/netumscan/printersdk/ManagerResult$Completion;`

这是 Java 编译器为枚举生成的公共方法。

<a id="api-352c269dc1c32a562773"></a>

### public static com.netumscan.printersdk.ManagerResult$Completion[] values()

枚举声明顺序中的全部常量。

```java
public static com.netumscan.printersdk.ManagerResult$Completion[] values();
```

由Java编译器生成，返回新数组；常量名称和值属于本版本公开契约。

文档 ID：`android:member:com.netumscan.printersdk.ManagerResult$Completion:bbc424685d75dbb8`

JVM descriptor：`()[Lcom/netumscan/printersdk/ManagerResult$Completion;`

这是 Java 编译器为枚举生成的公共方法。

<a id="api-290d9cf4014b21ab8076"></a>

### public static final com.netumscan.printersdk.ManagerResult$Completion DATA_SENT

仅数据发送完成。

```java
public static final com.netumscan.printersdk.ManagerResult$Completion DATA_SENT;
```

该常量的精确名称、原始值或条件位属于本版本契约；完成证据不得被解释为强于实际协议可证明的级别。

文档 ID：`android:member:com.netumscan.printersdk.ManagerResult$Completion:25379bc32a34dbdc`

JVM descriptor：`Lcom/netumscan/printersdk/ManagerResult$Completion;`

<a id="api-544156fc54fe7a081b86"></a>

### public static final com.netumscan.printersdk.ManagerResult$Completion DEVICE_CONFIRMED

设备明确确认完成。

```java
public static final com.netumscan.printersdk.ManagerResult$Completion DEVICE_CONFIRMED;
```

该常量的精确名称、原始值或条件位属于本版本契约；完成证据不得被解释为强于实际协议可证明的级别。

文档 ID：`android:member:com.netumscan.printersdk.ManagerResult$Completion:0f0213ab646475bb`

JVM descriptor：`Lcom/netumscan/printersdk/ManagerResult$Completion;`

<a id="api-40186482712649e8f8f2"></a>

### public static final com.netumscan.printersdk.ManagerResult$Completion NONE

没有完成证据。

```java
public static final com.netumscan.printersdk.ManagerResult$Completion NONE;
```

该常量的精确名称、原始值或条件位属于本版本契约；完成证据不得被解释为强于实际协议可证明的级别。

文档 ID：`android:member:com.netumscan.printersdk.ManagerResult$Completion:b2288a2b928b5668`

JVM descriptor：`Lcom/netumscan/printersdk/ManagerResult$Completion;`

<a id="api-2784cdf665d09aa3045c"></a>

## ManagerResult.Outcome

请求的终态分类。

```java
public final class com.netumscan.printersdk.ManagerResult$Outcome extends java.lang.Enum<com.netumscan.printersdk.ManagerResult$Outcome>
```

SUCCESS/success、FAILURE/failure和CANCELLED/cancelled分别表示成功、失败及取消；同时检查稳定code和输出证据以决定应用提示。

文档 ID：`android:type:com.netumscan.printersdk.ManagerResult$Outcome`

枚举声明顺序（`ordinal()` 从 0 起；不作为持久化协议值）：

| ordinal | 枚举项 |
| --- | --- |
| 0 | `SUCCESS` |
| 1 | `FAILURE` |
| 2 | `CANCELLED` |

<a id="api-63c73722c03dc5aa768a"></a>

### public static com.netumscan.printersdk.ManagerResult$Outcome valueOf(java.lang.String)

按精确名称读取枚举常量。

```java
public static com.netumscan.printersdk.ManagerResult$Outcome valueOf(java.lang.String);
```

由Java编译器生成；名称不匹配时遵循Java标准枚举异常行为。

文档 ID：`android:member:com.netumscan.printersdk.ManagerResult$Outcome:e5d94c2fff5d9609`

JVM descriptor：`(Ljava/lang/String;)Lcom/netumscan/printersdk/ManagerResult$Outcome;`

这是 Java 编译器为枚举生成的公共方法。

<a id="api-37db37ba405e67a9a9e4"></a>

### public static com.netumscan.printersdk.ManagerResult$Outcome[] values()

枚举声明顺序中的全部常量。

```java
public static com.netumscan.printersdk.ManagerResult$Outcome[] values();
```

由Java编译器生成，返回新数组；常量名称和值属于本版本公开契约。

文档 ID：`android:member:com.netumscan.printersdk.ManagerResult$Outcome:48814b7d3efdb6ee`

JVM descriptor：`()[Lcom/netumscan/printersdk/ManagerResult$Outcome;`

这是 Java 编译器为枚举生成的公共方法。

<a id="api-d7e9bb44403ab6f37534"></a>

### public static final com.netumscan.printersdk.ManagerResult$Outcome CANCELLED

请求取消。

```java
public static final com.netumscan.printersdk.ManagerResult$Outcome CANCELLED;
```

该常量的精确名称及编译值由公开基线固定；使用与本类型匹配的字段或参数，能力是否受支持以profile描述和请求校验为准。

文档 ID：`android:member:com.netumscan.printersdk.ManagerResult$Outcome:6dc2c8acb95c7598`

JVM descriptor：`Lcom/netumscan/printersdk/ManagerResult$Outcome;`

<a id="api-851e4da4067f58b66f35"></a>

### public static final com.netumscan.printersdk.ManagerResult$Outcome FAILURE

请求失败。

```java
public static final com.netumscan.printersdk.ManagerResult$Outcome FAILURE;
```

该常量的精确名称及编译值由公开基线固定；使用与本类型匹配的字段或参数，能力是否受支持以profile描述和请求校验为准。

文档 ID：`android:member:com.netumscan.printersdk.ManagerResult$Outcome:e3c5f749b2c365b7`

JVM descriptor：`Lcom/netumscan/printersdk/ManagerResult$Outcome;`

<a id="api-515dec3613d752c82ce0"></a>

### public static final com.netumscan.printersdk.ManagerResult$Outcome SUCCESS

请求成功。

```java
public static final com.netumscan.printersdk.ManagerResult$Outcome SUCCESS;
```

该常量的精确名称及编译值由公开基线固定；使用与本类型匹配的字段或参数，能力是否受支持以profile描述和请求校验为准。

文档 ID：`android:member:com.netumscan.printersdk.ManagerResult$Outcome:458dd1ddc909dc38`

JVM descriptor：`Lcom/netumscan/printersdk/ManagerResult$Outcome;`

<a id="api-4b74ae3f0e52d0db0f65"></a>

## MediaMode

与打印语言独立的介质跟踪方式枚举。

```java
public final class com.netumscan.printersdk.MediaMode extends java.lang.Enum<com.netumscan.printersdk.MediaMode>
```

用于 `setMedia()` 或兼容 `setMediaMode()`，当前 CPCL 后端实现跟踪模式命令，ESC/POS 后端拒绝。选择枚举不会声明纸宽、纸长、间隙长度或传感器校准结果；调用方必须装入适当介质，并核实固件兼容性。枚举的 Java ordinal 不是公共协议编码，不应自行用于拼接命令。

文档 ID：`android:type:com.netumscan.printersdk.MediaMode`

枚举声明顺序（`ordinal()` 从 0 起；不作为持久化协议值）：

| ordinal | 枚举项 |
| --- | --- |
| 0 | `GAP` |
| 1 | `BLACK_MARK` |
| 2 | `CONTINUOUS` |

<a id="api-f964ddb8df3fb623eb9a"></a>

### public static com.netumscan.printersdk.MediaMode valueOf(java.lang.String)

按精确枚举名称查找介质模式。

```java
public static com.netumscan.printersdk.MediaMode valueOf(java.lang.String);
```

参数 `name` 必须为非 null 且大小写精确的 `GAP`、`BLACK_MARK` 或 `CONTINUOUS`；返回对应枚举常量。Java 自动生成此方法：null 抛出 `NullPointerException`，未匹配名称抛出 `IllegalArgumentException`。不按显示名称、数字或 ordinal 解析，不执行 IO。

文档 ID：`android:member:com.netumscan.printersdk.MediaMode:25dac0f8c6207093`

JVM descriptor：`(Ljava/lang/String;)Lcom/netumscan/printersdk/MediaMode;`

这是 Java 编译器为枚举生成的公共方法。

<a id="api-5e48053988753ec90a1e"></a>

### public static com.netumscan.printersdk.MediaMode[] values()

返回按声明顺序排列的介质枚举数组。

```java
public static com.netumscan.printersdk.MediaMode[] values();
```

无参数，返回新数组 `[GAP, BLACK_MARK, CONTINUOUS]`。这是 Java 枚举自动生成的辅助方法；修改返回数组不改变枚举常量集合。顺序仅用于枚举展示，不能推断设备支持或协议数值。

文档 ID：`android:member:com.netumscan.printersdk.MediaMode:e6138ae9aa44ee4e`

JVM descriptor：`()[Lcom/netumscan/printersdk/MediaMode;`

这是 Java 编译器为枚举生成的公共方法。

<a id="api-011c7bd7bc5fb2bb6e29"></a>

### public static final com.netumscan.printersdk.MediaMode BLACK_MARK

黑标介质的跟踪模式。

```java
public static final com.netumscan.printersdk.MediaMode BLACK_MARK;
```

用于已装入黑标介质的 CPCL 跟踪设置；不测量标记长度、偏移或纸张位置，不执行传感器校准。枚举存在不表示所有设备固件支持该模式。

文档 ID：`android:member:com.netumscan.printersdk.MediaMode:f19064fdd38fb10e`

JVM descriptor：`Lcom/netumscan/printersdk/MediaMode;`

<a id="api-13c48e699eb17d12cb27"></a>

### public static final com.netumscan.printersdk.MediaMode CONTINUOUS

连续纸的介质跟踪模式。

```java
public static final com.netumscan.printersdk.MediaMode CONTINUOUS;
```

用于 CPCL 连续介质跟踪设置；不会自动将标签作业转换为 ESC/POS 小票，也不控制切刀。应用须提供匹配的纸张和打印参数。

文档 ID：`android:member:com.netumscan.printersdk.MediaMode:cf069c583bcf0729`

JVM descriptor：`Lcom/netumscan/printersdk/MediaMode;`

<a id="api-904ffa7e8f2a61c5f45d"></a>

### public static final com.netumscan.printersdk.MediaMode GAP

间隙标签的介质跟踪模式。

```java
public static final com.netumscan.printersdk.MediaMode GAP;
```

用于已装入间隙标签的 CPCL 跟踪设置；不自动测量间隙、设置标签尺寸或执行校准。调用 `setMedia()` 仍可能因设备/协议不支持而失败，成功只证明命令数据已发送。

文档 ID：`android:member:com.netumscan.printersdk.MediaMode:4b03b38b1cbe2f6a`

JVM descriptor：`Lcom/netumscan/printersdk/MediaMode;`

<a id="api-2f5be4c6d23e94746056"></a>

## MediaRequirement

作业提交的显式介质前置条件。

```java
public final class com.netumscan.printersdk.MediaRequirement
```

包含非null MediaSettings和证据策略，不自动设置介质。当前仅CPCL tracking+ALLOW_LAST_SENT可用；REQUIRE_READBACK及其他字段在提交前1004拒绝，不自动降级。

文档 ID：`android:type:com.netumscan.printersdk.MediaRequirement`

<a id="api-934cea35f180203cee46"></a>

### public com.netumscan.printersdk.MediaRequirement(com.netumscan.printersdk.MediaSettings, int)

保存介质前置条件及证据策略。

```java
public com.netumscan.printersdk.MediaRequirement(com.netumscan.printersdk.MediaSettings, int);
```

settings不可null；policy只接受ALLOW_LAST_SENT=1或REQUIRE_READBACK=2，否则IllegalArgumentException。不执行IO；当前仅CPCL tracking+LAST_SENT可提交，读回在IO前1004拒绝。

文档 ID：`android:member:com.netumscan.printersdk.MediaRequirement:9b01646a36c59049`

JVM descriptor：`(Lcom/netumscan/printersdk/MediaSettings;I)V`

<a id="api-075e118dcce2937e38f1"></a>

### public final com.netumscan.printersdk.MediaSettings settings

介质前置条件的settings。

```java
public final com.netumscan.printersdk.MediaSettings settings;
```

settings为非null不可变MediaSettings；verificationPolicy为ALLOW_LAST_SENT=1或REQUIRE_READBACK=2。仅描述要求，不执行设置；当前读回策略不支持。

文档 ID：`android:member:com.netumscan.printersdk.MediaRequirement:c1088016778c7033`

JVM descriptor：`Lcom/netumscan/printersdk/MediaSettings;`

<a id="api-6b3c71590e4b08e71543"></a>

### public final int verificationPolicy

介质前置条件的verificationPolicy。

```java
public final int verificationPolicy;
```

settings为非null不可变MediaSettings；verificationPolicy为ALLOW_LAST_SENT=1或REQUIRE_READBACK=2。仅描述要求，不执行设置；当前读回策略不支持。

文档 ID：`android:member:com.netumscan.printersdk.MediaRequirement:8513b835b607017f`

JVM descriptor：`I`

<a id="api-763fa86084104696fcd6"></a>

### public static final int ALLOW_LAST_SENT = 1

ALLOW_LAST_SENT常量，值1。

```java
public static final int ALLOW_LAST_SENT = 1;
```

允许当前协议最近成功发送的介质缓存，非设备读回。 不执行IO，不表示硬件已验证支持。

文档 ID：`android:member:com.netumscan.printersdk.MediaRequirement:5f044e082d8420a3`

JVM descriptor：`I`

<a id="api-947792eed5478e949e92"></a>

### public static final int REQUIRE_READBACK = 2

REQUIRE_READBACK常量，值2。

```java
public static final int REQUIRE_READBACK = 2;
```

要求设备读回，当前IO前1004拒绝。 不执行IO，不表示硬件已验证支持。

文档 ID：`android:member:com.netumscan.printersdk.MediaRequirement:f4ae445776867292`

JVM descriptor：`I`

<a id="api-e5824d9e8f16babb6dab"></a>

## MediaSettings

以微米表示的不可变介质参数。

```java
public final class com.netumscan.printersdk.MediaSettings
```

null字段表示未指定，不能当作硬件默认；presentFields按非null字段生成。使用时tracking必须存在，现有标准后端只支持CPCL tracking，其他字段在IO前拒绝。

文档 ID：`android:type:com.netumscan.printersdk.MediaSettings`

<a id="api-06350cf378ee9c0d4586"></a>

### public com.netumscan.printersdk.MediaSettings(com.netumscan.printersdk.MediaMode, java.lang.Long, java.lang.Long, java.lang.Long, java.lang.Long, java.lang.Integer, int)

保存显式微米介质参数。

```java
public com.netumscan.printersdk.MediaSettings(com.netumscan.printersdk.MediaMode, java.lang.Long, java.lang.Long, java.lang.Long, java.lang.Long, java.lang.Integer, int);
```

tracking及尺寸字段可null，存在位据此生成；非null长度须在uint32范围（负数/超过0xffffffff构造IllegalArgumentException），实际操作进一步要求正数和tracking匹配。持久化policy只接受0或1；标准后端只支持tracking，其他字段IO前1004拒绝。

文档 ID：`android:member:com.netumscan.printersdk.MediaSettings:e4681a628fcf986d`

JVM descriptor：`(Lcom/netumscan/printersdk/MediaMode;Ljava/lang/Long;Ljava/lang/Long;Ljava/lang/Long;Ljava/lang/Long;Ljava/lang/Integer;I)V`

<a id="api-ff2768b6972dcbc90dc5"></a>

### public final com.netumscan.printersdk.MediaMode tracking

生成仅含tracking的易失介质设置。

```java
public final com.netumscan.printersdk.MediaMode tracking;
```

mode不可null，其余字段未指定、policy=VOLATILE_ONLY。不执行IO；通过PrinterManager.setMedia提交。

文档 ID：`android:member:com.netumscan.printersdk.MediaSettings:29ec59d75927919b`

JVM descriptor：`Lcom/netumscan/printersdk/MediaMode;`

<a id="api-6ce20c57ddf2984df3c5"></a>

### public final int persistencePolicy

介质persistencePolicy字段。

```java
public final int persistencePolicy;
```

VOLATILE_ONLY=0默认仅易失设置，ALLOW_PERSISTENT=1表示允许持久化，不要求后端必须持久化。 null表示未指定，不是硬件报告；标准后端支持性在提交前校验。

文档 ID：`android:member:com.netumscan.printersdk.MediaSettings:3203af0199d0da68`

JVM descriptor：`I`

<a id="api-91c5a169dc65d41b9147"></a>

### public final int presentFields

介质presentFields字段。

```java
public final int presentFields;
```

TRACKING=1/GAP_LENGTH=2/LABEL_LENGTH=4/WIDTH=8/MARK_LENGTH=16/MARK_OFFSET=32，按非null字段生成。 null表示未指定，不是硬件报告；标准后端支持性在提交前校验。

文档 ID：`android:member:com.netumscan.printersdk.MediaSettings:794dc497376c264f`

JVM descriptor：`I`

<a id="api-5c070755010f3fda64e6"></a>

### public final java.lang.Integer markOffsetMicrometers

介质markOffsetMicrometers字段。

```java
public final java.lang.Integer markOffsetMicrometers;
```

可选有符号黑标偏移（微米），仅BLACK_MARK模式。 null表示未指定，不是硬件报告；标准后端支持性在提交前校验。

文档 ID：`android:member:com.netumscan.printersdk.MediaSettings:4a5c58d30f50366d`

JVM descriptor：`Ljava/lang/Integer;`

<a id="api-dada2dd1d134d006d048"></a>

### public final java.lang.Long gapLengthMicrometers

介质gapLengthMicrometers字段。

```java
public final java.lang.Long gapLengthMicrometers;
```

可选间隙（微米），仅GAP模式。 null表示未指定，不是硬件报告；标准后端支持性在提交前校验。

文档 ID：`android:member:com.netumscan.printersdk.MediaSettings:49aefeb794b34aaf`

JVM descriptor：`Ljava/lang/Long;`

<a id="api-63d5c4618a10caef79aa"></a>

### public final java.lang.Long labelLengthMicrometers

介质labelLengthMicrometers字段。

```java
public final java.lang.Long labelLengthMicrometers;
```

可选标签长度（微米）。 null表示未指定，不是硬件报告；标准后端支持性在提交前校验。

文档 ID：`android:member:com.netumscan.printersdk.MediaSettings:1ab71564f3d5b5a9`

JVM descriptor：`Ljava/lang/Long;`

<a id="api-b3363ec26b6c849ac06e"></a>

### public final java.lang.Long markLengthMicrometers

介质markLengthMicrometers字段。

```java
public final java.lang.Long markLengthMicrometers;
```

可选黑标长度（微米），仅BLACK_MARK模式。 null表示未指定，不是硬件报告；标准后端支持性在提交前校验。

文档 ID：`android:member:com.netumscan.printersdk.MediaSettings:469334b25042ab9e`

JVM descriptor：`Ljava/lang/Long;`

<a id="api-837ac1cf2b702376736a"></a>

### public final java.lang.Long widthMicrometers

介质widthMicrometers字段。

```java
public final java.lang.Long widthMicrometers;
```

可选宽度（微米）。 null表示未指定，不是硬件报告；标准后端支持性在提交前校验。

文档 ID：`android:member:com.netumscan.printersdk.MediaSettings:d98d979bb5fb0351`

JVM descriptor：`Ljava/lang/Long;`

<a id="api-eb04add88b1f2a07c336"></a>

### public static com.netumscan.printersdk.MediaSettings tracking(com.netumscan.printersdk.MediaMode)

生成仅含tracking的易失介质设置。

```java
public static com.netumscan.printersdk.MediaSettings tracking(com.netumscan.printersdk.MediaMode);
```

mode不可null，其余字段未指定、policy=VOLATILE_ONLY。不执行IO；通过PrinterManager.setMedia提交。

文档 ID：`android:member:com.netumscan.printersdk.MediaSettings:ec3e07d79765b01f`

JVM descriptor：`(Lcom/netumscan/printersdk/MediaMode;)Lcom/netumscan/printersdk/MediaSettings;`

<a id="api-99b4670819c188d481e0"></a>

### public static final int ALLOW_PERSISTENT = 1

ALLOW_PERSISTENT常量，值1。

```java
public static final int ALLOW_PERSISTENT = 1;
```

允许后端持久化，不要求一定持久化。 不执行IO，不表示硬件已验证支持。

文档 ID：`android:member:com.netumscan.printersdk.MediaSettings:70ffa1d5adb462ca`

JVM descriptor：`I`

<a id="api-cdd22e96de0b4001e166"></a>

### public static final int GAP_LENGTH = 2

GAP_LENGTH常量，值2。

```java
public static final int GAP_LENGTH = 2;
```

表示GAP_LENGTH对应的已知/存在字段mask；先检查known或present位再解释相关字段。 不执行IO，不表示硬件已验证支持。

文档 ID：`android:member:com.netumscan.printersdk.MediaSettings:baa2bf119fedcc5e`

JVM descriptor：`I`

<a id="api-e28057198450a2970dad"></a>

### public static final int LABEL_LENGTH = 4

LABEL_LENGTH常量，值4。

```java
public static final int LABEL_LENGTH = 4;
```

表示LABEL_LENGTH对应的已知/存在字段mask；先检查known或present位再解释相关字段。 不执行IO，不表示硬件已验证支持。

文档 ID：`android:member:com.netumscan.printersdk.MediaSettings:7fc0684ca801c37f`

JVM descriptor：`I`

<a id="api-e40c8ce4469837329fee"></a>

### public static final int MARK_LENGTH = 16

MARK_LENGTH常量，值16。

```java
public static final int MARK_LENGTH = 16;
```

表示MARK_LENGTH对应的已知/存在字段mask；先检查known或present位再解释相关字段。 不执行IO，不表示硬件已验证支持。

文档 ID：`android:member:com.netumscan.printersdk.MediaSettings:166a764d81b676c2`

JVM descriptor：`I`

<a id="api-91248227efd9d134953e"></a>

### public static final int MARK_OFFSET = 32

MARK_OFFSET常量，值32。

```java
public static final int MARK_OFFSET = 32;
```

表示MARK_OFFSET对应的已知/存在字段mask；先检查known或present位再解释相关字段。 不执行IO，不表示硬件已验证支持。

文档 ID：`android:member:com.netumscan.printersdk.MediaSettings:542e43778f1fbf84`

JVM descriptor：`I`

<a id="api-298798dd8bd76c5f8357"></a>

### public static final int TRACKING = 1

TRACKING常量，值1。

```java
public static final int TRACKING = 1;
```

表示TRACKING对应的已知/存在字段mask；先检查known或present位再解释相关字段。 不执行IO，不表示硬件已验证支持。

文档 ID：`android:member:com.netumscan.printersdk.MediaSettings:83415fc166517f4b`

JVM descriptor：`I`

<a id="api-cf997c833d8a6ef4cbd9"></a>

### public static final int VOLATILE_ONLY = 0

VOLATILE_ONLY常量，值0。

```java
public static final int VOLATILE_ONLY = 0;
```

仅允许易失设置。 不执行IO，不表示硬件已验证支持。

文档 ID：`android:member:com.netumscan.printersdk.MediaSettings:9275e9849a183175`

JVM descriptor：`I`

<a id="api-716625f2b58108b19de8"></a>

### public static final int WIDTH = 8

WIDTH常量，值8。

```java
public static final int WIDTH = 8;
```

表示WIDTH对应的已知/存在字段mask；先检查known或present位再解释相关字段。 不执行IO，不表示硬件已验证支持。

文档 ID：`android:member:com.netumscan.printersdk.MediaSettings:26874840ab7f4a0d`

JVM descriptor：`I`

<a id="api-a4466c8912ce3d2b0c5b"></a>

## PixelImage

不可变、紧密排列的像素快照。

```java
public final class com.netumscan.printersdk.PixelImage
```

支持GRAY8、RGB24和非预乘RGBA32。copyOf验证尺寸及长度并复制，首次复制应在后台；单图像素上限16MiB，不代表App或进程内存上限。

文档 ID：`android:type:com.netumscan.printersdk.PixelImage`

<a id="api-880557d2af108a9e558f"></a>

### public final int format

紧密像素排列格式。

```java
public final int format;
```

GRAY8每像素1字节、RGB24为R/G/B三字节、RGBA32为R/G/B/A四字节且alpha非预乘；无stride填充。

文档 ID：`android:member:com.netumscan.printersdk.PixelImage:7d80c06872b09b89`

JVM descriptor：`I`

<a id="api-a21c1f77293b49b5ad5c"></a>

### public final int heightPx

输入图片像素高度。

```java
public final int heightPx;
```

正整数，与宽度和格式共同决定精确字节数，不决定标签画布高度或走纸长度。

文档 ID：`android:member:com.netumscan.printersdk.PixelImage:27e856bf420d6cba`

JVM descriptor：`I`

<a id="api-59d8cb2f7888082929a1"></a>

### public final int widthPx

输入图片像素宽度。

```java
public final int widthPx;
```

正整数，描述像素排列；不是屏幕密度、毫米或打印头最大dots宽度。

文档 ID：`android:member:com.netumscan.printersdk.PixelImage:3abf20ef9564e527`

JVM descriptor：`I`

<a id="api-4c076bcf199b4dbfe3b4"></a>

### public static com.netumscan.printersdk.PixelImage copyOf(byte[], int, int, int)

验证并复制紧密排列的像素。

```java
public static com.netumscan.printersdk.PixelImage copyOf(byte[], int, int, int);
```

输入非空、宽高正数、format为三个声明值，长度必须精确匹配且不超过16MiB。非法纯值同步异常；复制后原数组可修改，大数组复制由App放后台。

文档 ID：`android:member:com.netumscan.printersdk.PixelImage:eb11a63aa9fb7697`

JVM descriptor：`([BIII)Lcom/netumscan/printersdk/PixelImage;`

编译器注解：`@androidx.annotation.NonNull`。

参数 0（从 0 起）注解：`@androidx.annotation.NonNull`。

<a id="api-dcb164f4894bc3efad16"></a>

### public static final int GRAY8 = 1

声明8位灰度像素格式。

```java
public static final int GRAY8 = 1;
```

数值1，每像素1字节，0为黑、255为白，行紧密排列。

文档 ID：`android:member:com.netumscan.printersdk.PixelImage:4909d4185765b94a`

JVM descriptor：`I`

<a id="api-10b7d98ba95d5ce32ff2"></a>

### public static final int RGB24 = 2

声明24位RGB像素格式。

```java
public static final int RGB24 = 2;
```

数值2，每像素R/G/B各1字节，SDK负责灰度及二值化，行紧密排列。

文档 ID：`android:member:com.netumscan.printersdk.PixelImage:72c70a2f4730535e`

JVM descriptor：`I`

<a id="api-1597525194c9d49d51bf"></a>

### public static final int RGBA32 = 3

声明32位非预乘RGBA像素格式。

```java
public static final int RGBA32 = 3;
```

数值3，每像素R/G/B/A各1字节；alpha按白底合成，不能传入预乘字节而未转换。

文档 ID：`android:member:com.netumscan.printersdk.PixelImage:5ff7bf192f2c5b09`

JVM descriptor：`I`

<a id="api-5cf4c76272c2df7a9a90"></a>

## PreparedPrint

保留的不可变纯数据打印描述。

```java
public final class com.netumscan.printersdk.PreparedPrint
```

描述布局和图片／空白元素，不持有native任务或图像句柄；当前Manager内部仍使用此值，但公开print只接受PrintOptions，不再接受Preparation。新调用使用PixelImage／ImageSource添加图片。

文档 ID：`android:type:com.netumscan.printersdk.PreparedPrint`

<a id="api-8617a994ccb9d516bea9"></a>

### public static com.netumscan.printersdk.PreparedPrint$Builder label(int, int, int)

创建标签布局的打印描述构建器。

```java
public static com.netumscan.printersdk.PreparedPrint$Builder label(int, int, int);
```

宽、高以dots计，dpi为明确分辨率；元素与原点等信息在build后固定，Manager负责后台native导入与校验。

文档 ID：`android:member:com.netumscan.printersdk.PreparedPrint:bb2ee17d7110b13e`

JVM descriptor：`(III)Lcom/netumscan/printersdk/PreparedPrint$Builder;`

<a id="api-e7b052b087d3bd055711"></a>

### public static com.netumscan.printersdk.PreparedPrint$Builder receipt(int, long, int)

创建小票布局的打印描述构建器。

```java
public static com.netumscan.printersdk.PreparedPrint$Builder receipt(int, long, int);
```

宽度与最大长度以dots计，并指定dpi；可追加图像或空白，最终由Manager后台构建native任务。

文档 ID：`android:member:com.netumscan.printersdk.PreparedPrint:fb69c31c3d616b0b`

JVM descriptor：`(IJI)Lcom/netumscan/printersdk/PreparedPrint$Builder;`

<a id="api-53c008e0301074d5af58"></a>

### public static final int BAYER_4X4 = 2

4×4 Bayer抖动。

```java
public static final int BAYER_4X4 = 2;
```

该常量的精确名称及编译值由公开基线固定；使用与本类型匹配的字段或参数，能力是否受支持以profile描述和请求校验为准。

文档 ID：`android:member:com.netumscan.printersdk.PreparedPrint:0dcb27f1cd32e35c`

JVM descriptor：`I`

<a id="api-771700c49c1997c40209"></a>

### public static final int BAYER_8X8 = 3

8×8 Bayer抖动。

```java
public static final int BAYER_8X8 = 3;
```

该常量的精确名称及编译值由公开基线固定；使用与本类型匹配的字段或参数，能力是否受支持以profile描述和请求校验为准。

文档 ID：`android:member:com.netumscan.printersdk.PreparedPrint:122522976173ccef`

JVM descriptor：`I`

<a id="api-9b2738ba3a169d64fd0a"></a>

### public static final int CUT_FULL = 1

全切纸。

```java
public static final int CUT_FULL = 1;
```

该常量的精确名称及编译值由公开基线固定；使用与本类型匹配的字段或参数，能力是否受支持以profile描述和请求校验为准。

文档 ID：`android:member:com.netumscan.printersdk.PreparedPrint:64956941107fd9c3`

JVM descriptor：`I`

<a id="api-841edbae5f025122122e"></a>

### public static final int CUT_NONE = 0

不切纸。

```java
public static final int CUT_NONE = 0;
```

该常量的精确名称及编译值由公开基线固定；使用与本类型匹配的字段或参数，能力是否受支持以profile描述和请求校验为准。

文档 ID：`android:member:com.netumscan.printersdk.PreparedPrint:a31548e7e8ce3a2e`

JVM descriptor：`I`

<a id="api-228f18e16e242eff52b8"></a>

### public static final int CUT_PARTIAL = 2

半切纸。

```java
public static final int CUT_PARTIAL = 2;
```

该常量的精确名称及编译值由公开基线固定；使用与本类型匹配的字段或参数，能力是否受支持以profile描述和请求校验为准。

文档 ID：`android:member:com.netumscan.printersdk.PreparedPrint:b6051533a14a0c38`

JVM descriptor：`I`

<a id="api-2ba47dc359bab6fc5b11"></a>

### public static final int EACH_COPY = 0

每一份应用尾部处理。

```java
public static final int EACH_COPY = 0;
```

该常量的精确名称及编译值由公开基线固定；使用与本类型匹配的字段或参数，能力是否受支持以profile描述和请求校验为准。

文档 ID：`android:member:com.netumscan.printersdk.PreparedPrint:447559c0d9447997`

JVM descriptor：`I`

<a id="api-740d8bc387a75bf68376"></a>

### public static final int END_OF_JOB = 1

整任务结束应用尾部处理。

```java
public static final int END_OF_JOB = 1;
```

该常量的精确名称及编译值由公开基线固定；使用与本类型匹配的字段或参数，能力是否受支持以profile描述和请求校验为准。

文档 ID：`android:member:com.netumscan.printersdk.PreparedPrint:0714561c652f45aa`

JVM descriptor：`I`

<a id="api-84aec6509d021e1eb22a"></a>

### public static final int FLOYD_STEINBERG = 1

Floyd-Steinberg抖动。

```java
public static final int FLOYD_STEINBERG = 1;
```

该常量的精确名称及编译值由公开基线固定；使用与本类型匹配的字段或参数，能力是否受支持以profile描述和请求校验为准。

文档 ID：`android:member:com.netumscan.printersdk.PreparedPrint:17319debb6248132`

JVM descriptor：`I`

<a id="api-cc8d78ec34350969517e"></a>

### public static final int GRAY8 = 1

一字节灰度像素格式。

```java
public static final int GRAY8 = 1;
```

该常量的精确名称及编译值由公开基线固定；使用与本类型匹配的字段或参数，能力是否受支持以profile描述和请求校验为准。

文档 ID：`android:member:com.netumscan.printersdk.PreparedPrint:4909d4185765b94a`

JVM descriptor：`I`

<a id="api-d63f0a63ad2b60c3d27b"></a>

### public static final int RGB24 = 2

三字节RGB像素格式。

```java
public static final int RGB24 = 2;
```

该常量的精确名称及编译值由公开基线固定；使用与本类型匹配的字段或参数，能力是否受支持以profile描述和请求校验为准。

文档 ID：`android:member:com.netumscan.printersdk.PreparedPrint:72c70a2f4730535e`

JVM descriptor：`I`

<a id="api-1e746648e751eb7ba9e0"></a>

### public static final int RGBA32 = 3

四字节RGBA像素格式。

```java
public static final int RGBA32 = 3;
```

该常量的精确名称及编译值由公开基线固定；使用与本类型匹配的字段或参数，能力是否受支持以profile描述和请求校验为准。

文档 ID：`android:member:com.netumscan.printersdk.PreparedPrint:5ff7bf192f2c5b09`

JVM descriptor：`I`

<a id="api-e66987c4630e845630d4"></a>

### public static final int THRESHOLD = 0

阈值二值化。

```java
public static final int THRESHOLD = 0;
```

该常量的精确名称及编译值由公开基线固定；使用与本类型匹配的字段或参数，能力是否受支持以profile描述和请求校验为准。

文档 ID：`android:member:com.netumscan.printersdk.PreparedPrint:885594684c18a57d`

JVM descriptor：`I`

<a id="api-9acaf968b2de3c0d2d3d"></a>

## PreparedPrint.Builder

构造旧纯数据描述的构建器。

```java
public final class com.netumscan.printersdk.PreparedPrint$Builder
```

设置布局和元素后build为不可变PreparedPrint，不执行设备IO；当前Manager不提供接收该值或Preparation的公开打印入口，新调用使用PrintOptions.Builder。

文档 ID：`android:type:com.netumscan.printersdk.PreparedPrint$Builder`

<a id="api-0ab9ead54f180df7f147"></a>

### public com.netumscan.printersdk.PreparedPrint build()

固定当前布局和元素并生成PreparedPrint。

```java
public com.netumscan.printersdk.PreparedPrint build();
```

复制构建器元素清单形成不可变描述；像素数据不因此重复导入native，后续Manager仍执行完整校验。

文档 ID：`android:member:com.netumscan.printersdk.PreparedPrint$Builder:d900c33e25f03f69`

JVM descriptor：`()Lcom/netumscan/printersdk/PreparedPrint;`

<a id="api-e0cee1e48780f69702c7"></a>

### public com.netumscan.printersdk.PreparedPrint$Builder addImage(com.netumscan.printersdk.PreparedPrint$Pixels, int, int)

在指定标签坐标添加像素元素。

```java
public com.netumscan.printersdk.PreparedPrint$Builder addImage(com.netumscan.printersdk.PreparedPrint$Pixels, int, int);
```

x、y以dots计；同一Pixels对象可被多元素引用，native导入阶段共享一次图像存储。

文档 ID：`android:member:com.netumscan.printersdk.PreparedPrint$Builder:de70c2c8e2a844e5`

JVM descriptor：`(Lcom/netumscan/printersdk/PreparedPrint$Pixels;II)Lcom/netumscan/printersdk/PreparedPrint$Builder;`

<a id="api-464f27b4f40daa22908e"></a>

### public com.netumscan.printersdk.PreparedPrint$Builder appendBlank(int)

向小票追加指定dots的空白。

```java
public com.netumscan.printersdk.PreparedPrint$Builder appendBlank(int);
```

只记录空白描述，参数与布局在构建/准入时校验；不会在调用构建器方法时发送设备命令。

文档 ID：`android:member:com.netumscan.printersdk.PreparedPrint$Builder:9d6ca812b38d2212`

JVM descriptor：`(I)Lcom/netumscan/printersdk/PreparedPrint$Builder;`

<a id="api-0a5b355fc9feadc82bed"></a>

### public com.netumscan.printersdk.PreparedPrint$Builder appendImage(com.netumscan.printersdk.PreparedPrint$Pixels)

向小票顺序追加像素元素。

```java
public com.netumscan.printersdk.PreparedPrint$Builder appendImage(com.netumscan.printersdk.PreparedPrint$Pixels);
```

仅用于小票构建器；相邻元素的顺序决定输出顺序，复用Pixels不会额外创建重复native图像。

文档 ID：`android:member:com.netumscan.printersdk.PreparedPrint$Builder:21f9b2f4c8e7c2bc`

JVM descriptor：`(Lcom/netumscan/printersdk/PreparedPrint$Pixels;)Lcom/netumscan/printersdk/PreparedPrint$Builder;`

<a id="api-254bdd08bd74b84bb90f"></a>

### public com.netumscan.printersdk.PreparedPrint$Builder copies(int)

设置整个打印描述的份数。

```java
public com.netumscan.printersdk.PreparedPrint$Builder copies(int);
```

使用有效正整数；份数与完成范围在native准入时校验，不通过应用循环自动重放失败任务。

文档 ID：`android:member:com.netumscan.printersdk.PreparedPrint$Builder:4f6f683c9d09277c`

JVM descriptor：`(I)Lcom/netumscan/printersdk/PreparedPrint$Builder;`

<a id="api-096948fcb267611e32b7"></a>

### public com.netumscan.printersdk.PreparedPrint$Builder mediaRequirement(com.netumscan.printersdk.MediaRequirement)

绑定打印任务要求的介质条件。

```java
public com.netumscan.printersdk.PreparedPrint$Builder mediaRequirement(com.netumscan.printersdk.MediaRequirement);
```

描述任务对介质设置的明确要求；该值不执行隐式型号探测或自动介质恢复。

文档 ID：`android:member:com.netumscan.printersdk.PreparedPrint$Builder:22e9666355086120`

JVM descriptor：`(Lcom/netumscan/printersdk/MediaRequirement;)Lcom/netumscan/printersdk/PreparedPrint$Builder;`

<a id="api-d5aaebec1aa584949f1b"></a>

### public com.netumscan.printersdk.PreparedPrint$Builder nextLabel(boolean)

设置标签打印后的下一张定位要求。

```java
public com.netumscan.printersdk.PreparedPrint$Builder nextLabel(boolean);
```

只记录本次标签请求的完成要求；最终是否支持由明确profile能力决定。

文档 ID：`android:member:com.netumscan.printersdk.PreparedPrint$Builder:27570e49e4b5f7b4`

JVM descriptor：`(Z)Lcom/netumscan/printersdk/PreparedPrint$Builder;`

<a id="api-c31db99a6dfeef6e0613"></a>

### public com.netumscan.printersdk.PreparedPrint$Builder origin(int, int)

设置标签原点偏移。

```java
public com.netumscan.printersdk.PreparedPrint$Builder origin(int, int);
```

x、y以dots计，仅适用于标签布局；不改变像素准备和协议编码的分层。

文档 ID：`android:member:com.netumscan.printersdk.PreparedPrint$Builder:e368678d47a99dce`

JVM descriptor：`(II)Lcom/netumscan/printersdk/PreparedPrint$Builder;`

<a id="api-1b6fe367072b80ab0442"></a>

### public com.netumscan.printersdk.PreparedPrint$Builder receiptFinish(int, int, int)

设置小票尾部空白、切纸方式和应用范围。

```java
public com.netumscan.printersdk.PreparedPrint$Builder receiptFinish(int, int, int);
```

尾部空白以dots计，cutMode和finishScope使用PreparedPrint常量；不支持切纸时由准入校验拒绝。

文档 ID：`android:member:com.netumscan.printersdk.PreparedPrint$Builder:b9150a9859c07c55`

JVM descriptor：`(III)Lcom/netumscan/printersdk/PreparedPrint$Builder;`

<a id="api-7a856229800daad8444a"></a>

### public com.netumscan.printersdk.PreparedPrint$Builder receiptMargins(int, int)

设置小票左右边距。

```java
public com.netumscan.printersdk.PreparedPrint$Builder receiptMargins(int, int);
```

leftDots、rightDots以dots计，仅对小票布局有效；不会通过缩放图像暗中改变纸张宽度。

文档 ID：`android:member:com.netumscan.printersdk.PreparedPrint$Builder:fd1f365df039aeb0`

JVM descriptor：`(II)Lcom/netumscan/printersdk/PreparedPrint$Builder;`

<a id="api-52e18138e529bfa1a76c"></a>

## PreparedPrint.Pixels

原始像素及图像处理参数。

```java
public final class com.netumscan.printersdk.PreparedPrint$Pixels
```

只描述GRAY8、RGB24或RGBA32原始像素；编码图片解码由应用准备函数负责。传入byte[]从准备提交到请求终态保持只读，同一Pixels对象重复使用时共享一次native导入。

文档 ID：`android:type:com.netumscan.printersdk.PreparedPrint$Pixels`

<a id="api-f769f13c96f24617f0db"></a>

### public com.netumscan.printersdk.PreparedPrint$Pixels(byte[], int, int, int, int, int, boolean, int)

构造PreparedPrint$Pixels只读数据。

```java
public com.netumscan.printersdk.PreparedPrint$Pixels(byte[], int, int, int, int, int, boolean, int);
```

只描述GRAY8、RGB24或RGBA32原始像素；编码图片解码由应用准备函数负责。传入byte[]从准备提交到请求终态保持只读，同一Pixels对象重复使用时共享一次native导入。

文档 ID：`android:member:com.netumscan.printersdk.PreparedPrint$Pixels:01112b49f4903a1f`

JVM descriptor：`([BIIIIIZI)V`

<a id="api-dbe5c3107692807ec377"></a>

### public final boolean invert

是否反转输出黑白。

```java
public final boolean invert;
```

只改变图像处理配置，不修改输入像素对象。

文档 ID：`android:member:com.netumscan.printersdk.PreparedPrint$Pixels:702f6f1660fce0d8`

JVM descriptor：`Z`

<a id="api-2b460e5c14f643e38872"></a>

### public final int algorithm

二值化算法。

```java
public final int algorithm;
```

选择阈值、Floyd-Steinberg或Bayer算法；图像处理独立于协议编码。

文档 ID：`android:member:com.netumscan.printersdk.PreparedPrint$Pixels:6d8363603cc9fb29`

JVM descriptor：`I`

<a id="api-e115c8f44ce3e289f679"></a>

### public final int format

原始像素格式。

```java
public final int format;
```

选择GRAY8、RGB24或RGBA32；输入数据长度与宽高须一致。

文档 ID：`android:member:com.netumscan.printersdk.PreparedPrint$Pixels:7d80c06872b09b89`

JVM descriptor：`I`

<a id="api-2f3b4ef48902ab86e34e"></a>

### public final int height

原图高度。

```java
public final int height;
```

单位为像素；大图按SDK图像/协议流水线分块处理，不要求完整最终打印缓冲区。

文档 ID：`android:member:com.netumscan.printersdk.PreparedPrint$Pixels:1e8b3775e799a9fa`

JVM descriptor：`I`

<a id="api-8f60b8f57e5328fe8179"></a>

### public final int targetWidthDots

缩放目标宽度。

```java
public final int targetWidthDots;
```

单位为打印dots；默认0表示不提供显式目标宽度，实际行为遵循图像适配配置。

文档 ID：`android:member:com.netumscan.printersdk.PreparedPrint$Pixels:65cdfcdcafb435c0`

JVM descriptor：`I`

<a id="api-11088a51f4bd6745b2b3"></a>

### public final int threshold

二值化阈值。

```java
public final int threshold;
```

有效范围按图像契约校验，默认128；不改变原始像素格式。

文档 ID：`android:member:com.netumscan.printersdk.PreparedPrint$Pixels:b0f72a281f2c8781`

JVM descriptor：`I`

<a id="api-f628ffc14780e5016ad7"></a>

### public final int width

原图宽度。

```java
public final int width;
```

单位为像素，与height和format共同确定输入数据布局；不等同于传输分块大小。

文档 ID：`android:member:com.netumscan.printersdk.PreparedPrint$Pixels:5727cdfd5221243c`

JVM descriptor：`I`

<a id="api-d5fd81bb07f52ad8e40f"></a>

### public static com.netumscan.printersdk.PreparedPrint$Pixels gray8(byte[], int, int)

用默认阈值设置构造GRAY8像素描述。

```java
public static com.netumscan.printersdk.PreparedPrint$Pixels gray8(byte[], int, int);
```

传入一字节灰度像素及宽高，默认threshold=128且不反色、不强制目标宽度；byte[]在请求完成前保持只读。

文档 ID：`android:member:com.netumscan.printersdk.PreparedPrint$Pixels:46d6216c01ea5816`

JVM descriptor：`([BII)Lcom/netumscan/printersdk/PreparedPrint$Pixels;`

<a id="api-4dc5d455340db9937535"></a>

## PrintOptions

一次完整结构化打印的不可变输入。

```java
public final class com.netumscan.printersdk.PrintOptions
```

合并来源、布局、份数、总超时、介质前置要求及软件预算。build不解码或连接，提交时固定有效Profile及来源引用。动态来源内容不因Options不可变而自动固定。

文档 ID：`android:type:com.netumscan.printersdk.PrintOptions`

<a id="api-107c99c34589685041cd"></a>

### public com.netumscan.printersdk.PrintOptions(int)

创建仅包含超时的旧纯值对象。

```java
public com.netumscan.printersdk.PrintOptions(int);
```

保留类型构造兼容，但缺少打印内容，传入print会通过Callback拒绝1001；旧显式Profile和Preparation打印入口已移除。完整打印须使用labelDots、labelMm或receiptDots构建器，自定义加载使用ImageSource.deferred。

文档 ID：`android:member:com.netumscan.printersdk.PrintOptions:e86d26104a321d47`

JVM descriptor：`(I)V`

<a id="api-4c7c59da4716bbbc5f26"></a>

### public final int timeoutMs

请求总超时配置。

```java
public final int timeoutMs;
```

超时预算覆盖等待、应用准备、native准入和执行阶段；timeoutMs/timeoutMilliseconds使用毫秒，必须为有效正值。

文档 ID：`android:member:com.netumscan.printersdk.PrintOptions:ccf22bfbf1a01308`

JVM descriptor：`I`

<a id="api-872ce7cdd19f884662bc"></a>

### public static com.netumscan.printersdk.PrintOptions$Builder labelDots(int, int)

创建以dots定义的标签构建器。

```java
public static com.netumscan.printersdk.PrintOptions$Builder labelDots(int, int);
```

宽高正数，实际范围按提交时有效Profile与软件页高检查；画布包括图片间和末尾空白，不自动裁剪。

文档 ID：`android:member:com.netumscan.printersdk.PrintOptions:924e3e2bca5ac94e`

JVM descriptor：`(II)Lcom/netumscan/printersdk/PrintOptions$Builder;`

编译器注解：`@androidx.annotation.NonNull`。

<a id="api-2451f630191b5f9d4e4d"></a>

### public static com.netumscan.printersdk.PrintOptions$Builder labelMm(double, double)

创建以毫米定义的标签构建器。

```java
public static com.netumscan.printersdk.PrintOptions$Builder labelMm(double, double);
```

宽高须正有限数，提交时按固定Profile的dotsPerMm进行round(mm*dotsPerMm)，拒绝归零、溢出与设备范围越界。LT-M90的50×50mm为400×400dots。其他Dots入口仍以dots计，不进行单位改写。

文档 ID：`android:member:com.netumscan.printersdk.PrintOptions:2e53e0a4761cfa3f`

JVM descriptor：`(DD)Lcom/netumscan/printersdk/PrintOptions$Builder;`

编译器注解：`@androidx.annotation.NonNull`。

<a id="api-f86fad5c4d0478528a48"></a>

### public static com.netumscan.printersdk.PrintOptions$Builder receiptDots(int, long)

创建小票顺序内容构建器。

```java
public static com.netumscan.printersdk.PrintOptions$Builder receiptDots(int, long);
```

宽度及整作业最大总长度须正数，整单预算包含所有图片、空白段、末尾空白及全部份数；只匹配ESC/POS配置。

文档 ID：`android:member:com.netumscan.printersdk.PrintOptions:94e0b2a7bbd57d58`

JVM descriptor：`(IJ)Lcom/netumscan/printersdk/PrintOptions$Builder;`

编译器注解：`@androidx.annotation.NonNull`。

<a id="api-feb2999a7f7069358719"></a>

### public static final int CUT_FULL = 1

声明全切纸策略。

```java
public static final int CUT_FULL = 1;
```

数值1，当前结构化切纸未实现，提交明确拒绝。

文档 ID：`android:member:com.netumscan.printersdk.PrintOptions:64956941107fd9c3`

JVM descriptor：`I`

<a id="api-0d8f2d56dd0e0a4a013f"></a>

### public static final int CUT_NONE = 0

不切纸。

```java
public static final int CUT_NONE = 0;
```

数值0，当前支持的收尾默认值。

文档 ID：`android:member:com.netumscan.printersdk.PrintOptions:a31548e7e8ce3a2e`

JVM descriptor：`I`

<a id="api-1a59aa1723b7d5e82941"></a>

### public static final int CUT_PARTIAL = 2

声明半切纸策略。

```java
public static final int CUT_PARTIAL = 2;
```

数值2，当前结构化切纸未实现，提交明确拒绝。

文档 ID：`android:member:com.netumscan.printersdk.PrintOptions:b6051533a14a0c38`

JVM descriptor：`I`

<a id="api-920085226b850cef094a"></a>

### public static final int EACH_COPY = 0

每份执行收尾。

```java
public static final int EACH_COPY = 0;
```

数值0，默认作用域，尾白计入每份和整单预算。

文档 ID：`android:member:com.netumscan.printersdk.PrintOptions:447559c0d9447997`

JVM descriptor：`I`

<a id="api-2e0d4e3eeebe29dca170"></a>

### public static final int END_OF_JOB = 1

声明整单末尾执行收尾。

```java
public static final int END_OF_JOB = 1;
```

数值1，当前未实现，不能静默降级为每份。

文档 ID：`android:member:com.netumscan.printersdk.PrintOptions:0714561c652f45aa`

JVM descriptor：`I`

<a id="api-0bdf506a98412f3151e5"></a>

## PrintOptions.Builder

标签或小票的打印输入构建器。

```java
public final class com.netumscan.printersdk.PrintOptions$Builder
```

标签位置和小票顺序追加各有明确语义；最多1024个元素。构建校验纯值，设备范围、图源与不支持功能通过请求回调拒绝；build后输入结构固定。

文档 ID：`android:type:com.netumscan.printersdk.PrintOptions$Builder`

<a id="api-66b3e3ad75b207fe8ece"></a>

### public com.netumscan.printersdk.PrintOptions build()

固定作业结构与参数。

```java
public com.netumscan.printersdk.PrintOptions build();
```

不连接、不解码，不保留读取租约。来源在每次提交时统一保留；来源关闭后不能新提交，同源动态内容不因build而固定。

文档 ID：`android:member:com.netumscan.printersdk.PrintOptions$Builder:73a8d7ad4823121c`

JVM descriptor：`()Lcom/netumscan/printersdk/PrintOptions;`

编译器注解：`@androidx.annotation.NonNull`。

<a id="api-dd519c6b08330541b70e"></a>

### public com.netumscan.printersdk.PrintOptions$Builder addImage(com.netumscan.printersdk.ImageSource, int, int)

向标签添加来源与dots位置。

```java
public com.netumscan.printersdk.PrintOptions$Builder addImage(com.netumscan.printersdk.ImageSource, int, int);
```

默认阈值128，或明确提供ImageRender。x/y非负；当前CPCL只支持x=0、纵向不重叠，不承诺任意二维布局。build不读图，提交全成或全败保留图源。

文档 ID：`android:member:com.netumscan.printersdk.PrintOptions$Builder:f9022c234cf900bd`

JVM descriptor：`(Lcom/netumscan/printersdk/ImageSource;II)Lcom/netumscan/printersdk/PrintOptions$Builder;`

编译器注解：`@androidx.annotation.NonNull`。

参数 0（从 0 起）注解：`@androidx.annotation.NonNull`。

<a id="api-12f94cbe67ffdc6ca25a"></a>

### public com.netumscan.printersdk.PrintOptions$Builder addImage(com.netumscan.printersdk.ImageSource, int, int, com.netumscan.printersdk.ImageRender)

向标签添加来源与dots位置。

```java
public com.netumscan.printersdk.PrintOptions$Builder addImage(com.netumscan.printersdk.ImageSource, int, int, com.netumscan.printersdk.ImageRender);
```

默认阈值128，或明确提供ImageRender。x/y非负；当前CPCL只支持x=0、纵向不重叠，不承诺任意二维布局。build不读图，提交全成或全败保留图源。

文档 ID：`android:member:com.netumscan.printersdk.PrintOptions$Builder:2ddb172a96c5971e`

JVM descriptor：`(Lcom/netumscan/printersdk/ImageSource;IILcom/netumscan/printersdk/ImageRender;)Lcom/netumscan/printersdk/PrintOptions$Builder;`

编译器注解：`@androidx.annotation.NonNull`。

参数 0（从 0 起）注解：`@androidx.annotation.NonNull`。

参数 3（从 0 起）注解：`@androidx.annotation.NonNull`。

<a id="api-5eb04c0971ffe9c0a8e5"></a>

### public com.netumscan.printersdk.PrintOptions$Builder appendBlankDots(int)

在小票中追加空白走纸段。

```java
public com.netumscan.printersdk.PrintOptions$Builder appendBlankDots(int);
```

长度非负、单位dots，计入每份及整单长度；不会绕过设备队列发送即时走纸。

文档 ID：`android:member:com.netumscan.printersdk.PrintOptions$Builder:1fc07ea9c64f93fd`

JVM descriptor：`(I)Lcom/netumscan/printersdk/PrintOptions$Builder;`

编译器注解：`@androidx.annotation.NonNull`。

<a id="api-2ffcceb3e737e09f5e30"></a>

### public com.netumscan.printersdk.PrintOptions$Builder appendImage(com.netumscan.printersdk.ImageSource)

在小票末尾追加来源。

```java
public com.netumscan.printersdk.PrintOptions$Builder appendImage(com.netumscan.printersdk.ImageSource);
```

仅用于receipt，默认阈值128或提供逐图处理；按添加顺序输出，与空白段共用同一作业和FIFO。

文档 ID：`android:member:com.netumscan.printersdk.PrintOptions$Builder:24685e4f066e1624`

JVM descriptor：`(Lcom/netumscan/printersdk/ImageSource;)Lcom/netumscan/printersdk/PrintOptions$Builder;`

编译器注解：`@androidx.annotation.NonNull`。

参数 0（从 0 起）注解：`@androidx.annotation.NonNull`。

<a id="api-72fc9b6ac30ef17fa905"></a>

### public com.netumscan.printersdk.PrintOptions$Builder appendImage(com.netumscan.printersdk.ImageSource, com.netumscan.printersdk.ImageRender)

在小票末尾追加来源。

```java
public com.netumscan.printersdk.PrintOptions$Builder appendImage(com.netumscan.printersdk.ImageSource, com.netumscan.printersdk.ImageRender);
```

仅用于receipt，默认阈值128或提供逐图处理；按添加顺序输出，与空白段共用同一作业和FIFO。

文档 ID：`android:member:com.netumscan.printersdk.PrintOptions$Builder:52c6ae08a3aa3165`

JVM descriptor：`(Lcom/netumscan/printersdk/ImageSource;Lcom/netumscan/printersdk/ImageRender;)Lcom/netumscan/printersdk/PrintOptions$Builder;`

编译器注解：`@androidx.annotation.NonNull`。

参数 0（从 0 起）注解：`@androidx.annotation.NonNull`。

参数 1（从 0 起）注解：`@androidx.annotation.NonNull`。

<a id="api-012c21df5720878840a8"></a>

### public com.netumscan.printersdk.PrintOptions$Builder copies(int)

设置本次打印份数。

```java
public com.netumscan.printersdk.PrintOptions$Builder copies(int);
```

范围1..100，默认1；超限为纯值参数错误。小票整单总长需包含所有份数，不把份数写入设备Profile。

文档 ID：`android:member:com.netumscan.printersdk.PrintOptions$Builder:b89bcedae0be7061`

JVM descriptor：`(I)Lcom/netumscan/printersdk/PrintOptions$Builder;`

编译器注解：`@androidx.annotation.NonNull`。

<a id="api-982187fa4ffa7daf95c2"></a>

### public com.netumscan.printersdk.PrintOptions$Builder horizontalCalibrationDots(int)

设置本次标签的有符号横向校准。

```java
public com.netumscan.printersdk.PrintOptions$Builder horizontalCalibrationDots(int);
```

单位dots，默认0，正向右、负向左；只适用标签。提交后由固定Profile计算设备起点，不改变Profile或图像在标签内的位置。负起点、右侧越界和溢出在加载／IO前通过Callback拒绝1001；不得与非零绝对originX混用。

文档 ID：`android:member:com.netumscan.printersdk.PrintOptions$Builder:3bdec68192c627ec`

JVM descriptor：`(I)Lcom/netumscan/printersdk/PrintOptions$Builder;`

编译器注解：`@androidx.annotation.NonNull`。

<a id="api-6d847f4fe347d6a56825"></a>

### public com.netumscan.printersdk.PrintOptions$Builder maxPageHeightDots(int)

设置软件页高保护预算。

```java
public com.netumscan.printersdk.PrintOptions$Builder maxPageHeightDots(int);
```

默认65535，须1..65535，不代表设备已确认的硬件最大长度；与小票全部份数的总长预算区分。

文档 ID：`android:member:com.netumscan.printersdk.PrintOptions$Builder:c02892e5073fc9cb`

JVM descriptor：`(I)Lcom/netumscan/printersdk/PrintOptions$Builder;`

编译器注解：`@androidx.annotation.NonNull`。

<a id="api-bd8b8bd10d61726a9eea"></a>

### public com.netumscan.printersdk.PrintOptions$Builder mediaRequirement(com.netumscan.printersdk.MediaRequirement)

声明发送前的介质证据要求。

```java
public com.netumscan.printersdk.PrintOptions$Builder mediaRequirement(com.netumscan.printersdk.MediaRequirement);
```

有效证据绑定内部连接和相容配置，执行时再校验；ALLOW_LAST_SENT只代表已发送设置，不能当作硬件回读确认。

文档 ID：`android:member:com.netumscan.printersdk.PrintOptions$Builder:e28a076ad6397786`

JVM descriptor：`(Lcom/netumscan/printersdk/MediaRequirement;)Lcom/netumscan/printersdk/PrintOptions$Builder;`

编译器注解：`@androidx.annotation.NonNull`。

参数 0（从 0 起）注解：`@androidx.annotation.NonNull`。

<a id="api-0675e9fdd5fc15f218d7"></a>

### public com.netumscan.printersdk.PrintOptions$Builder nextLabel(boolean)

设置本次标签末尾FORM行为。

```java
public com.netumscan.printersdk.PrintOptions$Builder nextLabel(boolean);
```

默认false，属于作业而非型号配置；仅label可用，连续纸应由业务明确关闭，不触发隐式校准。

文档 ID：`android:member:com.netumscan.printersdk.PrintOptions$Builder:2222174cacc9c2cf`

JVM descriptor：`(Z)Lcom/netumscan/printersdk/PrintOptions$Builder;`

编译器注解：`@androidx.annotation.NonNull`。

<a id="api-cc758e4472f03f8ef5dc"></a>

### public com.netumscan.printersdk.PrintOptions$Builder originDots(int, int)

设置兼容的标签绝对原点。

```java
public com.netumscan.printersdk.PrintOptions$Builder originDots(int, int);
```

x／y非负，默认0／0。非零x仅适用于LEFT Profile，且不能与非零horizontalCalibrationDots混用；CENTERED时x须为0，由SDK算横向起点，y仍保留。配置相关冲突在加载和IO前通过Callback拒绝1001，不静默忽略。

文档 ID：`android:member:com.netumscan.printersdk.PrintOptions$Builder:9199b6afdfe08951`

JVM descriptor：`(II)Lcom/netumscan/printersdk/PrintOptions$Builder;`

编译器注解：`@androidx.annotation.NonNull`。

<a id="api-fb433394c97a4d9cbebd"></a>

### public com.netumscan.printersdk.PrintOptions$Builder receiptFinish(int, int, int)

设置每份小票尾白及收尾。

```java
public com.netumscan.printersdk.PrintOptions$Builder receiptFinish(int, int, int);
```

尾白非负、默认0，切纸默认CUT_NONE、作用域EACH_COPY；其他已声明但未实现组合通过请求回调明确拒绝，不静默忽略。

文档 ID：`android:member:com.netumscan.printersdk.PrintOptions$Builder:ceae859b5c996629`

JVM descriptor：`(III)Lcom/netumscan/printersdk/PrintOptions$Builder;`

编译器注解：`@androidx.annotation.NonNull`。

<a id="api-878d517a04047257b2a5"></a>

### public com.netumscan.printersdk.PrintOptions$Builder receiptMargins(int, int)

声明小票左右边距。

```java
public com.netumscan.printersdk.PrintOptions$Builder receiptMargins(int, int);
```

两边非负，当前只有0/0受支持；非零值后续明确UNSUPPORTED_OPERATION，不以裁剪方式偷偷实现。

文档 ID：`android:member:com.netumscan.printersdk.PrintOptions$Builder:384ae91fd90b5239`

JVM descriptor：`(II)Lcom/netumscan/printersdk/PrintOptions$Builder;`

编译器注解：`@androidx.annotation.NonNull`。

<a id="api-501b467b087674dde30a"></a>

### public com.netumscan.printersdk.PrintOptions$Builder timeoutMs(int)

设置提交后的总执行预算。

```java
public com.netumscan.printersdk.PrintOptions$Builder timeoutMs(int);
```

必须大于0，默认120000ms，包含排队、图片加载／处理和发送，不包括提交前系统选图或独立导入。

文档 ID：`android:member:com.netumscan.printersdk.PrintOptions$Builder:9e27f03b28f154ce`

JVM descriptor：`(I)Lcom/netumscan/printersdk/PrintOptions$Builder;`

编译器注解：`@androidx.annotation.NonNull`。

<a id="api-a36171733f6c01585161"></a>

## PrinterCapabilities

聚合本地能力、约束与几何观察。

```java
public final class com.netumscan.printersdk.PrinterCapabilities
```

八项feature、四项constraint和geometry分别保留known/deviceSupport/evidence。不执行IO、不预留接纳额度；纯连接设备几何未知，不能由最近Profile或零数值推断硬件支持。

文档 ID：`android:type:com.netumscan.printersdk.PrinterCapabilities`

<a id="api-99761482a4773c008c52"></a>

### public com.netumscan.printersdk.FeatureSupport feature(int)

按ID取一项功能能力。

```java
public com.netumscan.printersdk.FeatureSupport feature(int);
```

ID1..8对应PrinterManager功能常量；其他ID返回unknown/未实现值，不执行IO。实现、deviceSupport与evidence独立，检查knownFields后才能解释集合mask。

文档 ID：`android:member:com.netumscan.printersdk.PrinterCapabilities:02554ca7e27ea317`

JVM descriptor：`(I)Lcom/netumscan/printersdk/FeatureSupport;`

<a id="api-e4cdb110786ced1f5c22"></a>

### public com.netumscan.printersdk.ScalarConstraint constraint(int)

按ID取一项范围约束。

```java
public com.netumscan.printersdk.ScalarConstraint constraint(int);
```

ID1..4对应PrinterConstraint参数；其他ID返回known=false，不执行IO。copies及源载荷额度是SDK策略，未知介质范围不是无限。

文档 ID：`android:member:com.netumscan.printersdk.PrinterCapabilities:a258c0e7ea03dc6f`

JVM descriptor：`(I)Lcom/netumscan/printersdk/ScalarConstraint;`

<a id="api-1144e63ac4d0fc5c6a33"></a>

### public final int dpiX

打印几何dpiX观察值。

```java
public final int dpiX;
```

横向DPI，须geometryPresentFields的DPI位1有效。 当前纯通信设备无几何证据，必须检查存在位，不能从数值零推断正常或支持。

文档 ID：`android:member:com.netumscan.printersdk.PrinterCapabilities:5253b3056074e745`

JVM descriptor：`I`

<a id="api-442b043e5d7a36e6f759"></a>

### public final int dpiY

打印几何dpiY观察值。

```java
public final int dpiY;
```

纵向DPI，须DPI位1有效。 当前纯通信设备无几何证据，必须检查存在位，不能从数值零推断正常或支持。

文档 ID：`android:member:com.netumscan.printersdk.PrinterCapabilities:414f3289a65cc726`

JVM descriptor：`I`

<a id="api-b2acb3e6aacd017bcd27"></a>

### public final int geometryEvidence

打印几何geometryEvidence观察值。

```java
public final int geometryEvidence;
```

PROFILE_DECLARED=1、DEVICE_REPORTED=2、HARDWARE_VERIFIED=4；零表示无证据。 当前纯通信设备无几何证据，必须检查存在位，不能从数值零推断正常或支持。

文档 ID：`android:member:com.netumscan.printersdk.PrinterCapabilities:913c74adc3bf7c03`

JVM descriptor：`I`

<a id="api-6ef929d87e42388914f9"></a>

### public final int geometryPresentFields

打印几何geometryPresentFields观察值。

```java
public final int geometryPresentFields;
```

DPI=1、WIDTH=2、LENGTH=4、CUT_BLANK=8；未知字段数值零不能解释为已知零或无限。 当前纯通信设备无几何证据，必须检查存在位，不能从数值零推断正常或支持。

文档 ID：`android:member:com.netumscan.printersdk.PrinterCapabilities:cf4ac59529e54952`

JVM descriptor：`I`

<a id="api-cf25b3fad6e01659f4c4"></a>

### public final int maxPageLengthDots

打印几何maxPageLengthDots观察值。

```java
public final int maxPageLengthDots;
```

一页/一份最大长度（点），须LENGTH位4有效，不乘copies，不是小票整单预算。 当前纯通信设备无几何证据，必须检查存在位，不能从数值零推断正常或支持。

文档 ID：`android:member:com.netumscan.printersdk.PrinterCapabilities:d4fa201acc001d96`

JVM descriptor：`I`

<a id="api-0bc443d11dcb982b726a"></a>

### public final int minCutBlankDots

打印几何minCutBlankDots观察值。

```java
public final int minCutBlankDots;
```

切纸前最小留白（点），须CUT_BLANK位8有效。 当前纯通信设备无几何证据，必须检查存在位，不能从数值零推断正常或支持。

文档 ID：`android:member:com.netumscan.printersdk.PrinterCapabilities:fd704893c25f2b1c`

JVM descriptor：`I`

<a id="api-5cd56ea0b31eb4942545"></a>

### public final int printableWidthDots

打印几何printableWidthDots观察值。

```java
public final int printableWidthDots;
```

可打印宽度（点），须WIDTH位2有效。 当前纯通信设备无几何证据，必须检查存在位，不能从数值零推断正常或支持。

文档 ID：`android:member:com.netumscan.printersdk.PrinterCapabilities:338e757da58637fb`

JVM descriptor：`I`

<a id="api-bfc786be5b911ef74d19"></a>

## PrinterManager

Android打印机运行时唯一控制入口。

```java
public final class com.netumscan.printersdk.PrinterManager
```

每实例同时控制一台设备，连接保存可选默认Profile；PrintOptions提交固定配置快照，所有设备IO仍走唯一FIFO。每请求独立Callback主线程异步一次，不公开连接身份、Task或原生控制句柄。

文档 ID：`android:type:com.netumscan.printersdk.PrinterManager`

编译器注解：`@androidx.annotation.AnyThread`。

<a id="api-6761fb538c137c164625"></a>

### public com.netumscan.printersdk.PrinterManager(android.content.Context)

创建独立Manager实例。

```java
public com.netumscan.printersdk.PrinterManager(android.content.Context);
```

每个实例至多管理一个连接；不使用共享单例。应用按页面或业务所有权保存实例，并在结束时调用异步close释放资源。

文档 ID：`android:member:com.netumscan.printersdk.PrinterManager:989371a8a2f8fb1d`

JVM descriptor：`(Landroid/content/Context;)V`

参数 0（从 0 起）注解：`@androidx.annotation.NonNull`。

<a id="api-67aaa2c77fa3ef876418"></a>

### public com.netumscan.printersdk.RequestId bondedDevices(com.netumscan.printersdk.PrinterManager$Callback<java.util.List<com.netumscan.printersdk.DiscoveredDevice>>)

枚举系统已配对的蓝牙设备。

```java
public com.netumscan.printersdk.RequestId bondedDevices(com.netumscan.printersdk.PrinterManager$Callback<java.util.List<com.netumscan.printersdk.DiscoveredDevice>>);
```

通过 Manager 异步返回系统已配对蓝牙设备的只读列表。底层读取 `getBondedDevices()`，不按经典蓝牙或 BLE 类型过滤；出现在列表中不保证支持 SPP/RFCOMM，也不表示已连接或识别为打印机。不发起隐式配对，权限和生命周期错误通过本次完成回调返回。

文档 ID：`android:member:com.netumscan.printersdk.PrinterManager:250ef332e85fdb96`

JVM descriptor：`(Lcom/netumscan/printersdk/PrinterManager$Callback;)Lcom/netumscan/printersdk/RequestId;`

编译器注解：`@androidx.annotation.NonNull`。

参数 0（从 0 起）注解：`@androidx.annotation.NonNull`。

<a id="api-aa164db0d48e84572c33"></a>

### public com.netumscan.printersdk.RequestId capabilities(com.netumscan.printersdk.PrinterManager$Callback<com.netumscan.printersdk.PrinterCapabilities>)

读取当前设备的能力描述。

```java
public com.netumscan.printersdk.RequestId capabilities(com.netumscan.printersdk.PrinterManager$Callback<com.netumscan.printersdk.PrinterCapabilities>);
```

异步交付PrinterCapabilities；未知或不支持的能力应按描述处理，能力查询不执行自动打印探测，也不替代每次请求的准入校验。

文档 ID：`android:member:com.netumscan.printersdk.PrinterManager:3b59a91b1fcc68e8`

JVM descriptor：`(Lcom/netumscan/printersdk/PrinterManager$Callback;)Lcom/netumscan/printersdk/RequestId;`

编译器注解：`@androidx.annotation.NonNull`。

参数 0（从 0 起）注解：`@androidx.annotation.NonNull`。

<a id="api-704c0cf7d94eeebbfb7f"></a>

### public com.netumscan.printersdk.RequestId close(com.netumscan.printersdk.PrinterManager$Callback<java.lang.Void>)

永久关闭Manager并释放其连接资源。

```java
public com.netumscan.printersdk.RequestId close(com.netumscan.printersdk.PrinterManager$Callback<java.lang.Void>);
```

这是幂等的异步清理屏障；所有属于该会话的请求终态先交付，清理完成后再交付close回调。关闭后业务请求异步返回InvalidState。

文档 ID：`android:member:com.netumscan.printersdk.PrinterManager:9ffe7f34b33337a5`

JVM descriptor：`(Lcom/netumscan/printersdk/PrinterManager$Callback;)Lcom/netumscan/printersdk/RequestId;`

编译器注解：`@androidx.annotation.NonNull`。

参数 0（从 0 起）注解：`@androidx.annotation.NonNull`。

<a id="api-c5d3af6de82b1dcb267e"></a>

### public com.netumscan.printersdk.RequestId configurePrintProfile(com.netumscan.printersdk.PrinterProfile, com.netumscan.printersdk.PrinterManager$Callback<com.netumscan.printersdk.ConnectionInfo>)

连接后空闲时原子补充或更正打印Profile。

```java
public com.netumscan.printersdk.RequestId configurePrintProfile(com.netumscan.printersdk.PrinterProfile, com.netumscan.printersdk.PrinterManager$Callback<com.netumscan.printersdk.ConnectionInfo>);
```

未知配置和已匹配配置均可手动配置。未连接、清理中或设备业务未结束时1002，保留旧配置和介质证据。成功只更新当前连接本地值，不发送IO，并使旧介质证据保守失效；CPCL需明确setMedia重新初始化。断开后不保留，下次连接重新匹配或手动配置。

文档 ID：`android:member:com.netumscan.printersdk.PrinterManager:9b70aeccad0b00c1`

JVM descriptor：`(Lcom/netumscan/printersdk/PrinterProfile;Lcom/netumscan/printersdk/PrinterManager$Callback;)Lcom/netumscan/printersdk/RequestId;`

编译器注解：`@androidx.annotation.NonNull`。

参数 0（从 0 起）注解：`@androidx.annotation.NonNull`。

参数 1（从 0 起）注解：`@androidx.annotation.NonNull`。

<a id="api-ecc3f72b49b051f4b751"></a>

### public com.netumscan.printersdk.RequestId connect(com.netumscan.printersdk.ConnectionOptions, int, com.netumscan.printersdk.PrinterManager$Callback<com.netumscan.printersdk.ConnectionInfo>, com.netumscan.printersdk.PrinterManager$DisconnectedListener)

异步连接并返回采用的ConnectionInfo。

```java
public com.netumscan.printersdk.RequestId connect(com.netumscan.printersdk.ConnectionOptions, int, com.netumscan.printersdk.PrinterManager$Callback<com.netumscan.printersdk.ConnectionInfo>, com.netumscan.printersdk.PrinterManager$DisconnectedListener);
```

每请求一次主线程结果，意外断连监听独立绑定该次连接且清理后通知。connect不替换已有设备；失败或取消后需要重连时先等待disconnect屏障。泛型由Void迁至ConnectionInfo，旧源码须适配，不依靠类型擦除重载。

文档 ID：`android:member:com.netumscan.printersdk.PrinterManager:c4de8a3c83f81cbe`

JVM descriptor：`(Lcom/netumscan/printersdk/ConnectionOptions;ILcom/netumscan/printersdk/PrinterManager$Callback;Lcom/netumscan/printersdk/PrinterManager$DisconnectedListener;)Lcom/netumscan/printersdk/RequestId;`

编译器注解：`@androidx.annotation.NonNull`。

参数 0（从 0 起）注解：`@androidx.annotation.NonNull`。

参数 2（从 0 起）注解：`@androidx.annotation.NonNull`。

参数 3（从 0 起）注解：`@androidx.annotation.Nullable`。

<a id="api-eace0338b53324c15ab8"></a>

### public com.netumscan.printersdk.RequestId disconnect(com.netumscan.printersdk.PrinterManager$Callback<java.lang.Void>)

关闭当前连接并结束本会话请求。

```java
public com.netumscan.printersdk.RequestId disconnect(com.netumscan.printersdk.PrinterManager$Callback<java.lang.Void>);
```

尽快取消等待、准备和native任务并异步释放资源；清理完成后才可显式连接新设备。不合作的准备代码可以迟到返回，但结果不会再次准入。

文档 ID：`android:member:com.netumscan.printersdk.PrinterManager:9b3300e7a5bc65cd`

JVM descriptor：`(Lcom/netumscan/printersdk/PrinterManager$Callback;)Lcom/netumscan/printersdk/RequestId;`

编译器注解：`@androidx.annotation.NonNull`。

参数 0（从 0 起）注解：`@androidx.annotation.NonNull`。

<a id="api-edbea2a41134aeab2c23"></a>

### public com.netumscan.printersdk.RequestId locateNextLabel(int, com.netumscan.printersdk.PrinterManager$Callback<java.lang.Void>)

使用当前连接配置定位下一标签。

```java
public com.netumscan.printersdk.RequestId locateNextLabel(int, com.netumscan.printersdk.PrinterManager$Callback<java.lang.Void>);
```

在同锁登记时捕获当前CPCL Profile，仅经原设备FIFO执行；配置缺失1002，不支持协议1004。成功仅证明命令发送，不代表实际位置已硬件回读确认。

文档 ID：`android:member:com.netumscan.printersdk.PrinterManager:ad546835fb64a14e`

JVM descriptor：`(ILcom/netumscan/printersdk/PrinterManager$Callback;)Lcom/netumscan/printersdk/RequestId;`

编译器注解：`@androidx.annotation.NonNull`。

参数 1（从 0 起）注解：`@androidx.annotation.NonNull`。

<a id="api-b4ccafa955769754f4fd"></a>

### public com.netumscan.printersdk.RequestId print(com.netumscan.printersdk.PrintOptions, com.netumscan.printersdk.PrinterManager$Callback<java.lang.Void>)

提交完整PrintOptions并返回请求标识。

```java
public com.netumscan.printersdk.RequestId print(com.netumscan.printersdk.PrintOptions, com.netumscan.printersdk.PrinterManager$Callback<java.lang.Void>);
```

提交固定内部连接、有效Profile、截止时间和全部图源引用；轻量保留全成或全败，不在UI解码。后台准备按调用顺序，所有设备IO仍同FIFO。原Callback交付校验、准备、接纳或真实终态；DATA_SENT不是纸面完成。

文档 ID：`android:member:com.netumscan.printersdk.PrinterManager:33c369b19cec2172`

JVM descriptor：`(Lcom/netumscan/printersdk/PrintOptions;Lcom/netumscan/printersdk/PrinterManager$Callback;)Lcom/netumscan/printersdk/RequestId;`

编译器注解：`@androidx.annotation.NonNull`。

参数 0（从 0 起）注解：`@androidx.annotation.NonNull`。

参数 1（从 0 起）注解：`@androidx.annotation.NonNull`。

<a id="api-f54cb667a80e5fd96c3e"></a>

### public com.netumscan.printersdk.RequestId queryStatus(com.netumscan.printersdk.StatusQueryConfig, int, com.netumscan.printersdk.PrinterManager$Callback<com.netumscan.printersdk.StatusResult>)

提交硬件状态查询请求。

```java
public com.netumscan.printersdk.RequestId queryStatus(com.netumscan.printersdk.StatusQueryConfig, int, com.netumscan.printersdk.PrinterManager$Callback<com.netumscan.printersdk.StatusResult>);
```

配置明确协议请求、响应和超时；请求/响应事务占用同一设备队列。结果通过StatusResult报告已知位与活动位，不支持查询时异步失败。

文档 ID：`android:member:com.netumscan.printersdk.PrinterManager:8cb94bf2e247d99b`

JVM descriptor：`(Lcom/netumscan/printersdk/StatusQueryConfig;ILcom/netumscan/printersdk/PrinterManager$Callback;)Lcom/netumscan/printersdk/RequestId;`

编译器注解：`@androidx.annotation.NonNull`。

参数 0（从 0 起）注解：`@androidx.annotation.NonNull`。

参数 2（从 0 起）注解：`@androidx.annotation.NonNull`。

<a id="api-efdba753fb81e704f7f0"></a>

### public com.netumscan.printersdk.RequestId scanBle(java.util.UUID, int, com.netumscan.printersdk.PrinterManager$Callback<java.util.List<com.netumscan.printersdk.DiscoveredDevice>>)

枚举一段时间内发现的BLE设备。

```java
public com.netumscan.printersdk.RequestId scanBle(java.util.UUID, int, com.netumscan.printersdk.PrinterManager$Callback<java.util.List<com.netumscan.printersdk.DiscoveredDevice>>);
```

服务过滤和扫描时长由请求给定，结果只含只读DiscoveredDevice数据；扫描由Manager持有并可按请求标识取消。

文档 ID：`android:member:com.netumscan.printersdk.PrinterManager:a66e97929d58d2ae`

JVM descriptor：`(Ljava/util/UUID;ILcom/netumscan/printersdk/PrinterManager$Callback;)Lcom/netumscan/printersdk/RequestId;`

编译器注解：`@androidx.annotation.NonNull`。

参数 0（从 0 起）注解：`@androidx.annotation.Nullable`。

参数 2（从 0 起）注解：`@androidx.annotation.NonNull`。

<a id="api-89db5e6d23981953297a"></a>

### public com.netumscan.printersdk.RequestId sdkInfo(com.netumscan.printersdk.PrinterManager$Callback<com.netumscan.printersdk.SdkInfo>)

读取已加载SDK的身份信息。

```java
public com.netumscan.printersdk.RequestId sdkInfo(com.netumscan.printersdk.PrinterManager$Callback<com.netumscan.printersdk.SdkInfo>);
```

异步交付分发版本、native版本及ABI版本/修订，用于本地集成诊断；该值不等同于硬件验证证明。

文档 ID：`android:member:com.netumscan.printersdk.PrinterManager:cabce4d23a0c13a1`

JVM descriptor：`(Lcom/netumscan/printersdk/PrinterManager$Callback;)Lcom/netumscan/printersdk/RequestId;`

编译器注解：`@androidx.annotation.NonNull`。

参数 0（从 0 起）注解：`@androidx.annotation.NonNull`。

<a id="api-b5659199b3bac62370ba"></a>

### public com.netumscan.printersdk.RequestId sendRaw(byte[], int, int, com.netumscan.printersdk.PrinterManager$Callback<java.lang.Void>)

发送显式原始命令请求。

```java
public com.netumscan.printersdk.RequestId sendRaw(byte[], int, int, com.netumscan.printersdk.PrinterManager$Callback<java.lang.Void>);
```

原始字节在返回前制作有界快照，大小上限为1MiB；copies和超时在准入时校验。请求仍经过同一native设备队列，与普通打印及查询保持串行顺序。

文档 ID：`android:member:com.netumscan.printersdk.PrinterManager:4906567398108084`

JVM descriptor：`([BIILcom/netumscan/printersdk/PrinterManager$Callback;)Lcom/netumscan/printersdk/RequestId;`

编译器注解：`@androidx.annotation.NonNull`。

参数 0（从 0 起）注解：`@androidx.annotation.NonNull`。

参数 3（从 0 起）注解：`@androidx.annotation.NonNull`。

<a id="api-f06c069bf92e43eed544"></a>

### public com.netumscan.printersdk.RequestId setMedia(com.netumscan.printersdk.MediaSettings, int, com.netumscan.printersdk.PrinterManager$Callback<java.lang.Void>)

使用当前连接配置提交介质设置。

```java
public com.netumscan.printersdk.RequestId setMedia(com.netumscan.printersdk.MediaSettings, int, com.netumscan.printersdk.PrinterManager$Callback<java.lang.Void>);
```

在同锁登记时捕获当前CPCL Profile，不能逐次传入其他配置；配置缺失1002，不支持协议1004。完整发送建立绑定证据，不自动复位、校准或存储。

文档 ID：`android:member:com.netumscan.printersdk.PrinterManager:7e9f8af844c17b79`

JVM descriptor：`(Lcom/netumscan/printersdk/MediaSettings;ILcom/netumscan/printersdk/PrinterManager$Callback;)Lcom/netumscan/printersdk/RequestId;`

编译器注解：`@androidx.annotation.NonNull`。

参数 0（从 0 起）注解：`@androidx.annotation.NonNull`。

参数 2（从 0 起）注解：`@androidx.annotation.NonNull`。

<a id="api-99a5653b30edc0f80216"></a>

### public static final int FEATURE_CUT = 5

切纸能力。

```java
public static final int FEATURE_CUT = 5;
```

该常量的精确名称及编译值由公开基线固定；使用与本类型匹配的字段或参数，能力是否受支持以profile描述和请求校验为准。

文档 ID：`android:member:com.netumscan.printersdk.PrinterManager:7ac9bf17bd0dbac2`

JVM descriptor：`I`

<a id="api-860b6fb550faa21e1bc8"></a>

### public static final int FEATURE_FEED_TO_NEXT_LABEL = 8

下一张标签定位能力。

```java
public static final int FEATURE_FEED_TO_NEXT_LABEL = 8;
```

该常量的精确名称及编译值由公开基线固定；使用与本类型匹配的字段或参数，能力是否受支持以profile描述和请求校验为准。

文档 ID：`android:member:com.netumscan.printersdk.PrinterManager:bb60d15581a3e604`

JVM descriptor：`I`

<a id="api-5862e2dc34ab7af5aafc"></a>

### public static final int FEATURE_IMAGE = 7

图像打印能力。

```java
public static final int FEATURE_IMAGE = 7;
```

该常量的精确名称及编译值由公开基线固定；使用与本类型匹配的字段或参数，能力是否受支持以profile描述和请求校验为准。

文档 ID：`android:member:com.netumscan.printersdk.PrinterManager:bbf3ec21c86c2b1d`

JVM descriptor：`I`

<a id="api-99fc492258e13ba8a594"></a>

### public static final int FEATURE_LABEL = 1

标签打印能力。

```java
public static final int FEATURE_LABEL = 1;
```

该常量的精确名称及编译值由公开基线固定；使用与本类型匹配的字段或参数，能力是否受支持以profile描述和请求校验为准。

文档 ID：`android:member:com.netumscan.printersdk.PrinterManager:4c5619690e84e1bf`

JVM descriptor：`I`

<a id="api-719f05fa74d690d82c3b"></a>

### public static final int FEATURE_MEDIA = 4

介质设置能力。

```java
public static final int FEATURE_MEDIA = 4;
```

该常量的精确名称及编译值由公开基线固定；使用与本类型匹配的字段或参数，能力是否受支持以profile描述和请求校验为准。

文档 ID：`android:member:com.netumscan.printersdk.PrinterManager:bdac68bb07a3ff93`

JVM descriptor：`I`

<a id="api-99299bc1232a036b8491"></a>

### public static final int FEATURE_RAW = 6

原始命令能力。

```java
public static final int FEATURE_RAW = 6;
```

该常量的精确名称及编译值由公开基线固定；使用与本类型匹配的字段或参数，能力是否受支持以profile描述和请求校验为准。

文档 ID：`android:member:com.netumscan.printersdk.PrinterManager:7bb504424a281ed6`

JVM descriptor：`I`

<a id="api-e0a815cc01161ca39ea6"></a>

### public static final int FEATURE_RECEIPT = 2

小票打印能力。

```java
public static final int FEATURE_RECEIPT = 2;
```

该常量的精确名称及编译值由公开基线固定；使用与本类型匹配的字段或参数，能力是否受支持以profile描述和请求校验为准。

文档 ID：`android:member:com.netumscan.printersdk.PrinterManager:34743d368e7191d9`

JVM descriptor：`I`

<a id="api-b39525c6bb6bc9b8c8ca"></a>

### public static final int FEATURE_STATUS = 3

状态查询能力。

```java
public static final int FEATURE_STATUS = 3;
```

该常量的精确名称及编译值由公开基线固定；使用与本类型匹配的字段或参数，能力是否受支持以profile描述和请求校验为准。

文档 ID：`android:member:com.netumscan.printersdk.PrinterManager:57a2e549cda799a3`

JVM descriptor：`I`

<a id="api-c495b55a4368ea9d6525"></a>

### public void cancel(com.netumscan.printersdk.RequestId)

请求取消本Manager拥有的请求。

```java
public void cancel(com.netumscan.printersdk.RequestId);
```

取消意图覆盖等待、准备、准入与native执行阶段；外部Manager的标识和已完成请求不改变设备。取消本身不证明没有输出，最终结果的outputMayHaveOccurred用于提示可能已部分发送。

文档 ID：`android:member:com.netumscan.printersdk.PrinterManager:80c2dece04d1d2f6`

JVM descriptor：`(Lcom/netumscan/printersdk/RequestId;)V`

参数 0（从 0 起）注解：`@androidx.annotation.Nullable`。

<a id="api-a8eff59edaee0576f508"></a>

## PrinterManager.Callback

单次业务请求的完成回调。

```java
public interface com.netumscan.printersdk.PrinterManager$Callback<T>
```

Manager在主线程异步交付一次ManagerResult；参数错误、状态错误、准备失败、准入失败和native终态都使用同一完成通道。 `Callback` 是 `@FunctionalInterface`，方法名为 `onComplete`，接收 `@NonNull ManagerResult<T>`，并以 `@MainThread` 声明回调线程；支持 Java lambda 和匿名类。

文档 ID：`android:type:com.netumscan.printersdk.PrinterManager$Callback`

编译器注解：`@java.lang.FunctionalInterface`。

<a id="api-fda923ad62298237ac93"></a>

### public abstract void onComplete(com.netumscan.printersdk.ManagerResult<T>)

单次业务请求的完成回调。

```java
public abstract void onComplete(com.netumscan.printersdk.ManagerResult<T>);
```

Manager在主线程异步交付一次ManagerResult；参数错误、状态错误、准备失败、准入失败和native终态都使用同一完成通道。 `Callback` 是 `@FunctionalInterface`，方法名为 `onComplete`，接收 `@NonNull ManagerResult<T>`，并以 `@MainThread` 声明回调线程；支持 Java lambda 和匿名类。

文档 ID：`android:member:com.netumscan.printersdk.PrinterManager$Callback:ad4a015b2cb5b1b7`

JVM descriptor：`(Lcom/netumscan/printersdk/ManagerResult;)V`

编译器注解：`@androidx.annotation.MainThread`。

参数 0（从 0 起）注解：`@androidx.annotation.NonNull`。

<a id="api-9149b393dfb876ce6b8e"></a>

## PrinterManager.DisconnectedListener

当前连接的意外断连通知。

```java
public interface com.netumscan.printersdk.PrinterManager$DisconnectedListener
```

通知只属于注册时的连接会话；主动disconnect通过自身完成回调报告，旧会话事件不能影响后续连接。 `DisconnectedListener` 是 `@FunctionalInterface`，方法名为 `onDisconnected`，接收 `@NonNull ManagerResult<Void>`，以 `@MainThread` 声明回调线程。

文档 ID：`android:type:com.netumscan.printersdk.PrinterManager$DisconnectedListener`

编译器注解：`@java.lang.FunctionalInterface`。

<a id="api-4ba251e1239d4f7bbd7f"></a>

### public abstract void onDisconnected(com.netumscan.printersdk.ManagerResult<java.lang.Void>)

当前连接的意外断连通知。

```java
public abstract void onDisconnected(com.netumscan.printersdk.ManagerResult<java.lang.Void>);
```

通知只属于注册时的连接会话；主动disconnect通过自身完成回调报告，旧会话事件不能影响后续连接。 `DisconnectedListener` 是 `@FunctionalInterface`，方法名为 `onDisconnected`，接收 `@NonNull ManagerResult<Void>`，以 `@MainThread` 声明回调线程。

文档 ID：`android:member:com.netumscan.printersdk.PrinterManager$DisconnectedListener:bec808a8d85c7ee3`

JVM descriptor：`(Lcom/netumscan/printersdk/ManagerResult;)V`

编译器注解：`@androidx.annotation.MainThread`。

参数 0（从 0 起）注解：`@androidx.annotation.NonNull`。

<a id="api-926b9d0552ac918f6e21"></a>

## PrinterManager.Preparation

保留的旧纯数据准备接口。

```java
public interface com.netumscan.printersdk.PrinterManager$Preparation
```

保留接口类型和工作线程注解以避免额外类型删除；Manager已无接受此接口的公开打印入口。当前自定义图像准备使用ImageSource.deferred，将内容与布局统一放入PrintOptions。

文档 ID：`android:type:com.netumscan.printersdk.PrinterManager$Preparation`

编译器注解：`@java.lang.FunctionalInterface`。

<a id="api-71d53807452a19f29817"></a>

### public abstract com.netumscan.printersdk.PreparedPrint prepare(com.netumscan.printersdk.CancellationContext) throws java.lang.Exception

旧准备接口的纯数据方法声明。

```java
public abstract com.netumscan.printersdk.PreparedPrint prepare(com.netumscan.printersdk.CancellationContext) throws java.lang.Exception;
```

接收CancellationContext并返回非空PreparedPrint；Manager不再调用或接收此接口。迁移到ImageSource.deferred的ImageLoader，布局放入PrintOptions。方法本身不得执行设备IO。

文档 ID：`android:member:com.netumscan.printersdk.PrinterManager$Preparation:8098e318858ba01d`

JVM descriptor：`(Lcom/netumscan/printersdk/CancellationContext;)Lcom/netumscan/printersdk/PreparedPrint;`

编译器注解：`@androidx.annotation.NonNull`、`@androidx.annotation.WorkerThread`。

参数 0（从 0 起）注解：`@androidx.annotation.NonNull`。

<a id="api-73cf28c112dad2127298"></a>

## PrinterProfile

不可变的连接打印配置。

```java
public final class com.netumscan.printersdk.PrinterProfile
```

包含协议、名义DPI、精确dotsPerMm、最大dots宽度、标签定位规则及可空型号。仅连接时SDK名称匹配或连接后空闲configurePrintProfile采用；不包含单张尺寸、份数、FORM或校准。每请求原子固定快照，更新成功失效介质证据，断开清除；不表示硬件回读确认。

文档 ID：`android:type:com.netumscan.printersdk.PrinterProfile`

<a id="api-cd39945f3be919193d43"></a>

### public final com.netumscan.printersdk.PrinterProfile$LabelAlignment labelAlignment

设备的标签定位规则。

```java
public final com.netumscan.printersdk.PrinterProfile$LabelAlignment labelAlignment;
```

非空不可变，LEFT从左端开始，CENTERED按标签整字节宽度居中；单次PrintOptions不能覆盖该规则。LT-M90高层Profile使用CENTERED，旧通用工厂使用LEFT。

文档 ID：`android:member:com.netumscan.printersdk.PrinterProfile:3f41d35c62ee6605`

JVM descriptor：`Lcom/netumscan/printersdk/PrinterProfile$LabelAlignment;`

编译器注解：`@androidx.annotation.NonNull`。

<a id="api-a33286cf84b4dfed5151"></a>

### public final com.netumscan.printersdk.PrinterProfile$Protocol protocol

结构化打印采用的协议。

```java
public final com.netumscan.printersdk.PrinterProfile$Protocol protocol;
```

CPCL标签或ESC_POS小票，不随SPP/BLE传输自动改变；有效Profile与作业类型不匹配时拒绝。

文档 ID：`android:member:com.netumscan.printersdk.PrinterProfile:e3cf81c945fab8ef`

JVM descriptor：`Lcom/netumscan/printersdk/PrinterProfile$Protocol;`

编译器注解：`@androidx.annotation.NonNull`。

<a id="api-063984caeb0a1f9d1837"></a>

### public final double dotsPerMm

毫米换算使用的精确物理点密度。

```java
public final double dotsPerMm;
```

正有限数，单位dots/mm。LT-M90按已提供的有效108mm／864dots规格采用8；旧通用工厂默认dpi/25.4。labelMm、containBoxMm和图片解码目标共用提交时Profile快照；图片文件DPI不参与。

文档 ID：`android:member:com.netumscan.printersdk.PrinterProfile:8368f0091ca5d535`

JVM descriptor：`D`

<a id="api-b9ac42681a209b097b87"></a>

### public final int dpi

名义打印分辨率与协议参数。

```java
public final int dpi;
```

正整数，ESC/POS为1..255；用于协议命令，不是屏幕或图片DPI。毫米换算使用独立dotsPerMm，LT-M90保留203协议参数并用8dots/mm。名称推荐不是硬件回读。

文档 ID：`android:member:com.netumscan.printersdk.PrinterProfile:a54ea9eab4fa22b3`

JVM descriptor：`I`

<a id="api-3e0b657f36ac9edad600"></a>

### public final int maxPrintWidthDots

配置的最大输出宽度。

```java
public final int maxPrintWidthDots;
```

单位dots，正数且8对齐、最大7992；本次标签宽度另由PrintOptions提供，不能把画布宽度自动当成最大能力。

文档 ID：`android:member:com.netumscan.printersdk.PrinterProfile:8cb261ed04118bf2`

JVM descriptor：`I`

<a id="api-d215e9b4ad6dcba21e15"></a>

### public final java.lang.String modelId

可空的预设型号标识。

```java
public final java.lang.String modelId;
```

自定义Profile返回null，不使用特殊占位字符串；标识不提供连接控制或硬件认证。

文档 ID：`android:member:com.netumscan.printersdk.PrinterProfile:8b8488e3a097865d`

JVM descriptor：`Ljava/lang/String;`

编译器注解：`@androidx.annotation.Nullable`。

<a id="api-ed5f86e3594cb794d75d"></a>

### public int dotsFromMm(double)

按当前Profile将毫米四舍五入为dots。

```java
public int dotsFromMm(double);
```

纯值round(mm*dotsPerMm)，输入须正有限且结果为1..Integer.MAX_VALUE，否则同步IllegalArgumentException。结果不强制8倍数，不限制到设备最大宽度，具体标签范围在提交时检查。LT-M90的50／99／108mm分别为400／792／864dots。

文档 ID：`android:member:com.netumscan.printersdk.PrinterProfile:08c2e65ebf32547e`

JVM descriptor：`(D)I`

<a id="api-efb188161e2d33405145"></a>

### public int labelOriginXDots(int, int)

预览标签整体在打印头中的横向起点。

```java
public int labelOriginXDots(int, int);
```

仅CPCL，宽度须正数；packed=ceil(widthDots/8)*8。CENTERED使用floor((maxPrintWidthDots-packed)/2)+calibration，LEFT使用calibration；校准为有符号dots。纯值无IO，负起点、右越界、超宽或溢出同步IllegalArgumentException，不裁切或钳制。真实打印使用提交时Profile重新计算。

文档 ID：`android:member:com.netumscan.printersdk.PrinterProfile:d79b13bc7b73294c`

JVM descriptor：`(II)I`

<a id="api-0441d0448ea84d4379e8"></a>

### public static com.netumscan.printersdk.PrinterProfile cpcl(int, int)

创建保留通用默认行为的CPCL配置。

```java
public static com.netumscan.printersdk.PrinterProfile cpcl(int, int);
```

DPI须正数，最大宽度1..7992且8对齐；默认LEFT及dpi/25.4 dots/mm。不包含标签宽高、份数、FORM或校准。需要精确点密度和居中设备规则时使用四参数纯值工厂；采用仅通过连接后configurePrintProfile。

文档 ID：`android:member:com.netumscan.printersdk.PrinterProfile:f00e6064dc294be2`

JVM descriptor：`(II)Lcom/netumscan/printersdk/PrinterProfile;`

编译器注解：`@androidx.annotation.NonNull`。

<a id="api-d202feb9dd8a5c7b4961"></a>

### public static com.netumscan.printersdk.PrinterProfile cpcl(int, int, double, com.netumscan.printersdk.PrinterProfile$LabelAlignment)

创建含精确点密度与标签定位规则的CPCL配置。

```java
public static com.netumscan.printersdk.PrinterProfile cpcl(int, int, double, com.netumscan.printersdk.PrinterProfile$LabelAlignment);
```

dpi为正整数，最大宽度1..7992且8对齐，dotsPerMm须正有限，labelAlignment非空。只构造不可变值、不初始化设备；仅能在连接后configurePrintProfile采用。dpi用于协议，精确密度用于mm换算，两者职责独立；不从最大纸张宽度推断点密度。

文档 ID：`android:member:com.netumscan.printersdk.PrinterProfile:ad5a01271bd52d99`

JVM descriptor：`(IIDLcom/netumscan/printersdk/PrinterProfile$LabelAlignment;)Lcom/netumscan/printersdk/PrinterProfile;`

编译器注解：`@androidx.annotation.NonNull`。

参数 3（从 0 起）注解：`@androidx.annotation.NonNull`。

<a id="api-c701ff56339fc66d176a"></a>

### public static com.netumscan.printersdk.PrinterProfile escPos(int, int)

创建自定义ESC/POS稳定配置。

```java
public static com.netumscan.printersdk.PrinterProfile escPos(int, int);
```

DPI为1..255，最大宽度1..7992且8对齐；点密度默认为dpi/25.4，标签定位规则为LEFT。只描述协议与几何，不自动切纸、查询或配置介质。

文档 ID：`android:member:com.netumscan.printersdk.PrinterProfile:192267bb81e2c6c7`

JVM descriptor：`(II)Lcom/netumscan/printersdk/PrinterProfile;`

编译器注解：`@androidx.annotation.NonNull`。

<a id="api-044125a48fcbc00942aa"></a>

## PrinterProfile.LabelAlignment

设备的标签横向定位规则。

```java
public final class com.netumscan.printersdk.PrinterProfile$LabelAlignment extends java.lang.Enum<com.netumscan.printersdk.PrinterProfile$LabelAlignment>
```

属于不可变PrinterProfile，只能通过当前连接配置采用，不由单次打印切换。描述整个标签画布在打印头内的位置，不改变图像在画布内的布局。

文档 ID：`android:type:com.netumscan.printersdk.PrinterProfile$LabelAlignment`

枚举声明顺序（`ordinal()` 从 0 起；不作为持久化协议值）：

| ordinal | 枚举项 |
| --- | --- |
| 0 | `LEFT` |
| 1 | `CENTERED` |

<a id="api-80abed8e8feca1659171"></a>

### public static com.netumscan.printersdk.PrinterProfile$LabelAlignment valueOf(java.lang.String)

按精确枚举名查找定位规则。

```java
public static com.netumscan.printersdk.PrinterProfile$LabelAlignment valueOf(java.lang.String);
```

Java编译器生成；名称须精确匹配，null或未知名称使用Java标准异常，不执行设备IO。

文档 ID：`android:member:com.netumscan.printersdk.PrinterProfile$LabelAlignment:18acc82548f07d2d`

JVM descriptor：`(Ljava/lang/String;)Lcom/netumscan/printersdk/PrinterProfile$LabelAlignment;`

这是 Java 编译器为枚举生成的公共方法。

<a id="api-01ddc6e725e1786c8c2a"></a>

### public static com.netumscan.printersdk.PrinterProfile$LabelAlignment[] values()

取得声明顺序的标签定位规则。

```java
public static com.netumscan.printersdk.PrinterProfile$LabelAlignment[] values();
```

Java编译器生成的新数组；修改数组不改变枚举或已采用Profile。

文档 ID：`android:member:com.netumscan.printersdk.PrinterProfile$LabelAlignment:21b614e27a839404`

JVM descriptor：`()[Lcom/netumscan/printersdk/PrinterProfile$LabelAlignment;`

这是 Java 编译器为枚举生成的公共方法。

<a id="api-0ff3cd1e67743b9b22e6"></a>

### public static final com.netumscan.printersdk.PrinterProfile$LabelAlignment CENTERED

标签整体按整字节宽度在打印头内居中。

```java
public static final com.netumscan.printersdk.PrinterProfile$LabelAlignment CENTERED;
```

LT-M90高层打印Profile采用此规则；默认起点由SDK按每张标签宽度计算，可叠加有符号校准，不能逐请求改为绝对X。

文档 ID：`android:member:com.netumscan.printersdk.PrinterProfile$LabelAlignment:f6824af889a4d171`

JVM descriptor：`Lcom/netumscan/printersdk/PrinterProfile$LabelAlignment;`

<a id="api-08f3f909173bb4dbaa2e"></a>

### public static final com.netumscan.printersdk.PrinterProfile$LabelAlignment LEFT

标签从打印头左端起始。

```java
public static final com.netumscan.printersdk.PrinterProfile$LabelAlignment LEFT;
```

旧通用工厂默认规则。可以使用有符号校准，或保留的originDots绝对起点，但非零校准与非零绝对X不能混用。

文档 ID：`android:member:com.netumscan.printersdk.PrinterProfile$LabelAlignment:8e4a61e01f575574`

JVM descriptor：`Lcom/netumscan/printersdk/PrinterProfile$LabelAlignment;`

<a id="api-a6f2b17ef78a597d41ca"></a>

## PrinterProfile.Protocol

结构化打印协议类型。

```java
public final class com.netumscan.printersdk.PrinterProfile$Protocol extends java.lang.Enum<com.netumscan.printersdk.PrinterProfile$Protocol>
```

CPCL对应标签，ESC_POS对应小票。协议独立于通信方式，名称推荐不是设备能力回读；不支持的组合明确拒绝。

文档 ID：`android:type:com.netumscan.printersdk.PrinterProfile$Protocol`

枚举声明顺序（`ordinal()` 从 0 起；不作为持久化协议值）：

| ordinal | 枚举项 |
| --- | --- |
| 0 | `CPCL` |
| 1 | `ESC_POS` |

<a id="api-77a29c55c803357672eb"></a>

### public static com.netumscan.printersdk.PrinterProfile$Protocol valueOf(java.lang.String)

按精确枚举名查找。

```java
public static com.netumscan.printersdk.PrinterProfile$Protocol valueOf(java.lang.String);
```

Java编译器生成的方法，名称必须精确匹配；null或未知名称使用Java标准异常，不执行设备IO。

文档 ID：`android:member:com.netumscan.printersdk.PrinterProfile$Protocol:1ccb2e17ed6d929a`

JVM descriptor：`(Ljava/lang/String;)Lcom/netumscan/printersdk/PrinterProfile$Protocol;`

这是 Java 编译器为枚举生成的公共方法。

<a id="api-e965d2b59b5f65e0f89a"></a>

### public static com.netumscan.printersdk.PrinterProfile$Protocol[] values()

取得按声明顺序排列的枚举值。

```java
public static com.netumscan.printersdk.PrinterProfile$Protocol[] values();
```

Java编译器生成的新数组，修改数组不修改枚举定义或SDK配置。

文档 ID：`android:member:com.netumscan.printersdk.PrinterProfile$Protocol:c37c277941f01c8f`

JVM descriptor：`()[Lcom/netumscan/printersdk/PrinterProfile$Protocol;`

这是 Java 编译器为枚举生成的公共方法。

<a id="api-f0986c78728665ebeae4"></a>

### public static final com.netumscan.printersdk.PrinterProfile$Protocol CPCL

CPCL标签协议。

```java
public static final com.netumscan.printersdk.PrinterProfile$Protocol CPCL;
```

仅选择协议，不决定传输、介质、标签高度或FORM。

文档 ID：`android:member:com.netumscan.printersdk.PrinterProfile$Protocol:a26d5cec3befafe4`

JVM descriptor：`Lcom/netumscan/printersdk/PrinterProfile$Protocol;`

<a id="api-67866c2a6a219343952f"></a>

### public static final com.netumscan.printersdk.PrinterProfile$Protocol ESC_POS

ESC/POS小票协议。

```java
public static final com.netumscan.printersdk.PrinterProfile$Protocol ESC_POS;
```

仅选择协议，未实现切纸／状态读回仍明确不支持。

文档 ID：`android:member:com.netumscan.printersdk.PrinterProfile$Protocol:2ef159eee25b9753`

JVM descriptor：`Lcom/netumscan/printersdk/PrinterProfile$Protocol;`

<a id="api-c4cd867a0c3c5e8c017c"></a>

## PrinterProfiles

按蓝牙名称提供 BLE／CPCL 推荐配置，显式配置优先。

```java
public final class com.netumscan.printersdk.PrinterProfiles
```

仅完整 ASCII 名称 `LT-M90_[0-9A-Fa-f]{4}` 匹配，型号前缀区分大小写；不修剪输入，空值、空白、换行、未知型号或格式不符均无推荐。名称只作为配置线索，不核对蓝牙地址或认证硬件。 两个静态方法独立解析配置，同步执行且不进行扫描、连接、打印机 IO 或 Manager 状态保存；不创建型号聚合对象，不推断 ESC/POS、介质、FORM 或业务布局。

文档 ID：`android:type:com.netumscan.printersdk.PrinterProfiles`

<a id="api-d7228a40d4fb9435170a"></a>

### public static com.netumscan.printersdk.BleOptions resolveBleOptions(java.lang.String, com.netumscan.printersdk.BleOptions)

解析名称对应的 BLE 推荐配置，优先返回显式配置。

```java
public static com.netumscan.printersdk.BleOptions resolveBleOptions(java.lang.String, com.netumscan.printersdk.BleOptions);
```

`explicitOptions` 非 null 时原样返回该对象，不检查名称。否则，仅完整 ASCII 名称 `LT-M90_[0-9A-Fa-f]{4}` 匹配，型号前缀区分大小写；不修剪输入，空值、空白、换行、未知型号或格式不符均无推荐。名称只作为配置线索，不核对蓝牙地址或认证硬件。 未命中返回 null；命中调用 `LTM90Profile.bleOptions()`，使用 MTU 请求 517、写入上限 512 字节以及现有无响应写入／Credit 配置。上限不保证实际包长；方法不发起连接或 IO。 这是无IO纯值工厂，不改变Manager；手动采用结果必须在连接后调用configurePrintProfile。

文档 ID：`android:member:com.netumscan.printersdk.PrinterProfiles:a1c913d41cbe74cb`

JVM descriptor：`(Ljava/lang/String;Lcom/netumscan/printersdk/BleOptions;)Lcom/netumscan/printersdk/BleOptions;`

编译器注解：`@androidx.annotation.Nullable`。

参数 0（从 0 起）注解：`@androidx.annotation.Nullable`。

参数 1（从 0 起）注解：`@androidx.annotation.Nullable`。

<a id="api-c7bca183899d17caaa17"></a>

### public static com.netumscan.printersdk.CpclProfile resolveCpclProfile(java.lang.String, com.netumscan.printersdk.CpclProfile, int, boolean)

解析名称对应的 CPCL 推荐配置，优先返回显式配置。

```java
public static com.netumscan.printersdk.CpclProfile resolveCpclProfile(java.lang.String, com.netumscan.printersdk.CpclProfile, int, boolean);
```

explicitProfile非null时原样返回，不检查名称或兜底参数；否则仅完整ASCII名称LT-M90_[0-9A-Fa-f]{4}匹配，未知返回null。命中调用旧LTM90Profile.cpclProfile(maxLabelHeightDots,form)，页高1..65535、原点0，保留纯值兼容语义且不执行IO。Manager不接收此CpclProfile或逐操作显式配置，不能用此工厂绕过高层标签定位。当前打印配置使用PrinterProfile的自动匹配或连接后configurePrintProfile。

文档 ID：`android:member:com.netumscan.printersdk.PrinterProfiles:abdc93cc5212ff39`

JVM descriptor：`(Ljava/lang/String;Lcom/netumscan/printersdk/CpclProfile;IZ)Lcom/netumscan/printersdk/CpclProfile;`

编译器注解：`@androidx.annotation.Nullable`。

参数 0（从 0 起）注解：`@androidx.annotation.Nullable`。

参数 1（从 0 起）注解：`@androidx.annotation.Nullable`。

<a id="api-512f33163e8a33506861"></a>

### public static com.netumscan.printersdk.PrinterProfile forModel(java.lang.String)

按精确型号取得稳定打印预设。

```java
public static com.netumscan.printersdk.PrinterProfile forModel(java.lang.String);
```

当前只接受LT-M90，未知或null同步参数异常；不发IO，不猜测页高、FORM或其他型号协议。

文档 ID：`android:member:com.netumscan.printersdk.PrinterProfiles:c96310671dd00874`

JVM descriptor：`(Ljava/lang/String;)Lcom/netumscan/printersdk/PrinterProfile;`

<a id="api-f727e591d381e5a65971"></a>

### public static com.netumscan.printersdk.PrinterProfile resolvePrintProfile(java.lang.String)

按原始蓝牙名提供打印配置推荐。

```java
public static com.netumscan.printersdk.PrinterProfile resolvePrintProfile(java.lang.String);
```

只完整匹配LT-M90_[0-9A-Fa-f]{4}，不修剪、不核对地址。显式Profile优先且原样返回；无显式值且未知名称返回null，不认证硬件。 这是无IO纯值工厂，不改变Manager；手动采用结果必须在连接后调用configurePrintProfile。

文档 ID：`android:member:com.netumscan.printersdk.PrinterProfiles:da8b48dde2e1fbed`

JVM descriptor：`(Ljava/lang/String;)Lcom/netumscan/printersdk/PrinterProfile;`

编译器注解：`@androidx.annotation.Nullable`。

参数 0（从 0 起）注解：`@androidx.annotation.Nullable`。

<a id="api-3ba464b7cf78ab7b98e7"></a>

### public static com.netumscan.printersdk.PrinterProfile resolvePrintProfile(java.lang.String, com.netumscan.printersdk.PrinterProfile)

按原始蓝牙名提供打印配置推荐。

```java
public static com.netumscan.printersdk.PrinterProfile resolvePrintProfile(java.lang.String, com.netumscan.printersdk.PrinterProfile);
```

只完整匹配LT-M90_[0-9A-Fa-f]{4}，不修剪、不核对地址。显式Profile优先且原样返回；无显式值且未知名称返回null，不认证硬件。 这是无IO纯值工厂，不改变Manager；手动采用结果必须在连接后调用configurePrintProfile。

文档 ID：`android:member:com.netumscan.printersdk.PrinterProfiles:3dcfd817cf075dea`

JVM descriptor：`(Ljava/lang/String;Lcom/netumscan/printersdk/PrinterProfile;)Lcom/netumscan/printersdk/PrinterProfile;`

编译器注解：`@androidx.annotation.Nullable`。

参数 0（从 0 起）注解：`@androidx.annotation.Nullable`。

参数 1（从 0 起）注解：`@androidx.annotation.Nullable`。

<a id="api-764c5a4147cfdf641cc3"></a>

## RequestId

仅由原Manager识别的不透明请求标识。

```java
public final class com.netumscan.printersdk.RequestId
```

将该值交给返回它的Manager.cancel请求取消；标识本身不持有native任务句柄，也不提供独立打印控制入口。

文档 ID：`android:type:com.netumscan.printersdk.RequestId`

<a id="api-29b3eab038d8022694c1"></a>

### public java.lang.String toString()

读取请求标识的辅助文本。

```java
public java.lang.String toString();
```

仅用于应用诊断；取消应传递原RequestId对象，不应解析文本重建native任务标识。

文档 ID：`android:member:com.netumscan.printersdk.RequestId:a9e4d7b84d18cdab`

JVM descriptor：`()Ljava/lang/String;`

<a id="api-b49ec93e3a988bef134a"></a>

## ScalarConstraint

不可变标量约束及依据。

```java
public final class com.netumscan.printersdk.ScalarConstraint
```

parameterId 指定约束，known=false 表示未知，不能解释为无限制。源载荷约束来自 SDK 接纳策略；协议几何或硬件支持仍须使用相应配置和能力证据。

文档 ID：`android:type:com.netumscan.printersdk.ScalarConstraint`

<a id="api-8477e1284dd57f499175"></a>

### public final boolean known

此约束是否有可靠已知范围。

```java
public final boolean known;
```

false 时 minimum、maximum、step 不能解释为无限制或硬件保证。

文档 ID：`android:member:com.netumscan.printersdk.ScalarConstraint:a2fcb139f5d87956`

JVM descriptor：`Z`

<a id="api-d43001a41eba5b3320b7"></a>

### public final int evidence

范围的证据来源。

```java
public final int evidence;
```

区分 SDK 接纳策略、协议定义及其他证据，不能把 SDK 限制解释为已验证硬件能力。

文档 ID：`android:member:com.netumscan.printersdk.ScalarConstraint:958eb37813714ec1`

JVM descriptor：`I`

<a id="api-0e47382cd2419c821178"></a>

### public final int maximum

已知约束的闭区间最大值。

```java
public final int maximum;
```

单位由 parameterId 决定；仅 known=true 时有意义。

文档 ID：`android:member:com.netumscan.printersdk.ScalarConstraint:324164fa97d559e7`

JVM descriptor：`I`

<a id="api-5efa04f35bb99b9c6002"></a>

### public final int minimum

已知约束的闭区间最小值。

```java
public final int minimum;
```

单位由 parameterId 决定；仅 known=true 时有意义。

文档 ID：`android:member:com.netumscan.printersdk.ScalarConstraint:4c11d30ea9a16e9f`

JVM descriptor：`I`

<a id="api-118090c567ac5cb1249a"></a>

### public final int parameterId

本快照的约束参数标识。

```java
public final int parameterId;
```

使用 COPIES、MEDIA_GAP_UM、JOB_IMAGE_BYTES 或 PENDING_PAYLOAD_BYTES 查询；未知标识返回已知位为 false 的约束。

文档 ID：`android:member:com.netumscan.printersdk.ScalarConstraint:55be3d55c85c3eb8`

JVM descriptor：`I`

<a id="api-fefbccf3a2f0c364c608"></a>

### public final long step

已知约束的取值步长。

```java
public final long step;
```

在 minimum 起点按此步长取值；仅 known=true 时有意义。

文档 ID：`android:member:com.netumscan.printersdk.ScalarConstraint:5aa9cefccae2739d`

JVM descriptor：`J`

<a id="api-7b40261535e852085c70"></a>

### public static final int COPIES = 1

份数约束标识，单位份。

```java
public static final int COPIES = 1;
```

固定公共查询参数标识；范围通过 PrinterCapabilities.constraint 读取，未知时不能推断无限制。

文档 ID：`android:member:com.netumscan.printersdk.ScalarConstraint:1cb3e86c38667f1f`

JVM descriptor：`I`

<a id="api-e3f831803f36216af72a"></a>

### public static final int JOB_IMAGE_BYTES = 3

单作业图像源载荷约束标识，单位字节。

```java
public static final int JOB_IMAGE_BYTES = 3;
```

固定公共查询参数标识；范围通过 PrinterCapabilities.constraint 读取，未知时不能推断无限制。

文档 ID：`android:member:com.netumscan.printersdk.ScalarConstraint:633c5aa13832b2cb`

JVM descriptor：`I`

<a id="api-f817ae755a9cdbe8ec44"></a>

### public static final int MEDIA_GAP_UM = 2

介质间隙约束标识，单位微米。

```java
public static final int MEDIA_GAP_UM = 2;
```

固定公共查询参数标识；范围通过 PrinterCapabilities.constraint 读取，未知时不能推断无限制。

文档 ID：`android:member:com.netumscan.printersdk.ScalarConstraint:050af49c580b838f`

JVM descriptor：`I`

<a id="api-6c3c1651b1e3c2abcccf"></a>

### public static final int PENDING_PAYLOAD_BYTES = 4

设备未终态源载荷约束标识，单位字节。

```java
public static final int PENDING_PAYLOAD_BYTES = 4;
```

固定公共查询参数标识；范围通过 PrinterCapabilities.constraint 读取，未知时不能推断无限制。

文档 ID：`android:member:com.netumscan.printersdk.ScalarConstraint:da0e3bdd16ea3c2c`

JVM descriptor：`I`

<a id="api-e68d6e49886d528a891c"></a>

## SdkInfo

实际加载原生运行库的身份。

```java
public final class com.netumscan.printersdk.SdkInfo
```

由原生 C ABI 返回版本和构建身份，与 Demo BuildConfig 独立。生产 CMake 将原生版本、分发 VERSION 和源输入 SHA-256 编入库；缺失构建身份明确未知。该信息不证明签名、发布或硬件兼容。

文档 ID：`android:type:com.netumscan.printersdk.SdkInfo`

<a id="api-2fe33d151571157e902c"></a>

### public final int abiRevision

实际加载库的abiRevision。

```java
public final int abiRevision;
```

当前ABI3最低修订1，须与同版包装一起使用。

文档 ID：`android:member:com.netumscan.printersdk.SdkInfo:3c9a67871faec9fc`

JVM descriptor：`I`

<a id="api-fa7deb09d69a004e2a1f"></a>

### public final int abiVersion

实际加载库的abiVersion。

```java
public final int abiVersion;
```

当前ABI主版本3；ABI2不能混用，必须成套更新包装与库。

文档 ID：`android:member:com.netumscan.printersdk.SdkInfo:da69401829dacd32`

JVM descriptor：`I`

<a id="api-8a11e88892aadb1fa51d"></a>

### public final java.lang.String buildId

可选原生源构建 SHA-256 标识。

```java
public final java.lang.String buildId;
```

未知时为 null。生产 CMake 对受审定原生源和构建文件白名单计算，不包含 Git dirty、主机路径、环境或私有配置；不等同于完整二进制哈希。

文档 ID：`android:member:com.netumscan.printersdk.SdkInfo:1637e37d8d13a768`

JVM descriptor：`Ljava/lang/String;`

<a id="api-c3ffff62c449b0a3e8f2"></a>

### public final java.lang.String distributionVersion

构建库时嵌入的分发版本。

```java
public final java.lang.String distributionVersion;
```

生产 CMake 来源于 VERSION；未嵌入时为空字符串，不猜测或继承 Demo 版本。

文档 ID：`android:member:com.netumscan.printersdk.SdkInfo:17e16ecc61815642`

JVM descriptor：`Ljava/lang/String;`

<a id="api-17210b8724d6da37e2a2"></a>

### public final java.lang.String nativeVersion

实际加载库的nativeVersion。

```java
public final java.lang.String nativeVersion;
```

当前原生SDK语义版本0.3.0，生产CMake来源PROJECT_VERSION；不表示已发布。

文档 ID：`android:member:com.netumscan.printersdk.SdkInfo:9d70e248890bcc50`

JVM descriptor：`Ljava/lang/String;`

<a id="api-31d51fc34c8d04a03b44"></a>

## StatusQueryConfig

逐次硬件状态查询配置。

```java
public final class com.netumscan.printersdk.StatusQueryConfig
```

与打印几何、纸宽和 DPI 独立，接纳时复制，不改变连接默认值。通过参数和生命周期校验后，标准 CPCL/ESC/POS 后端因不支持状态查询而拒绝时，`queryStatus` 通过主线程完成回调异步返回 1004 UNSUPPORTED_OPERATION，不执行打印机 IO。未连接、已关闭或参数非法等情况仍返回相应校验错误；硬件状态仍未知。

文档 ID：`android:type:com.netumscan.printersdk.StatusQueryConfig`

<a id="api-972333d5cecdc3dfd0e8"></a>

### public com.netumscan.printersdk.StatusQueryConfig(int, int)

创建本次状态查询配置。

```java
public com.netumscan.printersdk.StatusQueryConfig(int, int);
```

protocol 只能是 CPCL 或 ESC_POS，queryTimeoutMs 必须为正整数毫秒，非法值同步抛 IllegalArgumentException。构造不执行 IO，不设置打印几何，也不证明固件支持状态查询。

文档 ID：`android:member:com.netumscan.printersdk.StatusQueryConfig:bfc5664f8357a682`

JVM descriptor：`(II)V`

<a id="api-7d926b09b7dd58da8403"></a>

### public final int protocol

本次状态查询选择的协议。

```java
public final int protocol;
```

只接受 CPCL=1 或 ESC_POS=2；选择协议不证明目标固件支持状态读回。

文档 ID：`android:member:com.netumscan.printersdk.StatusQueryConfig:b02cb9ffeefa8216`

JVM descriptor：`I`

<a id="api-61205a74675bd4d9f2d1"></a>

### public final int queryTimeoutMs

本次请求／响应事务的超时预算。

```java
public final int queryTimeoutMs;
```

必须大于 0，单位毫秒；与 queryStatus 的排队及操作总预算分别设置，不能据此绕过总预算。

文档 ID：`android:member:com.netumscan.printersdk.StatusQueryConfig:ee80c97565d158f3`

JVM descriptor：`I`

<a id="api-f34b5e25eec42fda666e"></a>

### public static final int CPCL = 1

CPCL 协议标识。

```java
public static final int CPCL = 1;
```

固定公共常量；仅标识协议，不保证目标固件已验证。

文档 ID：`android:member:com.netumscan.printersdk.StatusQueryConfig:bc25745e438ddbc7`

JVM descriptor：`I`

<a id="api-885a4db7b2a8d2745ec9"></a>

### public static final int ESC_POS = 2

ESC/POS 协议标识。

```java
public static final int ESC_POS = 2;
```

固定公共常量；仅标识协议，不保证目标固件已验证。

文档 ID：`android:member:com.netumscan.printersdk.StatusQueryConfig:282da3ae891951ec`

JVM descriptor：`I`

<a id="api-346a10d907279a636fde"></a>

## StatusResult

查询得到的硬件状态值。

```java
public final class com.netumscan.printersdk.StatusResult
```

knownConditions指明哪些状态位已知，activeConditions只在相应已知位内有意义；不得把未知位解释为正常，也不能从状态查询推断某次任务已打印完成。

文档 ID：`android:type:com.netumscan.printersdk.StatusResult`

<a id="api-6bd4c7e2dcd974871acc"></a>

### public final int activeConditions

已激活的硬件条件位。

```java
public final int activeConditions;
```

与knownConditions组合读取，不能单独将零值解释为所有硬件条件正常。

文档 ID：`android:member:com.netumscan.printersdk.StatusResult:7399537314a382a4`

JVM descriptor：`I`

<a id="api-935cf315825864fc4d7f"></a>

### public final int evidence

状态值的证据类型。

```java
public final int evidence;
```

遵循状态查询契约解释；不代表指定打印请求已设备确认完成。

文档 ID：`android:member:com.netumscan.printersdk.StatusResult:958eb37813714ec1`

JVM descriptor：`I`

<a id="api-1a0ec13fb92024dbae8a"></a>

### public final int knownConditions

硬件状态中已知的条件位。

```java
public final int knownConditions;
```

只有对应位为已知时，才可解释activeConditions中的该状态；未知不是正常。

文档 ID：`android:member:com.netumscan.printersdk.StatusResult:ca1edcaa356737e1`

JVM descriptor：`I`

<a id="api-7db2a2673b7578c3377e"></a>

### public static final int COVER_OPEN = 2

开盖条件位。

```java
public static final int COVER_OPEN = 2;
```

该常量的精确名称及编译值由公开基线固定；使用与本类型匹配的字段或参数，能力是否受支持以profile描述和请求校验为准。

文档 ID：`android:member:com.netumscan.printersdk.StatusResult:63e53ba840019830`

JVM descriptor：`I`

<a id="api-b98a0e5b3700e2a0af4f"></a>

### public static final int CUTTER_ERROR = 8

切刀错误条件位。

```java
public static final int CUTTER_ERROR = 8;
```

该常量的精确名称及编译值由公开基线固定；使用与本类型匹配的字段或参数，能力是否受支持以profile描述和请求校验为准。

文档 ID：`android:member:com.netumscan.printersdk.StatusResult:637b2910ef015ada`

JVM descriptor：`I`

<a id="api-b7448e50b83e77c0c944"></a>

### public static final int PAPER_NEAR_END = 4

纸张将尽条件位。

```java
public static final int PAPER_NEAR_END = 4;
```

该常量的精确名称及编译值由公开基线固定；使用与本类型匹配的字段或参数，能力是否受支持以profile描述和请求校验为准。

文档 ID：`android:member:com.netumscan.printersdk.StatusResult:bf16c2c3cb59609d`

JVM descriptor：`I`

<a id="api-38d1b817317c7817d342"></a>

### public static final int PAPER_OUT = 1

缺纸条件位。

```java
public static final int PAPER_OUT = 1;
```

该常量的精确名称及编译值由公开基线固定；使用与本类型匹配的字段或参数，能力是否受支持以profile描述和请求校验为准。

文档 ID：`android:member:com.netumscan.printersdk.StatusResult:3f80da847c593bc9`

JVM descriptor：`I`
