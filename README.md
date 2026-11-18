# MEDIA\_LITE<a name="EN-US_TOPIC_0000001126988465"></a>

-   [Introduction](#section11660541593)
-   [Directory Structure](#section1829614156212)
-   [Usage](#section1648194512427)
-   [Constraints](#section161941989596)
-   [Repositories Involved](#section105062051111614)

## Introduction<a name="section11660541593"></a>

This repository provides APIs for media-related capabilities such as recording, playback, parsing, and decoding, and the engine capability for media recording and playback. The following aspects are covered on top of these capabilities:

- Media service component description: `services/bundle.json` registers the media service component and complements the service-side build targets and dependencies.

- HDMI hot-plug support: a hot-plug build switch and a connection detection counter allow the output device to switch between HDMI and the speaker.

- Dual codec interface adaptation: the decoder and the audio/video sinks provide both the new `InputInfo`/`OutputInfo` APIs and the original `CodecBuffer` APIs under `MEDIA_INTERFACE_V1_0`, staying compatible with the new codec HDI.

- Common playback-control module: `liteplayer_comm.cpp/h` encapsulates format-to-MIME mapping, timed condition-variable waiting, and video resolution query logic to reduce duplication.

- Playback test samples: `test/demo` provides AAC, video, Surface, and plugin playback samples to help verify the playback chain.

- Display library linking and build configuration: display library linking becomes configurable and the `frameworks` side allows product-specific link flags; the build config adds recorder include paths and dependencies and moves test targets into a dedicated test section.

- Core playback flow: `liteplayer` is refactored with common logic split out, improving pause-state detection, separated audio/video packet pushing, split audio output initialization, and buffer-full back-off handling for more robust scheduling.

- Demuxer and decoder interfaces: file-info retrieval uses the unified interface with corrected field spelling, and the decoder `SetPortBufferMode` signature is aligned with the new definition.

**Figure  1**  Position of this repository \(in the dotted blue box\) in the multimedia subsystem<a name="fig1737281407"></a>  
![](figures/position-of-this-repository-(in-the-dotted-blue-box)-in-the-multimedia-subsystem.png "position-of-this-repository-(in-the-dotted-blue-box)-in-the-multimedia-subsystem")

## Directory Structure<a name="section1829614156212"></a>

```
/foundation/multimedia/media_lite
├── frameworks # Framework code
├── interfaces # APIs
│   └── kits   # External APIs
├── services   # Service implementation (player, recorder, media service, and board configuration)
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

## Repositories Involved<a name="section105062051111614"></a>

[Multimedia subsystem](https://gitee.com/openharmony/docs/blob/master/en/readme/multimedia.md)

[camera\_lite](https://gitee.com/openharmony/multimedia_camera_lite)

[camera\_sample\_lite](https://gitee.com/openharmony/applications_sample_camera)

[media\_utils\_lite](https://gitee.com/openharmony/multimedia_utils_lite)

[audio\_lite](https://gitee.com/openharmony/multimedia_audio_lite)

[media\_lite](https://gitee.com/openharmony/multimedia_media_lite)
