---
layout: post
comments: true
title:  "使用ffmpeg裁剪想要的帧"
date:   2026-02-03 21:33:26 +0800
tags: ffmpeg
lang: zh
---

`原创`

看到 [https://www.youtube.com/watch?v=zzHeG7l6FT8](https://www.youtube.com/watch?v=zzHeG7l6FT8) 《【東方ボーカルMV】東方言えるかな》里有非常多漂亮的角色（我全都要），觉得很适合拿来做壁纸用，但是又有一些没有画面的场景，可以剪掉来提升观感。

成品链接：
[https://steamcommunity.com/sharedfiles/filedetails/?id=3659280460](https://steamcommunity.com/sharedfiles/filedetails/?id=3659280460)

在线观看：
[https://www.bilibili.com/video/BV1HGFLz8E1H](https://www.bilibili.com/video/BV1HGFLz8E1H)

### 步骤：

1. 从youtube上下载视频和音频
2. 使用ffmpeg合并视频和音频
3. 使用ffmpeg导出所有帧为图片
4. 人工过一遍所有图片，把没有人物的场景删掉
5. 整理出来筛选后的图片帧序号并使用ffmpeg裁剪视频

### 从youtube上下载视频和音频

第一步这里使用  
[https://snapany.com/zh/youtube-1](https://snapany.com/zh/youtube-1)  
直接解析youtube地址拿到googlevideo.com域名的链接来下载。  
这里下载到的视频和音频是两个独立的文件。

### 使用ffmpeg合并视频和音频

让AI生成一个ffmpeg命令

```sh
ffmpeg -i videoplayback.mp4 -i videoplayback.m4a -c copy output.mp4
```

### 使用ffmpeg导出所有帧为图片

让AI生成一个ffmpeg命令

```sh
ffmpeg -i input.mp4 frame_%05d.png
```

这里我导出了之后复制一份，一份是原始的，一份用来人工筛选后删除不需要的，方便后续剪视频。

### 让AI整理出来筛选后的图片帧序号并使用ffmpeg裁剪视频

直接让AI写个python脚本

```python
import os
import re
import subprocess

def find_deleted_ranges(images_dir, images2_dir):
    all_frames = []
    for f in os.listdir(images2_dir):
        m = re.match(r"frame_(\d+)\.png", f)
        if m:
            all_frames.append(int(m.group(1)))
    all_frames.sort()

    keep_frames = []
    for f in os.listdir(images_dir):
        m = re.match(r"frame_(\d+)\.png", f)
        if m:
            keep_frames.append(int(m.group(1)))
    keep_frames.sort()

    keep_set = set(keep_frames)
    deleted = [n for n in all_frames if n not in keep_set]

    print("\n===== 帧统计 =====")
    print(f"原视频总帧数: {len(all_frames)}")
    print(f"保留帧数: {len(keep_frames)}")
    print(f"删除帧数: {len(deleted)}")

    if not deleted:
        return [], all_frames, keep_frames, deleted

    ranges = []
    start = deleted[0]
    prev = deleted[0]

    for n in deleted[1:]:
        if n == prev + 1:
            prev = n
        else:
            ranges.append((start, prev))
            start = n
            prev = n
    ranges.append((start, prev))

    print("\n===== 删除的连续帧区间 =====")
    for s, e in ranges:
        print(f"  - 从 {s} 到 {e} （共 {e - s + 1} 帧）")

    return ranges, all_frames, keep_frames, deleted


def build_video_select(ranges):
    parts = []
    for s, e in ranges:
        parts.append(f"between(n,{s-1},{e-1})")
    return "not(" + "+".join(parts) + ")"


def build_audio_select(ranges, fps):
    parts = []
    for s, e in ranges:
        t1 = (s - 1) / fps
        t2 = (e - 1) / fps
        parts.append(f"between(t,{t1},{t2})")
    return "not(" + "+".join(parts) + ")"


def get_fps(ffmpeg, input_video):
    cmd = [ffmpeg, "-i", input_video]
    p = subprocess.Popen(cmd, stderr=subprocess.PIPE, stdout=subprocess.PIPE, text=True)
    out, err = p.communicate()
    fps_match = re.search(r"(\d+(?:\.\d+)?) fps", err)
    fps = float(fps_match.group(1)) if fps_match else 30.0
    print("\n原视频帧率 =", fps)
    return fps


def get_frame_count(ffprobe, video_path):
    cmd = [
        ffprobe,
        "-v", "error",
        "-select_streams", "v:0",
        "-count_packets",
        "-show_entries", "stream=nb_read_packets",
        "-of", "csv=p=0",
        video_path
    ]
    result = subprocess.run(cmd, capture_output=True, text=True)
    try:
        return int(result.stdout.strip())
    except:
        return None


def main():
    ffmpeg = r".\ffmpeg-2026-01-29-git-c898ddb8fe-full_build\bin\ffmpeg.exe"
    ffprobe = r".\ffmpeg-2026-01-29-git-c898ddb8fe-full_build\bin\ffprobe.exe"

    input_video = "videoplayback.mp4"
    output_video = "video_cut.mp4"

    images_dir = "images" # 这个是删过帧只保留了人物画面的文件夹
    images2_dir = "images2" # 这个是原始的所有帧的文件夹

    ranges, all_frames, keep_frames, deleted = find_deleted_ranges(images_dir, images2_dir)
    if not ranges:
        return

    fps = get_fps(ffmpeg, input_video)

    video_select = build_video_select(ranges)
    audio_select = build_audio_select(ranges, fps)

    print("\n===== 视频过滤表达式 =====")
    print(video_select)

    print("\n===== 音频过滤表达式（按时间） =====")
    print(audio_select)

    cmd = [
        ffmpeg,
        "-i", input_video,
        "-vf", f"select='{video_select}',setpts=N/({fps}*TB)",
        "-af", f"aselect='{audio_select}',asetpts=N/SR/TB",
        "-c:v", "libx264",
        "-c:a", "aac",
        "-pix_fmt", "yuv420p",
        "-y",
        output_video
    ]

    print("\n===== 开始裁剪视频（含音频） =====")
    subprocess.run(cmd)
    print("\n完成！输出文件：", output_video)

    actual_frames = get_frame_count(ffprobe, output_video)

    print("\n===== 裁剪后帧数验证 =====")
    print(f"预期保留帧数: {len(keep_frames)}")
    print(f"实际视频帧数: {actual_frames}")

    if actual_frames == len(keep_frames):
        print("✔ 帧数完全一致，音视频同步裁剪成功")
    else:
        print("✘ 帧数不一致，请检查源数据")


if __name__ == "__main__":
    main()
```