---
title: oligo：GEO/ArrayExpress芯片数据处理
tags: []
id: '1939'
categories:
  - - uncategorized
date: 2022-07-29 10:15:10
---

## conda配置环境

*   conda clean -y -all
*   conda create -n geo -c conda-forge r-base=4.1.3
*   conda activate geo
*   conda install -c conda-forge r-tidyverse=1.3.1 -y
*   conda install -c conda-forge r-irkernel -y
*   conda install -c conda-forge r-r.utils -y
*   conda install -c conda-forge r-rcolorbrewer -y
*   Rscript -e "IRkernel::installspec(name='geo', displayname='r-geo')"
*   conda install -c bioconda bioconductor-geoquery -y
*   conda install -c bioconda bioconductor-oligo -y
*   conda install -c bioconda bioconductor-affy -y
*   conda install -c bioconda bioconductor-pd.hugene.1.0.st.v1 -y # read.celfiles 需要啥[安装啥](https://anaconda.org/bioconda/bioconductor-pd.hg.u133a.2)
*   conda install -c bioconda bioconductor-pd.hg.u133a.2 -y
*   conda install -c bioconda bioconductor-arrayexpress -y
*   conda install -c bioconda bioconductor-biomart -y
*   conda install -c bioconda bioconductor-affycoretools -y

## 下载芯片数据

### GEO

```R
library(GEOquery)
library(oligo)
library(affy)
gse="GSE43332"
rawdata <- getGEOSuppFiles(gse) #下载原始数据
rawdata
setwd(gse)
untar("GSE43332_RAW.tar",exdir = ".")
celfiles <- list.files(pattern = "*CEL.gz$") #批量查找并列出后缀为.gz的文件
data.raw <- read.celfiles(celfiles)
```

### ArrayExpress

*   ~/dev/xray/xray -c ~/etc/xui2.json &
*   wget -e "https\_proxy=http://127.0.0.1:20809" https://www.ebi.ac.uk/arrayexpress/files/E-MEXP-1422/E-MEXP-1422.raw.1.zip -O E-MEXP-1422.raw.1.zip
*   unzip E-MEXP-1422.raw.1.zip

```R
library(ArrayExpress)
library(oligo)
library(affy)
# Sys.setenv("http_proxy"="http://127.0.0.1:20809")
# Sys.setenv("https_proxy"="http://127.0.0.1:20809")
# mexp1422=getAE("E-MTAB-6128", type = 'raw', extract = T)
celfiles <- list.files(pattern = "*.CEL$") #批量查找并列出后缀为CEL的文件
data.raw <- read.celfiles(celfiles)
```

## RMA标准化数据

```R
sampleNames(data.raw) <- sapply(strsplit(sampleNames(data.raw),"_",fixed=T), "[",1)
sampleNames(data.raw)
 
fit1 <- fitProbeLevelModel(data.raw)
 
image(fit1,type="weights",which=1,main="weights")
image(fit1,type="residuals",which=1,main="Residuals")
image(fit1,type="sign.residuals",which=1,main="Residuals.sign")
 
data.eset <- oligo::rma(data.raw)
data.exprs <- exprs(data.eset)
 
library(RColorBrewer)
display.brewer.all()
if(ncol(data.eset)>8){
    colors <- c(brewer.pal(8,"Set2"),brewer.pal(ncol(data.eset)-8,"Set1"))
}else{
    colors <- brewer.pal(ncol(data.eset),"Set2")
}
boxplot(data.exprs,col=colors,main="afterRMA")
```

## 探针转Symbol

*   [f\_dedup\_IQR 函数来源](https://occdn.limour.top/2157.html)
*   data.raw@annotation 可以得到注释的包信息，如pd.hg.u133a.2
*   在[bioconductor](https://www.bioconductor.org/)上搜索pd.hg.u133a.2，可以知道其Platform为HG-U133A\_2
*   谷歌搜索 HG-U133A\_2 + affymetrix，可以找到affymetrix官网对应的页面
*   谷歌搜索 affymetrix "HG-U133A\_2" site:www.ncbi.nlm.nih.gov，随便进一个GSE页面，找到对应的Platforms，如 GPL571 \[HG-U133A\_2\] Affymetrix Human Genome U133A 2.0 Array，说明HG-U133A\_2对应GPL571

```R
require(GEOquery)
f_getGPL <- function(lc_GPLN, lc_local = F){
    options(stringsAsFactors = F)
    if (!file.exists(lc_GPLN)){dir.create(lc_GPLN)}
    if(lc_local){
        gpl=read.table(file.path(lc_GPLN, lc_GPLN),
               header = TRUE,fill = T,sep = "\t",
               comment.char = "#",
               stringsAsFactors = FALSE,
               quote = "")
        return(gpl)
    }else{
        return(Table(getGEO(lc_GPLN,destdir = lc_GPLN)))
    }
}
GPL <- 'GPL6244'
GPL <- f_getGPL(GPL)
GPL[['Symbol']] <- GPL$gene_assignment
GPL <- GPL[c('ID','Symbol')]
GPL[GPL$Symbol == '---', 'Symbol'] = '---//---'
tmp <- strsplit(x = GPL$Symbol, split = '//')
tmp <- lapply(tmp, FUN = function(x){x[2]})
GPL[['Symbol']] = unlist(tmp)
GPL$Symbol[GPL$Symbol == '---'] = GPL$ID[GPL$Symbol == '---']
rownames(GPL) <- as.character(GPL$ID)
GPL <- GPL[as.character(rownames(data.exprs)),]
data.exprs <- f_dedup_IQR(data.exprs, GPL$Symbol)
data.exprs
```

## GPL与Platform对应表

gpl

bioc\_package

title

GPL32

mgu74a

\[MG\_U74A\] Affymetrix Murine Genome U74A Array

GPL33

mgu74b

\[MG\_U74B\] Affymetrix Murine Genome U74B Array

GPL34

mgu74c

\[MG\_U74C\] Affymetrix Murine Genome U74C Array

GPL71

ag

\[AG\] Affymetrix Arabidopsis Genome Array

GPL72

drosgenome1

\[DrosGenome1\] Affymetrix Drosophila Genome Array

GPL74

hcg110

\[HC\_G110\] Affymetrix Human Cancer Array

GPL75

mu11ksuba

\[Mu11KsubA\] Affymetrix Murine 11K SubA Array

GPL76

mu11ksubb

\[Mu11KsubB\] Affymetrix Murine 11K SubB Array

GPL77

mu19ksuba

\[Mu19KsubA\] Affymetrix Murine 19K SubA Array

GPL78

mu19ksubb

\[Mu19KsubB\] Affymetrix Murine 19K SubB Array

GPL79

mu19ksubc

\[Mu19KsubC\] Affymetrix Murine 19K SubC Array

GPL80

hu6800

\[Hu6800\] Affymetrix Human Full Length HuGeneFL Array

GPL81

mgu74av2

\[MG\_U74Av2\] Affymetrix Murine Genome U74A Version 2 Array

GPL82

mgu74bv2

\[MG\_U74Bv2\] Affymetrix Murine Genome U74B Version 2 Array

GPL83

mgu74cv2

\[MG\_U74Cv2\] Affymetrix Murine Genome U74 Version 2 Array

GPL85

rgu34a

\[RG\_U34A\] Affymetrix Rat Genome U34 Array

GPL86

rgu34b

\[RG\_U34B\] Affymetrix Rat Genome U34 Array

GPL87

rgu34c

\[RG\_U34C\] Affymetrix Rat Genome U34 Array

GPL88

rnu34

\[RN\_U34\] Affymetrix Rat Neurobiology U34 Array

GPL89

rtu34

\[RT\_U34\] Affymetrix Rat Toxicology U34 Array

GPL90

ygs98

\[YG\_S98\] Affymetrix Yeast Genome S98 Array

GPL91

hgu95av2

\[HG\_U95A\] Affymetrix Human Genome U95A Array

GPL92

hgu95b

\[HG\_U95B\] Affymetrix Human Genome U95B Array

GPL93

hgu95c

\[HG\_U95C\] Affymetrix Human Genome U95C Array

GPL94

hgu95d

\[HG\_U95D\] Affymetrix Human Genome U95D Array

GPL95

hgu95e

\[HG\_U95E\] Affymetrix Human Genome U95E Array

GPL96

hgu133a

\[HG-U133A\] Affymetrix Human Genome U133A Array

GPL97

hgu133b

\[HG-U133B\] Affymetrix Human Genome U133B Array

GPL98

hu35ksuba

\[Hu35KsubA\] Affymetrix Human 35K SubA Array

GPL99

hu35ksubb

\[Hu35KsubB\] Affymetrix Human 35K SubB Array

GPL100

hu35ksubc

\[Hu35KsubC\] Affymetrix Human 35K SubC Array

GPL101

hu35ksubd

\[Hu35KsubD\] Affymetrix Human 35K SubD Array

GPL198

ath1121501

\[ATH1-121501\] Affymetrix Arabidopsis ATH1 Genome Array

GPL199

ecoli2

\[Ecoli\_ASv2\] Affymetrix E. coli Antisense Genome Array

GPL200

celegans

\[Celegans\] Affymetrix C. elegans Genome Array

GPL201

hgfocus

\[HG-Focus\] Affymetrix Human HG-Focus Target Array

GPL339

moe430a

\[MOE430A\] Affymetrix Mouse Expression 430A Array

GPL340

mouse4302

\[MOE430B\] Affymetrix Mouse Expression 430B Array

GPL341

rae230a

\[RAE230A\] Affymetrix Rat Expression 230A Array

GPL342

rae230b

\[RAE230B\] Affymetrix Rat Expression 230B Array

GPL570

hgu133plus2

\[HG-U133\_Plus\_2\] Affymetrix Human Genome U133 Plus 2.0 Array

GPL571

hgu133a2

\[HG-U133A\_2\] Affymetrix Human Genome U133A 2.0 Array

GPL886

hgug4111a

Agilent-011871 Human 1B Microarray G4111A (Feature Number version)

GPL887

hgug4110b

Agilent-012097 Human 1A Microarray (V2) G4110B (Feature Number version)

GPL1261

mouse430a2

\[Mouse430\_2\] Affymetrix Mouse Genome 430 2.0 Array

GPL1318

xenopuslaevis

\[Xenopus\_laevis\] Affymetrix Xenopus laevis Genome Array

GPL1319

zebrafish

\[Zebrafish\] Affymetrix Zebrafish Genome Array

GPL1322

drosophila2

\[Drosophila\_2\] Affymetrix Drosophila Genome 2.0 Array

GPL1352

u133x3p

\[U133\_X3P\] Affymetrix Human X3P Array

GPL1355

rat2302

\[Rat230\_2\] Affymetrix Rat Genome 230 2.0 Array

GPL1708

hgug4112a

Agilent-012391 Whole Human Genome Oligo Microarray G4112A (Feature Number version)

GPL2112

bovine

\[Bovine\] Affymetrix Bovine Genome Array

GPL2529

yeast2

\[Yeast\_2\] Affymetrix Yeast Genome 2.0 Array

GPL2891

h20kcod

GE Healthcare/Amersham Biosciences CodeLink™ UniSet Human 20K I Bioarray

GPL2898

adme16cod

GE Healthcare/Amersham Biosciences CodeLink™ ADME Rat 16-Assay Bioarray

GPL3154

ecoli2

\[E\_coli\_2\] Affymetrix E. coli Genome 2.0 Array

GPL3213

chicken

\[Chicken\] Affymetrix Chicken Genome Array

GPL3533

porcine

\[Porcine\] Affymetrix Porcine Genome Array

GPL3738

canine2

\[Canine\_2\] Affymetrix Canine Genome 2.0 Array

GPL3921

hthgu133a

\[HT\_HG-U133A\] Affymetrix HT Human Genome U133A Array

GPL3979

canine

\[Canine\] Affymetrix Canine Genome 1.0 Array

GPL4032

\[Maize\] Affymetrix Maize Genome Array

GPL4191

h10kcod

CodeLink UniSet Human I Bioarray

GPL5188

huex10sttranscriptcluster

\[HuEx-1\_0-st\] Affymetrix Human Exon 1.0 ST Array \[probe set (exon) version\]

GPL5689

hgug4100a

Agilent Human 1 cDNA Microarray (G4100A) \[layout C\]

GPL6097

illuminaHumanv1

Illumina human-6 v1.0 expression beadchip

GPL6102

illuminaHumanv2

Illumina human-6 v2.0 expression beadchip

GPL6244

hugene10sttranscriptcluster

\[HuGene-1\_0-st\] Affymetrix Human Gene 1.0 ST Array \[transcript (gene) version\]

GPL6246

mogene10sttranscriptcluster

\[MoGene-1\_0-st\] Affymetrix Mouse Gene 1.0 ST Array \[transcript (gene) version\]

GPL6885

illuminaMousev2

Illumina MouseRef-8 v2.0 expression beadchip

GPL6947

illuminaHumanv3

Illumina HumanHT-12 V3.0 expression beadchip

GPL8300

hgu95av2

\[HG\_U95Av2\] Affymetrix Human Genome U95 Version 2 Array

GPL8321

mouse430a2

\[Mouse430A\_2\] Affymetrix Mouse Genome 430A 2.0 Array

GPL8490

IlluminaHumanMethylation27k

Illumina HumanMethylation27 BeadChip (HumanMethylation27\_270596\_v.1.2)

GPL10558

illuminaHumanv4

Illumina HumanHT-12 V4.0 expression beadchip

GPL11532

hugene11sttranscriptcluster

\[HuGene-1\_1-st\] Affymetrix Human Gene 1.1 ST Array \[transcript (gene) version\]

GPL13497

HsAgilentDesign026652

Agilent-026652 Whole Human Genome Microarray 4x44K v2 (Probe Name version)

GPL13534

IlluminaHumanMethylation450k

Illumina HumanMethylation450 BeadChip (HumanMethylation450\_15017482)

GPL13667

hgu219

\[HG-U219\] Affymetrix Human Genome U219 Array

GPL14877

hgu133plus2

Affymetrix Human Genome U133 Plus 2.0 Array \[Brainarray Version 13, HGU133Plus2\_Hs\_ENTREZG\]

GPL15380

GGHumanMethCancerPanelv1

Illumina Sentrix Array Matrix (SAM) - GoldenGate Methylation Cancer Panel I

GPL15396

hthgu133b

\[HT\_HG-U133B\] Affymetrix HT Human Genome U133B Array \[custom CDF: ENTREZ brainarray v. 14\]

GPL17556

hugene10sttranscriptcluster

\[HuGene-1\_0-st\] Affymetrix Human Gene 1.0 ST Array \[HuGene10stv1\_Hs\_ENTREZG\_17.0.0\]

GPL17897

hthgu133a

\[HT\_HG-U133A\] Affymetrix Human Genome U133A Array (custom CDF: HTHGU133A\_Hs\_ENTREZG.cdf version 17.0.0)

GPL18190

hugene11sttranscriptcluster

\[HuGene-1\_1-st\] Affymetrix Human Gene 1.1 ST Array \[CDF: Brainarray HuGene11stv1\_Hs\_ENTREZG\_15.1.0\]

来自：[爱代码爱编程](https://icode.best/i/32522930456304)