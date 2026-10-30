# MEDIA\_LITE<a name="EN-US_TOPIC_0000001126988465"></a>

-   [Introduction](#section11660541593)
-   [Directory Structure](#section1829614156212)
-   [Usage](#section1648194512427)
-   [Constraints](#section161941989596)
-   [Changes in This Release](#section_changes_en)
-   [Repositories Involved](#section105062051111614)

## Introduction<a name="section11660541593"></a>

This repository provides APIs for media-related capabilities such as recording, playback, parsing, and decoding, and the engine capability for media recording and playback.

**Figure  1**  Position of this repository \(in the dotted blue box\) in the multimedia subsystem<a name="fig1737281407"></a>  
![](figures/position-of-this-repository-(in-the-dotted-blue-box)-in-the-multimedia-subsystem.png "position-of-this-repository-(in-the-dotted-blue-box)-in-the-multimedia-subsystem")

## Directory Structure<a name="section1829614156212"></a>

```
/foundation/multimedia/media_lite
├── frameworks # Framework code
├── interfaces # APIs
│   └── kits   # External APIs
└── test       # Test code
```

## Usage<a name="section1648194512427"></a>

Build a single repository in the root directory.

```
# Select the development board.
hb set  
# Build this repository.
hb build media_lite
```

## Constraints<a name="section161941989596"></a>

C++ 11 or later

# Changes in This Release<a name="section_changes_en"></a>

This repository is adapted for the HiSpark AiFly (Hi3403V100) development board. For clarity, the changes are described in two parts: **Added Features** and **Enhanced Existing Features**.

## Added Features

1. Added board-level configuration
    -   `config.gni` adds the `media_lite_camera_config_file`, `media_lite_camera_config_file_01` and `media_lite_display_library` arguments and centralizes board mappings: HiSpark AiFly uses `cameradev_os08a20_928.ini` and `display_layer`, while existing boards such as Hi3516DV300/Hi3518EV300 keep their own settings. Adding a new board only requires a new mapping in `config.gni` instead of modifying build targets.
    -   Added `services/cameradev_os08a20_928.ini` (OS08A20 camera sensor configuration), `services/media_config.ini` and `services/codec_config.ini`; `services/BUILD.gn` adds the `camera_config`, `camera_config01`, `media_config` and `codec_config` copy targets deployed to `data/`.

2. Added the media service component description
    -   Added `services/bundle.json` (the `media_service` component) that registers the service-side build targets and dependencies.

3. Added distributed hardware service integration
    -   `services/media_main.cpp` registers `KvStoreDataServer` and `DistributedHardwareServer` under the `ENABLE_DH` macro to support the distributed hardware link.

4. Added HDMI hot-plug support
    -   `services/player_lite/BUILD.gn` adds the `enable_player_hdmi_hot_swap` switch (mapped to the `ENABLE_PLAYER_HDMI_HOT_SWAP` macro); the audio sink (`player_audio_sink`) adds HDMI connection detection with a query counter (`hdmiConnected_`/`hdmiQueryCnt_`) to support hot switching of the output device.

5. Added dual codec interface adaptation (`MEDIA_INTERFACE_V1_0`)
    -   The decoder (`decoder`), audio/video sinks (`player_audio_sink`/`player_video_sink`) and demuxer (`player_demux`) provide both the new `InputInfo`/`OutputInfo` APIs and the original `CodecBuffer` APIs under `MEDIA_INTERFACE_V1_0`, staying compatible with the new codec HDI.

6. Added the playback-control common module
    -   Added `liteplayer_comm.cpp/h`, which encapsulates the format-to-MIME mapping table, timed condition-variable waiting (`CondTimeWait`) and video resolution query logic.

7. Added playback test samples
    -   Added `test/demo`: an AAC player sample, a video player sample, a Surface sample and the `plugin_sample` plugin sample; `demo_group` is registered in the `test` section of `bundle.json`.

## Enhanced Existing Features

1. The existing display library linking is enhanced
    -   Original functionality: `frameworks/player_lite` and `services/player_lite` always linked `-lhdi_videodisplayer`.
    -   Enhancement: the linked library is now selected by `media_lite_display_library` (`-l${media_lite_display_library}`); `frameworks/player_lite/BUILD.gn` allows product-side override via `custom_display_ld_flags`, and a dependency on `${ohos_board_adapter_dir}/display:display_layer` is added.

2. The existing playback core flow is enhanced
    -   Original functionality: `liteplayer.cpp` concentrated event handling, data scheduling and codec packet pushing in one place, and `liteplayer.h` exposed limited capabilities.
    -   Enhancement: `liteplayer.cpp` is refactored and common logic is split into `liteplayer_comm.cpp`; `PlayerControl` adds `IsPaused` (atomic `isPaused_`), `OnAudioEndOfStream`, `SetLayerPriority` and `DoPlayFromPrepared`; audio/video packet pushing is split into `PushPacketToADecoderInner`/`PushPacketToVDecoderInner` with a consecutive-audio-full counter (`continuousAudFull_`), improving pause-state detection and scheduling robustness.

3. The existing audio output initialization is enhanced
    -   Original functionality: audio adapter loading and render creation were implemented inline.
    -   Enhancement: split into `CheckAudioAdapter`, `LoadAudioAdapter`, `InitAudioSampleAttributes` and `CreateAudioRender`, making the flow clearer and easier to adapt per board.

4. The existing demuxer file-info retrieval is enhanced
    -   Original functionality: `PlayerDemuxer::GetFileInfo` assembled `FileInfo` manually, and the struct field was misspelled as `stSteamResolution`.
    -   Enhancement: `FormatDemuxerGetFileInfo` is used directly under `MEDIA_INTERFACE_V1_0`; the spelling is corrected to `stStreamResolution`; the duplicate `StreamResolution`/`FormatFileInfo` definitions in `hi_demuxer.h` are removed and now come from the common header in `media_utils_lite`.

5. The existing decoder API signature is enhanced
    -   Original functionality: `SetPortBufferMode(DirectionType direct, AllocateBufferMode mode, BufferType type)` did not match the new codec HDI definition.
    -   Enhancement: corrected to `SetPortBufferMode(DirectionType type, BufferMode mode)`.

6. The existing component description and build configuration are enhanced
    -   Original functionality: `bundle.json` placed test targets in the main `target_cpu` build section.
    -   Enhancement: test targets are moved to the `test` section with `demo_group` added; `services/BUILD.gn` adds the `recorder_lite` include path and depends on `recorder_impl`.

7. The existing open-source compliance configuration is enhanced
    -   Original functionality: `OAT.xml` configures open-source compliance scanning.
    -   Enhancement: policyitems are added, registering the HiSilicon and RKH copyrights.

## Repositories Involved<a name="section105062051111614"></a>

[Multimedia subsystem](https://gitee.com/openharmony/docs/blob/master/en/readme/multimedia.md)

[camera\_lite](https://gitee.com/openharmony/multimedia_camera_lite)

[camera\_sample\_lite](https://gitee.com/openharmony/applications_sample_camera)

[media\_utils\_lite](https://gitee.com/openharmony/multimedia_utils_lite)

[audio\_lite](https://gitee.com/openharmony/multimedia_audio_lite)

[media\_lite](https://gitee.com/openharmony/multimedia_media_lite)

