---
title: 【AIGC:图】LoRA模型风格混合
tags: []
id: '2148'
categories:
  - - uncategorized
date: 2023-02-17 16:36:01
---

![](https://img-cdn.limour.top/i/2023/02/18/63efaab8c5d13.jpg)

Arona+naifu+hanfu

```txt
full body, <lora:aronaBlueArchive_v1:1>, arona, 1girl, white hairband, bow hairband, halo, short hair, single braid, blue eyes, two-tone hair, multicolored hair, blue hair, pink hair, masterpiece, best quality, hanfugirl,hanfu,<lora:lora-hanfugirl-v1-5:1>, ((ultra-detailed)),((an extremely delicate and beautiful)),(beautiful detailed eyes),((disheveled hair))
Negative prompt: EasyNegative, lowres, bad anatomy, bad hands, text, error, missing fingers, extra digit, fewer digits, cropped, worst quality, low quality, normal quality, jpeg artifacts, signature, watermark, username, blurry
Steps: 20, Sampler: Euler a, CFG scale: 7, Seed: 2039715882, Size: 768x1024, Model hash: 89d59c3dde, ENSD: 31337

Used embeddings: EasyNegative [119b]

Time taken: 1m 20.61sTorch active/reserved: 4898/5302 MiB, Sys VRAM: 6144/6144 MiB (100.0%)
```

这张图以NovelAI的模型为底，加上Arona的LoRA和hanfu的LoRA混合而成。

![](https://img-cdn.limour.top/i/2023/02/18/63efaba9118b2.png)

我将这些拆成了四个Styles：（以下为styles.csv中对应的内容）

*   Arona按其LoRA的官方触发词，设定如下：`Arona,", arona, 1girl, white hairband, bow hairband, halo, short hair, single braid, blue eyes, two-tone hair, multicolored hair, blue hair, pink hair"`,
*   naifu为NovelAI的起手式加上EasyNegative的embeddings：`naifu,"masterpiece, best quality","EasyNegative, lowres, bad anatomy, bad hands, text, error, missing fingers, extra digit, fewer digits, cropped, worst quality, low quality, normal quality, jpeg artifacts, signature, watermark, username, blurry"`
*   汉服为汉服LoRA的官方触发词：`汉服,"hanfugirl,hanfu,",`
*   美化：`美化,"((ultra-detailed)),((an extremely delicate and beautiful)),(beautiful detailed eyes),((disheveled hair))",`

加上这四个Styles后，以 `full body` 为prompt，即可生成风格融合的全身图。