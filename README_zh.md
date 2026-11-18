# 媒体子系统MEDIA\_LITE组件<a name="ZH-CN_TOPIC_0000001126988465"></a>

-   [简介](#section11660541593)
-   [目录](#section1829614156212)
-   [使用说明](#section1648194512427)
-   [约束](#section161941989596)
-   [相关仓](#section105062051111614)

## 简介<a name="section11660541593"></a>

提供播放、录制、解析、解码等接口能力，并提供媒体播放录制引擎服务化能力。在上述能力基础上，主要涉及以下方面：

- 媒体服务组件描述：通过 `services/bundle.json` 登记媒体服务组件，补充服务侧的构建目标与依赖。

- HDMI 热插拔：引入热插拔编译开关与连接状态检测计数，支持输出设备在 HDMI 与扬声器之间热切换。

- 编解码器双接口适配：解码器以及音频、视频输出在 `MEDIA_INTERFACE_V1_0` 下同时提供 `InputInfo`/`OutputInfo` 新接口与原 `CodecBuffer` 接口，兼容新版编解码 HDI。

- 播放控制公共模块：`liteplayer_comm.cpp/h` 封装格式与 MIME 映射、条件变量定时等待、视频分辨率查询等公共逻辑，减少重复实现。

- 播放测试示例：`test/demo` 提供 AAC 播放、视频播放、Surface 与插件播放示例，便于验证播放链路。

- 显示库链接与构建配置：显示库链接方式改为可配置，`frameworks` 侧支持产品自定义链接参数；构建配置补充录制相关头文件路径与依赖，测试目标归入独立的 test 段。

- 播放核心流程：`liteplayer` 重构并拆分公共实现，完善暂停态判定、音视频包推送拆分、音频输出初始化拆分及满帧退避处理，提升调度健壮性。

- 解封装与解码器接口：解封装信息获取改用统一接口并修正字段拼写，解码器 `SetPortBufferMode` 接口签名同步为新版定义。

**图 1**  媒体子系统架构（蓝色虚线框是media\_lite）<a name="fig1737281407"></a>  
![](figures/媒体子系统架构（蓝色虚线框是media_lite）.png "媒体子系统架构（蓝色虚线框是media_lite）")

## 目录<a name="section1829614156212"></a>

```
/foundation/multimedia/media_lite
├── frameworks # 框架代码
├── interfaces # 接口
│   └── kits   # 对外接口
├── services   # 服务化实现（播放、录制、媒体服务及板级配置）
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

## 相关仓<a name="section105062051111614"></a>

[媒体子系统](https://gitee.com/openharmony/docs/blob/master/zh-cn/readme/%E5%AA%92%E4%BD%93%E5%AD%90%E7%B3%BB%E7%BB%9F.md)

[camera\_lite](https://gitee.com/openharmony/multimedia_camera_lite)

[camera\_sample\_lite](https://gitee.com/openharmony/applications_sample_camera)

[media\_utils\_lite](https://gitee.com/openharmony/multimedia_utils_lite)

[audio\_lite](https://gitee.com/openharmony/multimedia_audio_lite)

[media\_lite](https://gitee.com/openharmony/multimedia_media_lite)
