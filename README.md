# legacy-SOC-project
## 小组成员
- 胡凯文2025303110074
- 高健平2025303110053 
  
## 项目信息
- 论文标题：《The centennial legacy of land-use change on organic carbon stocks of German agricultural soils.》
- 发表期刊：Global Change Biology（2024）
- DOI：https://doi.org/10.1111/gcb.17444
- 原始数据与代码：https://github.com/dsemde/LUC-legacy-SOC
  
## 复现目标
**对于文中的所有图片进行复现**

## 复现结果
- 小组成员根据作者所提供的数据代码并且结合课堂所学知识，对文献中的两张图片进行了复现。
  
**原文图片**
<img width="1154" height="856" alt="image" src="https://github.com/user-attachments/assets/ac0789b0-25e9-4909-a459-17a2b09e0990" />

<img width="1091" height="855" alt="image" src="https://github.com/user-attachments/assets/55b7516b-7b2d-49af-b2ca-b2bf0ebbf968" />



**复现图片**
<img width="3188" height="2125" alt="fig_3_grass_crop_luc_intervals_final" src="https://github.com/user-attachments/assets/02c277a0-acb3-44aa-b6e5-a6c94cfa4eb4" />

<img width="3188" height="2125" alt="fig_5_crop_grass_luc_intervals_final" src="https://github.com/user-attachments/assets/096c9dc0-eec0-4fbf-893e-9ad3fac8d03c" />
可以看出复现的图片与文献中的图片相似度是极高的，说明复现的结果是不错的，但是也存在一些不足。由于对于作者在文献中所给出的代码的了解程度不够，在运行剩余代码时出现了一些问题，且小组成员在查阅相关资料后仍无法解决，便不再对剩下的图片进行复现。作者所提供的数据资料中包含了涉及国家隐私的内容，作者没有给出这些数据，这也是无法进行复现的原因。
<img width="1205" height="342" alt="image" src="https://github.com/user-attachments/assets/e96af722-0799-4bae-8e58-98a97a920d56" />

<img width="1377" height="376" alt="image" src="https://github.com/user-attachments/assets/00d59861-8e66-453e-9f6f-ada23a414e0a" />


## 复现过程
- 1、加载所有依赖包
```shell
### Load libraries -------------------------------------------------------------
# 数据处理
library(tidyverse)
library(here)       # 路径管理
library(radiant.data)
library(boot)       # 自助法

# 聚类/倾向性得分
library(cluster)
library(twang)      # 多分类倾向性得分

# 绘图
library(ggplot2)
library(ggpubr)
library(gridExtra)
library(showtext)   # 字体

# 并行计算
library(doParallel)
library(foreach)

set.seed(1337) # 固定随机数
```

- 2、加载并清洗数据
```shell
### Load dataset ---------------------------------------------------------------
combined <- read.csv(here("dat/derived/g_to_c_3_depths.csv")) |>
  select(-c(xcoord, ycoord)) |>
  filter(!is.na(TOC_stock_cm))

dat <- combined  |> 
  filter(history == "G_to_C" & depth_increment == "0-10cm")
```

- 3、用自然断点法（Jenks）划分转换年限区间
```shell
### Generate year intervals-----------------------------------------------------
n <- 6
interval_list <- NULL

for (i in 1:n) {
  groups <- getBreaks(dat$year_change, nclass = 2, method = "jenks") |> 
    floor()
  
  dat <- dat  |> 
    mutate(history = case_when(
      between(year_change, groups[2], groups[3]) ~ "newest",
      between(year_change, groups[1], groups[2]) ~ paste0(groups[1], "-", groups[2])
    )) |> 
    filter(history == "newest")
  
  if (i < n) {
    interval_list[i] <- groups[1]
  } else {
    interval_list[i] <- groups[2]
    interval_list[i + 1] <- groups[3]
  }
}
```
