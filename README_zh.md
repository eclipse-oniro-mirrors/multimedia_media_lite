# 媒体子系统MEDIA\_LITE组件<a name="ZH-CN_TOPIC_0000001126988465"></a>

-   [简介](#section11660541593)
-   [目录](#section1829614156212)
-   [使用说明](#section1648194512427)
-   [约束](#section161941989596)
-   [本次改动](#section_changes_zh)
-   [相关仓](#section105062051111614)

## 简介<a name="section11660541593"></a>

提供播放、录制、解析、解码等接口能力，并提供媒体播放录制引擎服务化能力。

**图 1**  媒体子系统架构（蓝色虚线框是media\_lite）<a name="fig1737281407"></a>  
![](figures/媒体子系统架构（蓝色虚线框是media_lite）.png "媒体子系统架构（蓝色虚线框是media_lite）")

## 目录<a name="section1829614156212"></a>

```
/foundation/multimedia/media_lite
├── frameworks # 框架代码
├── interfaces # 接口
│   └── kits   # 对外接口
└── test       # 测试代码
```

## 使用说明<a name="section1648194512427"></a>

单仓的编译构建，在根目录下进行单仓的构建和编译

```
# 开发板选择
hb set  
# 单仓构建和编译
hb build media_lite
```

## 约束<a name="section161941989596"></a>

C++11版本或以上

# 本次改动<a name="section_changes_zh"></a>

本仓针对 HiSpark AiFly（Hi3403V100）开发板进行功能适配。为便于区分，以下按“新增能力”与“原有功能增强”两部分说明。

## 新增能力

1. 新增板级配置化能力
    -   `config.gni` 新增 `media_lite_camera_config_file`、`media_lite_camera_config_file_01`、`media_lite_display_library` 参数，并集中维护板级映射：HiSpark AiFly 使用 `cameradev_os08a20_928.ini` 与 `display_layer`，Hi3516DV300/Hi3518EV300 等原有板型保持各自配置。新增板型时仅需在 `config.gni` 增加映射，不再修改构建目标。
    -   新增 `services/cameradev_os08a20_928.ini`（OS08A20 摄像头 Sensor 配置）、`services/media_config.ini` 与 `services/codec_config.ini`，`services/BUILD.gn` 新增 `camera_config`、`camera_config01`、`media_config`、`codec_config` 拷贝目标，部署到 `data/` 目录。

2. 新增媒体服务组件描述
    -   新增 `services/bundle.json`（`media_service` 组件），登记服务化构建目标与依赖。

3. 新增分布式硬件服务接入
    -   `services/media_main.cpp` 在 `ENABLE_DH` 宏下注册 `KvStoreDataServer` 与 `DistributedHardwareServer`，支撑分布式硬件链路。

4. 新增 HDMI 热插拔支持
    -   `services/player_lite/BUILD.gn` 新增 `enable_player_hdmi_hot_swap` 开关（映射 `ENABLE_PLAYER_HDMI_HOT_SWAP` 宏）；音频输出（`player_audio_sink`）新增 HDMI 连接检测与查询计数（`hdmiConnected_`/`hdmiQueryCnt_`），支持输出设备热切换。

5. 新增编解码器双接口适配（`MEDIA_INTERFACE_V1_0`）
    -   解码器（`decoder`）、音频/视频输出（`player_audio_sink`/`player_video_sink`）与解封装（`player_demux`）按 `MEDIA_INTERFACE_V1_0` 提供 `InputInfo`/`OutputInfo` 新接口与原 `CodecBuffer` 接口双路径，兼容新版编解码 HDI。

6. 新增播放控制公共模块
    -   新增 `liteplayer_comm.cpp/h`：封装格式与 MIME 映射表、条件变量定时等待（`CondTimeWait`）、视频分辨率查询等公共逻辑。

7. 新增播放测试示例
    -   新增 `test/demo`：AAC 播放示例、视频播放示例、Surface 示例与 `plugin_sample` 插件示例，并在 `bundle.json` 的 `test` 段登记 `demo_group`。

## 原有功能增强

1. 原有显示库链接方式增强
    -   原有功能：`frameworks/player_lite` 与 `services/player_lite` 固定链接 `-lhdi_videodisplayer`。
    -   本次增强：改为按 `media_lite_display_library` 选择链接库（`-l${media_lite_display_library}`），`frameworks/player_lite/BUILD.gn` 支持产品侧通过 `custom_display_ld_flags` 覆盖，并新增对 `${ohos_board_adapter_dir}/display:display_layer` 的依赖。

2. 原有播放核心流程增强
    -   原有功能：`liteplayer.cpp` 将事件处理、数据调度、编解码推送等逻辑集中实现，`liteplayer.h` 中相关能力有限。
    -   本次增强：重构 `liteplayer.cpp` 并拆分公共逻辑至 `liteplayer_comm.cpp`；`PlayerControl` 新增 `IsPaused`（原子变量 `isPaused_`）、`OnAudioEndOfStream`、`SetLayerPriority`、`DoPlayFromPrepared`，音/视频包推送拆分出 `PushPacketToADecoderInner`/`PushPacketToVDecoderInner`，新增音频连续满帧计数（`continuousAudFull_`），提升暂停态判定与调度健壮性。

3. 原有音频输出初始化流程增强
    -   原有功能：音频适配器加载与渲染创建逻辑内联。
    -   本次增强：拆分出 `CheckAudioAdapter`、`LoadAudioAdapter`、`InitAudioSampleAttributes`、`CreateAudioRender`，流程更清晰、便于板级差异适配。

4. 原有解封装信息获取增强
    -   原有功能：`PlayerDemuxer::GetFileInfo` 手工组装 `FileInfo`，且结构体字段拼写为 `stSteamResolution`。
    -   本次增强：`MEDIA_INTERFACE_V1_0` 下直接使用 `FormatDemuxerGetFileInfo`；修正拼写为 `stStreamResolution`；`hi_demuxer.h` 中与 `media_utils_lite` 重复的 `StreamResolution`/`FormatFileInfo` 定义移除，统一由公共头提供。

5. 原有解码器接口签名增强
    -   原有功能：`SetPortBufferMode(DirectionType direct, AllocateBufferMode mode, BufferType type)` 与新版编解码 HDI 定义不一致。
    -   本次增强：修正为 `SetPortBufferMode(DirectionType type, BufferMode mode)`。

6. 原有组件描述与构建配置增强
    -   原有功能：`bundle.json` 将测试目标置于 `target_cpu` 主构建段。
    -   本次增强：测试目标归入 `test` 段并新增 `demo_group`；`services/BUILD.gn` 补充 `recorder_lite` 头文件路径并依赖 `recorder_impl`。

7. 原有开源合规配置增强
    -   原有功能：`OAT.xml` 用于开源合规扫描配置。
    -   本次增强：新增 policyitem，登记 HiSilicon 与 RKH 版权声明。

## 相关仓<a name="section105062051111614"></a>

[媒体子系统](https://gitee.com/openharmony/docs/blob/master/zh-cn/readme/%E5%AA%92%E4%BD%93%E5%AD%90%E7%B3%BB%E7%BB%9F.md)

[camera\_lite](https://gitee.com/openharmony/multimedia_camera_lite)

[camera\_sample\_lite](https://gitee.com/openharmony/applications_sample_camera)

[media\_utils\_lite](https://gitee.com/openharmony/multimedia_utils_lite)

[audio\_lite](https://gitee.com/openharmony/multimedia_audio_lite)

[media\_lite](https://gitee.com/openharmony/multimedia_media_lite)

