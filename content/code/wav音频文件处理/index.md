---
title: "wav音频文件处理"
subtitle: "wav音频文件数据结构初识"
description: ""
date: 2025-07-25T10:44:37+08:00
image: ""
tags: [python, wav]
categories: []
draft: false
---

## Zero

工作中需要使用 wav 文件通过 websocket 模拟流式音频发送文件内容，于是恶补了一下 wav 音频文件的数据结构以及各种相关知识点。

## First - wav 的各种数据结构和参数

### wav 格式简介

wav（Waveform Audio File Format）是一种无损的音频文件格式，由微软与 IBM 在 1991 年开发。其多应用于专业的音频制作（录音室，音乐制作），音频编辑，Windows 系统音效，音频分析等领域。

- 基本特征：
  - 无损压缩： 文件通常不进行压缩，保持原始音频数据的完整性
  - 高质量： 由于不压缩音频损失很小
  - 文件较大： 相较于 MP3 等压缩格式文件体积较大
  - 编码格式： 采用 PCM （脉冲编码调制）是最常见的编码方式

### wav 常用术语

wav 音频文件的实际引用过程中，我们最常接触到的会是如下几个文件参数术语：

- 采样率（Sample Rate）每秒采集音频信号的次数，单位 Hz 赫兹
- 帧率（Frame Rate）每秒处理的帧的数量，单位 fps（frames per second）
- 位深度（Bit Depth / Sample Width）每个采样点使用多少二进制位标识，决定音频的动态范围，常见值：
  - 8bit： 256 个量化级别，动态范围 48dB
  - 16bit：65,536 个量化级别，动态范围 96dB（CD 标准）
  - 24bit：16,777,216 个量化级别，动态范围 144dB
  - 32bit：4,294,967,296 个量化级别，动态范围 192dB
- 声道数（Channels）同时播放的独立的音频流数量，常见声道：
  - 单声道（Mono）：一个声道
  - 双声道，立体声（Stereo）：两个声道（左声道+右声道）
  - 环绕声：5.1,7.1 等多声道系统
- 帧（Frame）：包含所有声道的一个采样周期数据的单位
  - 16 位的立体声：1 帧 = 2 声道 x 2 字节 = 4 字节
  - 24 位的单声道：1 帧 = 1 声道 x 3 字节 = 3 字节
- 比特率（Bit Rate）每秒传输或处理的比特数量，单位是 bps（bits per second）

### 术语单位之间的关系

仅仅从上述罗列的数据中理解很可能还是有点不够清晰，以我自己为例之前一直对这些术语有一知半解但是完全无法理清楚它们之间的关系，接下来我将尝试用通俗一点的说法去拆解这些术语之间的关系。

- 任何一段 wav 音频文件，都是由**帧（frame）**这个基本单位构成的，一个**帧（frame）** 包含采集这一帧瞬间所有的 **声道（channel）** 的音频数据，这些数据在存储结构上线性排列，大多数情况下为小端存储。
- 一个音频 **帧（frame）** 内一个 **声道（channel）** 所占用的数据大小即位深，因此一个音频 **帧（frame）** 的大小即等于 **声道数（channels）** x **位深（bit depth）**，即一个 8bit 位深双声道音频 1 帧所占的空间是 8 bit \* 2 = 16 bit = 2 byte（字节）
- **采样率（sample rate）** 是指的每秒采集的音频信号的次数，一次采集的数据即为 1 **帧（frame）**，8kHz **采样率（sample rate）** 的 wav 音频在 1s 的时间内即采样 8000 **帧（frame）**，所以单帧音频的时长就是 1s 除以 **采样率（sample rate）** 也就是 1/8000s 。
- **帧率（frame rate）** 在音频场景下多数时候可以直接理解为一秒内处理 **帧（frame）** 的数量，因此在 wav 音频格式下**帧率（frame rate）** 即等于 **采样率（sample rate）**。
- **比特率（bit rate）** 是数据按照计算机字节计算一秒内处理的数据字节数，根据上述关系讲解我们可以简单计算得到一个 8000 Hz 采样率 8 bit 位深的双声道音频其比特率为： 8000 \* 2 \* 8 = 128,000 bps

> **不过需要注意的是，以上所有的计算方式都是以 wav pcm 编码下的计算逻辑，其他音频格式例如 MP3 和 AAC 都与之有着不小的差异，其采样率，帧等数据之间的关系并不会同 wav 格式一致，后面如果有机会会单独开文章详解。**

## Second - 使用 Python 解析 wav 文件

python 提供了内置的 wave 标准库用以支持 wave 文件的读取写入等操作，本文讨论的代码以 wave 库为基础。

### 获取 wav 文件的各种基本参数

```python
import wave

with wave.open("test.wav", "rb") as wav_file:
    n_channels = wav_file.getnchannels()      # 声道数
    n_frames = wav_file.getnframes()          # 总的帧数
    sample_width = wav_file.getsamplewidth() # 位宽 / 位深
    frame_rate = wav_file.getframerate()     # 帧率 / 采样率
    n_channels, sampwidth, framerate, n_frames = wav_file.getparams()[:4]
```

### 读取 wav 音频数据

1. 读取所有帧数据

```python
import wave

with wave.open("audio.wav", "rb") as wav_file:
    # 获取音频参数
    n_channels = wav_file.getnchannels()
    sample_width = wav_file.getsampwidth()
    sample_rate = wav_file.getframerate()
    n_frames = wav_file.getnframes()

    # 读取所有帧数据
    frames_data = wav_file.readframes(n_frames)
```

2. 逐帧读取

```python
import wave

with wave.open("audio.wav", "rb") as wav_file:
    # 逐帧读取
    while True:
        frame_data = wav_file.readframes(1)  # 读取1帧
        if not frame_data:
            break
        # 处理单帧数据
        process_frame(frame_data)
```

3. 流式读取

```python
import wave
import struct

with wave.open("audio.wav", "rb") as wav_file:
    sample_width = wav_file.getsampwidth()
    n_channels = wav_file.getnchannels()

    # 每次读取一小块数据
    chunk_frames = 1000
    while True:
        frames_data = wav_file.readframes(chunk_frames)
        if not frames_data:
            break

        # 将字节数据转换为数值
        if sample_width == 2:  # 16位
            samples = struct.unpack(f'<{len(frames_data)//2}h', frames_data)
        elif sample_width == 4:  # 32位
            samples = struct.unpack(f'<{len(frames_data)//4}i', frames_data)
```

### 分离音频声道

```python
import wave
import struct
import numpy as np

def separate_channels(input_file, output_prefix="channel"):
    """
    分离音频文件的声道

    Args:
        input_file: 输入音频文件路径
        output_prefix: 输出文件前缀
    """
    with wave.open(input_file, "rb") as wav_file:
        # 获取音频参数
        n_channels = wav_file.getnchannels()
        sample_width = wav_file.getsampwidth()
        sample_rate = wav_file.getframerate()
        n_frames = wav_file.getnframes()

        print(f"声道数: {n_channels}")
        print(f"采样率: {sample_rate} Hz")
        print(f"位深: {sample_width * 8} bit")
        print(f"总帧数: {n_frames}")

        # 读取所有音频数据
        frames_data = wav_file.readframes(n_frames)

        # 将字节数据转换为数值数组
        if sample_width == 2:  # 16位
            samples = struct.unpack(f'<{len(frames_data)//2}h', frames_data)
        elif sample_width == 4:  # 32位
            samples = struct.unpack(f'<{len(frames_data)//4}i', frames_data)
        else:
            raise ValueError(f"不支持的位深: {sample_width * 8} bit")

        # 重塑数组为 (帧数, 声道数)
        samples_array = np.array(samples).reshape(-1, n_channels)

        # 分离每个声道
        for channel_idx in range(n_channels):
            channel_data = samples_array[:, channel_idx]

            # 创建输出文件名
            output_file = f"{output_prefix}_{channel_idx + 1}.wav"

            # 写入单声道文件
            with wave.open(output_file, "wb") as output_wav:
                output_wav.setnchannels(1)  # 单声道
                output_wav.setsampwidth(sample_width)  # 设置位深
                output_wav.setframerate(sample_rate)  # 设置采样率

                # 将数据转换回字节格式
                if sample_width == 2:
                    channel_bytes = struct.pack(f'<{len(channel_data)}h', *channel_data)
                elif sample_width == 4:
                    channel_bytes = struct.pack(f'<{len(channel_data)}i', *channel_data)

                output_wav.writeframes(channel_bytes)

            print(f"声道 {channel_idx + 1} 已保存到: {output_file}")

# 使用示例
if __name__ == "__main__":
    # 分离立体声音频的左右声道
    separate_channels("stereo_audio.wav", "separated")

    # 结果会生成:
    # - separated_1.wav (左声道)
    # - separated_2.wav (右声道)
```

### 更高效的流式声道分离

```python
import wave
import struct

def separate_channels_streaming(input_file, output_prefix="channel"):
    """
    流式分离音频声道（适用于大文件）
    """
    with wave.open(input_file, "rb") as wav_file:
        n_channels = wav_file.getnchannels()
        sample_width = wav_file.getsampwidth()
        sample_rate = wav_file.getframerate()

        # 创建输出文件
        output_files = []
        for i in range(n_channels):
            output_file = f"{output_prefix}_{i + 1}.wav"
            output_wav = wave.open(output_file, "wb")
            output_wav.setnchannels(1)
            output_wav.setsampwidth(sample_width)
            output_wav.setframerate(sample_rate)
            output_files.append(output_wav)

        # 流式读取和处理
        chunk_frames = 1000  # 每次处理1000帧
        while True:
            frames_data = wav_file.readframes(chunk_frames)
            if not frames_data:
                break

            # 解析当前块的数据
            if sample_width == 2:
                samples = struct.unpack(f'<{len(frames_data)//2}h', frames_data)
            elif sample_width == 4:
                samples = struct.unpack(f'<{len(frames_data)//4}i', frames_data)

            # 分离声道
            for channel_idx in range(n_channels):
                channel_samples = samples[channel_idx::n_channels]

                # 转换回字节并写入对应文件
                if sample_width == 2:
                    channel_bytes = struct.pack(f'<{len(channel_samples)}h', *channel_samples)
                elif sample_width == 4:
                    channel_bytes = struct.pack(f'<{len(channel_samples)}i', *channel_samples)

                output_files[channel_idx].writeframes(channel_bytes)

        # 关闭所有输出文件
        for output_file in output_files:
            output_file.close()

        print(f"声道分离完成，生成了 {n_channels} 个单声道文件")
```
