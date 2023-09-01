---
title: "TCGAbiolinks (三）获取全面的临床数据\_"
tags: []
id: '1904'
categories:
  - - uncategorized
date: 2022-07-13 23:11:14
---

## 基础数据获取

```R
library(TCGAbiolinks)
PRAD <- GDCquery(project = 'TCGA-PRAD',
                  data.category = "Clinical",
                  file.type = "xml")
GDCdownload(query = PRAD)
? GDCprepare_clinic
clinical.info <- c('admin', 'patient', 'stage_event', 'new_tumor_event') # 不获取'drug', 'follow_up', 'radiation'
clinical.info
f_rm_colN <- function(df, regex){
    df[,!grepl(regex, colnames(df))]
}
f_rm_duplicated <- function(NameL, reverse=F){
    tmp <- data.frame(table(NameL))
    if(reverse){
        tmp <- tmp$NameL[tmp$Freq > 1]
    }else{
        tmp <- tmp$NameL[tmp$Freq == 1]
    }
    which(NameL %in% as.character(tmp))
}
clinical <- list()
for(info in clinical.info){
    clinical[[info]]  <- GDCprepare_clinic(PRAD, clinical.info = info)
    clinical[[info]] <- f_rm_colN(clinical[[info]], "project")
}
clinical$admin <- f_rm_colN(clinical$admin, "file_uuid")
for(info in clinical.info){
    clinical[[info]] <- unique(clinical[[info]])
}
f_merge  <- function(lc_mergedList, by, all=T){
    Reduce(function(...) merge(..., by=by, all=all), lc_mergedList)
}
clinical <- f_merge(clinical, by = 'bcr_patient_barcode', all = T)
clinical
```

## 数据更新补丁

```R
cl_new <- GDCquery_clinic(project = 'TCGA-PRAD', type = 'clinical')
clinical <- merge(clinical, cl_new, by = 'bcr_patient_barcode', all = T, suffixes = c('.old', '.new'))
clinical
```

## 生存分析补丁

```R
clinical[['os_status']]  <- with(clinical, ifelse(vital_status.new == 'Dead', 0, 1)) # 0表示因病死亡
clinical[['os_time']] <- with(clinical, ifelse(os_status == 0, days_to_death.new, days_to_last_follow_up))
sum(clinical$os_time)
clinical[clinical$patient_death_reason == 'Other, non-malignant disease', 'os_status'] = 1
sum(clinical$os_status)
 
clinical[['dcf_status']] <- with(clinical, ifelse(new_neoplasm_event_type == '', 0, 1)) # 1表示有新的肿瘤事件
clinical[['dcf_status']] <- with(clinical, ifelse(is.na(dcf_status), 0, dcf_status)) # 1表示有新的肿瘤事件
clinical[clinical$biochemical_recurrence == 'YES', 'dcf_status'] = 1
sum(clinical$dcf_status)
 
clinical[['tmp_dcf_time']] <- clinical[['days_to_first_biochemical_recurrence']]
clinical[['tmp_dcf_time']] <- with(clinical, ifelse(is.na(tmp_dcf_time), os_time, tmp_dcf_time))
sum(clinical$tmp_dcf_time)
clinical[['dcf_time']] <- with(clinical, ifelse(dcf_status == 1, days_to_new_tumor_event_after_initial_treatment, tmp_dcf_time))
clinical[['dcf_time']] <- with(clinical, ifelse(is.na(dcf_time), tmp_dcf_time, dcf_time))
sum(clinical$dcf_time)
clinical[['dcf_time']] <- with(clinical, ifelse(biochemical_recurrence == 'YES', tmp_dcf_time, dcf_time))
sum(clinical$dcf_time)
clinical <- f_rm_colN(clinical, "tmp_dcf_time")
write.csv(clinical, file = 'TCGA-PRAD_clinical.csv')
 
sum(clinical$os_status)
sum(clinical$os_time)
sum(clinical$dcf_status)
sum(clinical$dcf_time)
```