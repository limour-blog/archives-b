---
title: 【AIGC:图】Stable Diffusion 试用【汉服国风】
tags: []
id: '2147'
categories:
  - - uncategorized
date: 2023-02-17 10:59:21
---

![](https://img.limour.top/archives_2023/2023/02/17/63ef49707cf02.webp)

最终效果图

## 安装Web-UI

[【AI绘画】启动器正式发布！一键启动/修复/更新/模型下载管理全支持！](https://www.bilibili.com/video/BV1ne4y1V7QU/)

安装[秋葉aaaki](https://space.bilibili.com/12566101)大佬的启动器和整合包。[镜像和基本常识点此](https://od.limour.top/ai/SD)。并更新到最新版

启动器-高级选项-自定义参数里填入 `--deepdanbooru --always-batch-cond-uncond`

## 安装相关模型和LoRA

[Stable Diffusion 照骗级国风 个人制做的写实模型+国风汉服少女lora模型](https://www.bilibili.com/read/cv21493779)

*   SD基本模型：[dcy/AsiaFacemix](https://huggingface.co/dcy/AsiaFacemix/blob/main/AsiaFacemix-pruned-fp16fix.safetensors)
*   Embedding：[gsdf/EasyNegative](https://huggingface.co/datasets/gsdf/EasyNegative)
*   VAE：[vae-ft-mse-840000-ema-pruned.ckpt](https://huggingface.co/stabilityai/sd-vae-ft-mse-original/tree/main) 下载后改名 .vae.ckpt 进 Web-UI的设置 里开启
*   LoRA：[lora-hanfugirl-v1-5.safetensors](https://huggingface.co/dcy/AsiaFacemix/blob/main/lora-hanfugirl-v1-5.safetensors)

## 基本参数

### 正面prompt

```txt
solo,illustration.media of a hanfugirl,hanfu,perfect skin,make happy expressions,gorgeous,pure,beautyfull detailed face and eyes,large breasts,beautyfull intricacy clothing decorative pattern details,arms behind back, 

colorful,clear sharp focus,instagram most viewed,

official wallpaper, official art,

volumetric lighting,soft lights,cinematic lighting,cinematic effects,

wallpaper,Megapixel,highres,Intricate details,ultra detailed,8k,

masterpiece, best quality

 <lora:lora-hanfugirl-v1-5:0.8>
```

### 负面prompt

```txt
 EasyNegative,bad face, fused face, poorly drawn face, cloned face, big face, long face, dirty face, bad anatomy, liquid body, anatomical nonsense, morbid, mutilated, malformed, ugly, deformed, uncoordinated body, unnatural body, strong girl, obesity, big muscles, (long body: 1.3), (mutation), adult, bad hands, fused hand, missing hand, malformed hands, (((poorly drawn hands))), more than 1 left hand, more than 1 right hand, (mutated hands and fingers: 1.5),missing fingers, fused fingers, one hand with more than 5 fingers, one hand with less than 5 fingers, fused digit, missing digit, (((bad digit))), (((liquid digit))), (((extra-long digit))),
```

### 其他参数

*   面部修复 √
*   随机种子：114514

## 其他图

![](https://img.limour.top/archives_2023/2023/02/17/63ef5dd46423d.webp)

```txt
solo,illustration.media of a hanfugirl,hanfu,perfect skin,make happy expressions,gorgeous,pure,beautyfull detailed face and eyes,large breasts,beautyfull intricacy clothing decorative pattern details,arms behind back,

colorful,clear sharp focus,instagram most viewed,

official wallpaper, official art,

volumetric lighting,soft lights,cinematic lighting,cinematic effects,

wallpaper,Megapixel,highres,Intricate details,ultra detailed,8k,

masterpiece, best quality,
<lora:lora-hanfugirl-v1-5:0.7>,
Negative prompt: EasyNegative,
one hand with more than 5 fingers, one hand with less than 5 fingers,
bad face, fused face, poorly drawn face, cloned face, big face, long face, dirty face, bad anatomy, liquid body, anatomical nonsense, morbid, mutilated, malformed, ugly, deformed, uncoordinated body, unnatural body, strong girl, obesity, big muscles, (long body: 1.3), (mutation), adult, bad hands, fused hand, missing hand, malformed hands, (((poorly drawn hands))), more than 1 left hand, more than 1 right hand, (mutated hands and fingers: 1.5),missing fingers, fused fingers, one hand with more than 5 fingers, one hand with less than 5 fingers, fused digit, missing digit, (((bad digit))), (((liquid digit))), (((extra-long digit))),
Steps: 20, Sampler: Euler a, CFG scale: 7, Seed: 114514, Size: 392x512, Model hash: 073f540cd6, Clip skip: 2, ENSD: 31337, ControlNet Enabled: True, ControlNet Module: none, ControlNet Model: control_openpose-fp16 [9ca67cc5], ControlNet Weight: 0.8



Used embeddings: EasyNegative [119b]

Time taken: 1m 6.24sTorch active/reserved: 4909/5008 MiB, Sys VRAM: 6144/6144 MiB (100.0%)
```