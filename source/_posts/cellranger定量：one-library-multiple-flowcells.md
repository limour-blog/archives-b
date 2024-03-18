---
title: cellranger定量：One Library, Multiple Flowcells
tags: []
id: '2029'
categories:
  - - uncategorized
date: 2022-09-25 07:08:51
---

![](https://img.limour.top/archives_2023/2022/09/25/632fc9055a801.webp)

If you have a library which was sequenced across multiple flowcells, you can pool the reads from both sequencing runs. Follow the steps in [Running Multi-Library Samples](https://support.10xgenomics.com/single-cell-gene-expression/software/pipelines/2.0/advanced/multi-library-samples) to combine them in a single cellranger count run.

![](https://img.limour.top/archives_2023/2022/09/25/632fcc42859d5.webp)

## 重命名R1、R2

[My FASTQs are not named like any of the above examples](https://support.10xgenomics.com/single-cell-gene-expression/software/pipelines/2.0/using/fastq-input#wrongname). It is likely that you received files that were processed through a proprietary LIMS system, which employs its own naming conventions.

10x pipelines need files named in the `bcl2fastq` or `demux` convention in order to run properly. You will need to determine which file corresponds to which sample and which read type, likely by consulting your sequencing core or the individual who demultiplexed your flowcell.

It is highly likely that these files were initially processed with `bcl2fastq`, so you will need to rename the files in the following format, once you track down their origin:

`[Sample Name]`\_S1\_L00`[Lane Number]`\_`[Read Type]`\_001.fastq.gz

Where `Read Type` is one of:

*   `I1`: Sample index read (optional)
*   `I2`: Sample index read (optional)
*   `R1`: Read 1
*   `R2`: Read 2

After you have renamed those files into that format, you'll use the following arguments:

Situation

Argument+Value

All samples

`--fastqs=/PATH/TO/PROJECT_FOLDER`

Process `SAMPLENAME` from all lanes

`--fastqs=/PATH/TO/PROJECT_FOLDER \`  
`--sample=SAMPLENAME`

Process `SAMPLENAME` from lane 1 only

`--sample=SAMPLENAME \`  
`--fastqs=/PATH/TO/PROJECT_FOLDER \`  
`--lanes=1`

### 重命名前

├── SRR12391722  
│ ├── SRR12391722\_1.fastq.gz  
│ └── SRR12391722\_2.fastq.gz  
├── SRR12391723  
│ ├── SRR12391723\_1.fastq.gz  
│ └── SRR12391723\_2.fastq.gz  
├── SRR12391724  
│ ├── SRR12391724\_1.fastq.gz  
│ └── SRR12391724\_2.fastq.gz  
└── SRR12391725  
│├── SRR12391725\_1.fastq.gz  
│└── SRR12391725\_2.fastq.gz

### 重命名后

SRX8890106  
├── SRX8890106\_S1\_L001\_R1\_001.fastq.gz  
├── SRX8890106\_S1\_L001\_R2\_001.fastq.gz  
├── SRX8890106\_S1\_L002\_R1\_001.fastq.gz  
├── SRX8890106\_S1\_L002\_R2\_001.fastq.gz  
├── SRX8890106\_S1\_L003\_R1\_001.fastq.gz  
├── SRX8890106\_S1\_L003\_R2\_001.fastq.gz  
├── SRX8890106\_S1\_L004\_R1\_001.fastq.gz  
└── SRX8890106\_S1\_L004\_R2\_001.fastq.gz

## 进行定量

[Running cellranger count](https://support.10xgenomics.com/single-cell-gene-expression/software/pipelines/latest/using/tutorial_ct); [cellranger 安装](https://support.10xgenomics.com/single-cell-gene-expression/software/pipelines/latest/installation)

```bash
#!/bin/bash
export PATH=/opt/cellranger/cellranger-6.1.2:$PATH
db=/opt/cellranger/refdata-gex-GRCh38-2020-A
data=/home/jovyan/work_st/sce/GSE137829/data
work=/home/jovyan/work_st/sce/GSE137829/res
mkdir $work
cd $work
for sample in ${data}/*;
do
echo $sample
sample_res=${sample##*/}
cellranger count --id=$sample_res \
--localcores=12 \
--transcriptome=$db \
--fastqs=$sample \
--sample=$sample_res \
--expect-cells=5000
done
```

*   nano 103.sh
*   chmod +x 103.sh
*   ./103.sh