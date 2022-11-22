---
title: "How to encode a video by all frames"
date: 2022-11-22T15:41:15+08:00
description: 如题所示
draft: false
hideToc: false
enableToc: true
enableTocContent: false
author: puzzled
authorEmoji: 🦄
tocFolding: false
tocPosition: inner
tocLevels: ["h2", "h3", "h4"]
tags:
- encode
series:
-
categories:
- python
image: images/feature1/4.jpg
---


## 获取一个视频的所有帧

解码一个视频有多种方式，这里分别用pyav和opencv做示例。

```python
"""
decoder.py
Destination: decode a video, get all frames
Some documents：
http://ffmpeg.org/documentation.html
"""
import os
import os.path as osp
import shutil
import time
import uuid
import av
import cv2


class Decoder(object):

    def __init__(self):
        super(Decoder, self).__init__()

    @staticmethod
    def av_decoder(path_to_video, frame_dir="frames/"):
        """
        PyAV document: https://pyav.org/docs/stable/index.html
        https://pyav.org/docs/stable/cookbook/numpy.html
        Returns:
        """
        # 解码获取视频帧并转换np数组
        container = av.open(path_to_video)
        container.streams.video[0].thread_type = "AUTO"

        np_frames = []
        for frame in container.decode(video=0):
            print(frame)
            array = frame.to_ndarray(format="bgr24")
            np_frames.append(array)
        container.close()
        print(len(np_frames))
        print(np_frames[0].shape)
        return np_frames

    @staticmethod
    def cv2_decoder(path_to_video, frame_dir="frames/"):
        """
        opencv-python document: https://docs.opencv.org/3.0-beta/doc/py_tutorials/py_tutorials.html
        https://docs.opencv.org/3.0-beta/doc/py_tutorials/py_gui/py_video_display/py_video_display.html
        Returns:
        """
        cap = cv2.VideoCapture(path_to_video)
        np_frames = []

        while cap.isOpened():
            ret, frame = cap.read()
            if not ret:
                break
            print('read a new frame')
            np_frames.append(frame)
        cap.release()
        print(len(np_frames))
        print(np_frames[0].shape)
        return np_frames
```

## 编码所有帧

同样地分别用上述两种方法来做。

```python
"""
encoder.py
Destination: encode a video, only with video stream
Some documents：
http://ffmpeg.org/documentation.html
"""
import os
import os.path as osp
import shutil
import time
import cv2
import numpy as np
import av
import uuid
from decoder import Decoder


class Encoder(object):
    def __init__(self):
        super(Encoder, self).__init__()
        self.fps = 25
        self.width = 1920
        self.height = 1080
        self.bit_rate = 8 * 1014 * 1024

    def av_encoder(self, frames, encode_video):
        # 编码一个视频返回
        container = av.open(encode_video, mode="w")
        stream = container.add_stream("mpeg4", rate=self.fps)
        container.streams.video[0].thread_type = "AUTO"
        stream.width = self.width
        stream.height = self.height
        stream.bit_rate = self.bit_rate
        stream.pix_fmt = "yuv420p"

        for frame in frames:
            frame = av.VideoFrame.from_ndarray(frame, format="bgr24")
            for packet in stream.encode(frame):
                container.mux(packet)

        # Flush stream
        for packet in stream.encode():
            container.mux(packet)

        # Close the file
        container.close()
        print(f"{encode_video} encode complete")

    def cv2_encoder(self, frames, encode_video):
        size = (self.width, self.height)
        fourcc = cv2.VideoWriter_fourcc(*'mp4v') 
        out = cv2.VideoWriter(encode_video, fourcc, self.fps, size)
        for frame in frames:
            out.write(frame)
        out.release()
        print(f"{encode_video} encode complete")

    def av_set_parameter(self, video):
        with av.open(video) as container:
            stream = container.streams.video[0]
            self.fps = stream.codec_context.framerate  # type :Fraction
            self.width = stream.codec_context.width
            self.height = stream.codec_context.height
            self.bit_rate = stream.bit_rate  # video的码率
            # all_bit_rate = container.bit_rate  # 视频总码率

    def cv2_set_parameter(self, video):
        cap = cv2.VideoCapture(video)
        self.fps = int(round(cap.get(cv2.CAP_PROP_FPS)))  # 帧率
        self.width = int(cap.get(cv2.CAP_PROP_FRAME_WIDTH))  # 分辨率-宽度
        self.height = int(cap.get(cv2.CAP_PROP_FRAME_HEIGHT))  # 分辨率-高度

        # frame_counter = int(cap.get(cv2.CAP_PROP_FRAME_COUNT))   # 总帧数
        # https://docs.opencv.org/2.4/modules/highgui/doc/reading_and_writing_images_and_video.html#videocapture-get
        # 2.4 can't find bit rate
        # https://docs.opencv.org/5.x/d4/d15/group__videoio__flags__base.html#gaeb8dd9c89c10a5c63c139bf7c4f5704d
        # find it, but my opencv version can't use it
        # self.bit_rate = int(cap.get(cv2.CAP_PROP_BITRATE))
        cap.release()
```



