---
author: "LongWei"
title: "ECG论文和实验"
description: "ECG心电图论文创新点整理和初步实验"
date: "2024-04-19"
tags: ["ECG"]
categories: ["Learning"]
series: ["Learning"]
ShowToc: true
TocOpen: true
---


## 论文

### ⭐IM-ECG  - 2023

**IM-ECG: An interpretable framework for arrhythmia detection using  multi-lead ECG**   *- Expert Systems With Applications  sci-1*

创新点:

- Conv2D模型与Grad-CAM的适配更好  - 实时标注病理区域  **解释性**

- 双核残差块  - 横轴与竖轴+面 三种扫描方式

![image-20231008222054322](http://sthda9dn6.hd-bkt.clouddn.com/Fs6z6MvcLx6Jgu0ZSJQQhP_ELsED)

**k * n**(区域扫描)内核朝向中心压缩ECG以更直接地捕获**导联间**特征，而**1 * n**(横轴扫描)内核沿着时间维度压缩ECG并且因此更关注**导联内**特征



**流程图示**

![image-20240327154037056](http://sthda9dn6.hd-bkt.clouddn.com/Foik_oJeDo9NKImyPhkZ5ga6ZRKr)



**模型结构**

![image-20240327153836525](http://sthda9dn6.hd-bkt.clouddn.com/FpbYbRq6Dec9rkOSDKeD_5zMzfa5)

**Block**

![image-20240327153914172](http://sthda9dn6.hd-bkt.clouddn.com/Fv2Ky4pSh8O5sTf5Pm79Ea8BR_P6)







---

### Lightweight Transformer  - 2022

**Enhancing dynamic ECG heartbeat classification with lightweight transformer model**  

 *Artificial Intelligence In Medicine*  *sci-1*



创新点：

- 两级注意力机制： 局部 + 全局 // 局部注意力 == 通道注意力 SEBlock  || 全局注意力 == Transformer Encoder

- 卷积结构来代替自注意



**Input:**   检测R峰分段 

![image-20240327125618915](http://sthda9dn6.hd-bkt.clouddn.com/Fp0qMCKJehhYdZfSTcGFtSLKaqda)

<center style="color:red; font-weight:bolder;">模型框架</center>

![image-20240327125748959](http://sthda9dn6.hd-bkt.clouddn.com/FqS1JmYorR7xtQKst3yYNvfUfIL9)

<center style="color:blue; font-weight:bolder;">CNN Block</center>

**CNN+Attention来提取心跳内部的特征**



![image-20240327125829274](http://sthda9dn6.hd-bkt.clouddn.com/FvrU_l1mGaRqSGPnXmOWVgwqjJOY)

<center style="color:blue; font-weight:bolder;">light-Conv Attention</center>

**GLU:**  *gated linear unit* 

**Lconv:**   *depth-wise Convolution*





---

### SE-ECGNet  - 2020

**SE-ECGNet: A Multi-scale Deep Residual Network with Squeeze-and-Excitation Module for ECG Signal Classification**

*2020 IEEE International Conference on Bioinformatics and Biomedicine (BIBM)*



利用ECG信号中的多导联信息，将多尺度二维卷积块与一维卷积块结合起来进行特征提取

*创新点*

- 提供对心电数据的不同视角

![image-20231105144220615](http://sthda9dn6.hd-bkt.clouddn.com/FplCmRZMAWQfKug99yBNAV0UkgsD)

***将ECG信号视为二维图片(单通道灰度图)***





![image-20231105144204757](http://sthda9dn6.hd-bkt.clouddn.com/Fv-5ITe8vMDvLK5lMcRr9eHFI6ud)

<center style="color:red; font-weight:bolder;">模型框架</center>

![image-20240327131154616](http://sthda9dn6.hd-bkt.clouddn.com/FoGGVWKihGHZdOPfwlYf0_epkxS8)

<center style="color:blue; font-weight:bolder;">CNN Block</center>

前期较大的卷积核可以扩大感受野，提取更加广泛的信息，后期进行信息浓缩提取





---

### ⭐MVMS-ECG  - 2023

***A Multi-View Multi-Scale Neural Network for Multi-Label ECG Classification***

*IEEE Transactions on Emerging Topics in Computational Intelligence*    sci-2



创新点：

- 将导联按照连接观测角度分组

![image-20240327133550908](http://sthda9dn6.hd-bkt.clouddn.com/FnFnzWHtAv0k7IkwX0718iPNFNXB)

分视图的融合网络为教室网络(参数大)  =蒸馏> 单视图的学生网络



<strong style='color:blue'>模型结构</strong>

![image-20240327133617466](http://sthda9dn6.hd-bkt.clouddn.com/FiNsKiKuzti5awPUyu6-R0BO2KLw)

<strong>CNN Layers</strong>

![image-20240327133758012](http://sthda9dn6.hd-bkt.clouddn.com/FpbH-DAtQ65hSmDoDZh15ExizCWE)

**Fusion Layer**

![image-20240327133955260](http://sthda9dn6.hd-bkt.clouddn.com/Fgwlt20ID1AFUrT32X5VPdh7OeNO)

如果是多视图，那么就不要最后的FC将最后特征映射为类别数；

源码 => 每个视图output都是 [B, 128]  => 重新加权(128=>1的linear充当调和器) => 过多视图的FC(128 => num_class)



**Multi-scale Conv Block**

![image-20240327134133763](http://sthda9dn6.hd-bkt.clouddn.com/FhCurIVUjpxA5WlCZTtlMvGbdqb2)

网络的多尺度表示在更细粒度的级别上得到增强  - 采用Res2Block 

**Coordinate Attention 1D**

![image-20240327134033425](http://sthda9dn6.hd-bkt.clouddn.com/FsmP-vtRKCIdIuhC7CdcWzXi-2bB)

既建立了特征通道间的关系，对各通道的重要性进行重新加权，又获得了特征空间中的空间信息





---

### MVKT-ECG  - 2023

MVKT-ECG: Efficient single-lead ECG classification for multi-label arrhythmia by multi-view knowledge transferring

*Computers in Biology and Medicine*    *sci-2*



创新点：

​	# 多导联信息如何蒸馏到单导联中

![image-20240327140327574](http://sthda9dn6.hd-bkt.clouddn.com/Fi_18gQrmh7c-7go8uidNED7nf-Z)

不同导联是检测心脏状况的不同视角，提供对目标疾病的多种观测点和多种外貌

=> 目标：是教会网络从单导联ECG信号中恢复尽可能多的全导联信息



#### 导联共同信息的知识蒸馏

虽然有些疾病不能从单导联ECG信号中推断出来，但我们的目标是尽可能缩短单导联ECG解释模型和多导联ECG解释模型之间的信息差距，鼓励学生更多地关注某些特定疾病的关键细节，最大限度地利用单导联ECG解释模型和多导联ECG解释模型之间的疾病信息。

=> 通过缩短学生模型和教师模型对积极对的表示并将“消极"对之间的表示分开来实现的

⭐⭐⭐最大化单导联ECG信号和多导联ECG信号之间的互(共同)信息来传递有用的疾病信息。

因为：从单导联ECG信号中提取的疾病信息比12导联ECG信号少，但仍包含部分疾病信息。

❗❗❗ 引出CLT-Loss  太难了，没看

**面向特征与特征的相似度 与 最终预测和标签的Loss目的不一样**



#### 多标签知识蒸馏的优化

![image-20240327140614683](http://sthda9dn6.hd-bkt.clouddn.com/Fksp0kSQPFuG0pYKkHTJBL-MIpNg)

仿照CE-Loss 和 BCE-Loss





---

### Multi-Modal + Instance  - 2023

***Multimodal multi-instance learning for long-term ECG classification***   ***- Knowledge-Based Systems***



**节拍 视为 实例**

![image-20240327143559281](http://sthda9dn6.hd-bkt.clouddn.com/Fuo85Dhw-HA8ouH_YRBCHOZYwf8r)



**双模态：一维信号 + 二维图片**

![image-20240327143524604](http://sthda9dn6.hd-bkt.clouddn.com/FjcNYGoZN1YBN3HSxnZNdM80SWYj)



#### 多模态信息融合

*传统：*

![image-20240327145248469](http://sthda9dn6.hd-bkt.clouddn.com/FkVnwJp6tOwwxZ--DXo1l15fn3fh)

*新方法*

![image-20240327145025906](http://sthda9dn6.hd-bkt.clouddn.com/Fnddrqbhd0IT3RH11Xg5-YMHi-i4)

​	通过使用最大池化操作从每个模态的实例特征中选择**顶部激活实例特征(代表)**。然后**计算顶部激活实例特征和所有剩余实例特征之间的相关性分数**以获得特征向量，其中包含来自两种模态的信息。最后，将**特征向量和顶部激活实例特征向量融合**，得到每个模态的bag特征。每种模态的袋子特征通过线性层拼接在一起，得到最终的袋子特征。

![image-20240327145455889](http://sthda9dn6.hd-bkt.clouddn.com/Fu86vYzeaSrjlTNJqPk50rE85aSx)





---

### Multi Res Trans Net  - 2023

*Multi-scale SE-ResBlock + Transformer Encoder*

**Multi-scale SE-residual network with transformer encoder for myocardial infarction classification**    *Applied Soft Computing  sci-1*



提取局部特征和全局特征

**模型结构**

*分段*  - 用具有重叠的滑动窗口

![image-20240327145737234](http://sthda9dn6.hd-bkt.clouddn.com/FtU7Qr5ZWM0JAUX-n-CnwiucWtMS)

**Multi-scale sample layer**

![image-20240327145938698](http://sthda9dn6.hd-bkt.clouddn.com/Fo4R_1qJXLkbb6f0lzXD8I-jCS5l)

多尺度采样模块  名称不错

**SE-Resnet Block**

![image-20240327150034480](http://sthda9dn6.hd-bkt.clouddn.com/FrD7qjLSBvjRtVPqSWHmC5H6Ww-u)

**CPC**

![image-20240327150132721](http://sthda9dn6.hd-bkt.clouddn.com/Fo6jhylOIAAM38mCNeFx0PDKHeGw)

GAP(全局平均池化)定位的是整体区域，而GMP(全局最大池化)定位的是目标区域中最重要的部分。





---

### ⭐Dual-Branch CNN-Trans + Select  - 2023

**A token selection-based multi-scale dual-branch CNN-transformer network for 12-lead ECG signal classification**   

*Knowledge-Based Systems sci-1*

**模型结构**

![image-20240327151553655](http://sthda9dn6.hd-bkt.clouddn.com/Fh6H-Ok8KNDg1MrXuFwXNjNKiGCn)

RR表示计算两个分支中CLS token之间的相关系数  => RR-Loss 迫使两个分支朝着最终预测同向而行

**CNN Blocks**

![image-20240327151939118](http://sthda9dn6.hd-bkt.clouddn.com/FiVVTyUs1OP57irjiUK72gGSW7up)

⃝+ is element-wise addition

**MSEL**

![image-20240327152059740](http://sthda9dn6.hd-bkt.clouddn.com/FsosvYLsW72QYBTqMRjVFW8yWn5h)

一般采用ViT中的嵌入, Conv1D实现

通过不同大小的Token嵌入,或许可以捕获到不同的模式



**Token Select**

![image-20240327152029903](http://sthda9dn6.hd-bkt.clouddn.com/FqJlAwMplpETb2U_cCp7-0xQH603)

删掉冗余的Token,或许可以在论文中可视化一下自注意力图,直观的描述冗余





---

### ECGNet  - 2018

ECGNet: Deep Network for Arrhythmia Classification

![image-20240327152958835](http://sthda9dn6.hd-bkt.clouddn.com/FriRBcj2_DfxlrlMWU-kg8yaPsjA)

Inception Block + Conv Block × N





---

### Multi module: LSTM + CNN + AutoEncoder

Multi-module Recurrent Convolutional Neural Network with Transformer Encoder for ECG Arrhythmia Classification

![image-20240327153623587](http://sthda9dn6.hd-bkt.clouddn.com/Fkj6LwIApmG5o4N4065tHkp1O6Ar)



---

### ECG Dual-path RNN - 2022/8

**Single-lead ECG recordings modeling for end-to-end recognition of atrial fibrillation with dual-path RNN**

*Biomedical Signal Processing and Control*  二区

![image-20240227141810637](http://sthda9dn6.hd-bkt.clouddn.com/FjwE0iqlQ1pMIPOd3hGbi57kzJb7)



***Segmentation:***

![image-20240227143633528](http://sthda9dn6.hd-bkt.clouddn.com/Fru1F4NH2dxJMIbnipQdone8UfHs)

[batch, 1, L]  => [batch, num_seg, len_seg]

重叠50%

***Overlap-Add:***

​	Segmentation的逆操作，重叠相加



RNN为Bi-LSTM

LSTM 输入数据格式 Batch, num_seq, len_seq

段间建模 + 段内建模



***图例：***

![image-20240228141118686](http://sthda9dn6.hd-bkt.clouddn.com/FuGAi3WkezRO-whsi4IhnZ2AhOYm)

![image-20240228141137134](http://sthda9dn6.hd-bkt.clouddn.com/Fi2DDId0llMuv6ZCQELc4_3px43v)

类似MLP-Mixer，用RNN建模信息融合，隐状态传递时序信息





---

### ⭐MINA  - Signal Processing 2019

*MINA: **Multilevel Knowledge-Guided Attention** for Modeling Electrocardiography Signals*

做法：

- 特征工程：从心电图波形中提取信息特征 => 传统机器学习进行处理 => 结果

  探索P-QRS-T 波的各种幅度和持续时间特征，包括用于分类的形态学 和 RR 间期特征  // Hermite变换和小波变换

  ❗❗❗**依赖于提取的特征，很容易受到噪声干扰，特征没提取好，后续工作很难进展**

- 注意力引导网络关注重点 （通用自学习注意力）和（融合领域知识的注意力）



3级注意力：（节拍级、节律级和频率级）领域知识特征

![image-20240320125118402](http://sthda9dn6.hd-bkt.clouddn.com/FhMwwXFFzXiQK-fdxywD7iQOxcpz)



**⭐提取特定级别的领域知识特征并使用它们来引导注意力，包括<strong style="color:red">引导注意力CNN的节拍形态知识</strong>和<strong style="color:red">引导注意力RNN的节奏知识</strong>**

⭐**跨时域和频域进行注意力融合**

**节拍级**：主要考虑异常的波形或边缘。知识引导的注意力来聚合这些特征并获得节拍级别注意力

​				**卷积神经网络 (CNN) 用于学习节拍级别模式**

**节律级**：考虑异常节律变化

​				**循环神经网络（RNN）适合捕获节律特征**

提取特定级别的领域知识特征并使用它们来引导注意力，包括指导注意力 CNN 的节拍形态知识和指导注意力 RNN 的节奏知识

识别关键节拍位置、显着的节律变化、重要的频率分量



*知识特征 作用于 网络特征*

**特定级别的领域知识特征  - 引导注意力**

<div style='color:red; font-weight:bolder'>Beat Level</div>

​	 主要考虑异常波形或急剧变化的点 => 计算**一阶差分** Δ 和每个片段 s 上的**卷积**运算

​	一阶差分： 用来提取信号的变化趋势和特征

​	![image-20240320134815892](http://sthda9dn6.hd-bkt.clouddn.com/FmyxGSZaakCzhtV7kNZZN0i7B-tw)

<div style='color:red; font-weight:bolder'>Rhythm Level</div>

​	计算每个片段的**标准差**，以提取节奏水平知识特征向量 

​		***标准差***：**衡量信号的稳定性**、**检测异常值**、**描述信号波动性**、**比较不同信号之间的波动程度**

<div style='color:red; font-weight:bolder'>Frequency Level</div>

​	能量越大的信号包含的信息越多，**功率谱密度**（PSD）来提取频率级知识特征向量

​		频谱分析可以辅助了解信号在不同频率下的成分和能量分布情况

​		功率谱密度估计是计算信号功率在频域上的分布

​		例：100HzECG段数据 =>  periodogram(估计信号的功率谱密度的函数) => 返回频率范围(0-50Hz)和对应的功率值 => sum() 总频谱密度



**模型框架：**

![image-20240320130936524](http://sthda9dn6.hd-bkt.clouddn.com/Fvm0o2A8wCetOmgRDCYiUNPBrwsA)



**Input**:  单导联心电信号

**Frequency Transformation Layer**:  将信号按照频率区分开，利用高通滤波器、带通滤波器；  ***0-0.5Hz: 低频漂移；0.5-50hz：主要成分;  >50Hz: 噪声***

```python
### candidate channels for ECG
P_wave = (0.67, 5)
QRS_complex = (10, 50)
T_wave = (1, 7)
muscle = (5, 50)	# 肌肉干扰
resp = (0.12, 0.5)	# 呼吸信号
!

ECG_preprocessed = (0.5, 50) # ！！！ ECG主要部分
wander = (0.001, 0.5)	# 基线漂移
noise = 50

# low (wander), middle (ECG_preprocessed) and high (noise)
bandpass_list = [wander, ECG_preprocessed]
highpass_list = [noise]
```

![image-20240320133906384](http://sthda9dn6.hd-bkt.clouddn.com/FrR37eonOewex5q0nXqAJYTg6D4g)



**Sliding Window Segmentation**:  固定窗口，重叠分段（不用先定位R峰再分段）

![image-20240320134653351](http://sthda9dn6.hd-bkt.clouddn.com/FmCb242aCoG5ncSbExr9MPjd-J_1)

计算一阶差商、标准差、功率谱密度估计作为**统计特征**



**将统计信息作为注意力引导嵌入模型中：**

L： 经过Conv提取段内信息后的数据

H：经过RNN融合段间信息后的数据

![image-20240320141812190](http://sthda9dn6.hd-bkt.clouddn.com/FmGKfSedPYpDJXu9LswZJS-62TJW)

$$
V^{T} ∈ R^{1×D_{α}}
$$

再将α作用于特征

最后将不同频率区域的输出Concat 通道在一起，过频率注意力重新加权





---

### Self-supervised  - 2022

***Self-supervised representation learning from 12-lead ECG data***

*Computers in Biology and Medicine*



**问题：**

- 医学高质量标签很难获得，成本昂贵
- 公开数据集不够大
- 未标记数据的数量通常远远超过标记数据的数量

结合NLP、视觉、语音在无监督学习的成功

⬇️

**创新：**

- 使用自监督的方法，实现ECG表征学习



**对比学习CPC框架结构**

![image-20240416131704929](http://sthda9dn6.hd-bkt.clouddn.com/FofvehMrA5uriBI78cbBS5YbIShM)

为什么用MLP不用CNN，因为**ECG**采样频率为**100Hz**比**音频**典型采用频率**10 kHz**粗糙，使用MLP进行非线性映射





**数据集：**

![image-20240416131354561](http://sthda9dn6.hd-bkt.clouddn.com/FmyaN3BQaErrvtRqbXV81LJKpu8_)



**结果**

评价指标***macro AUC***

**线性评估**-冻结模型参数，将分类头改为线性层，验证模型学到的特征表示；

**微调**-在下游任务上进行微调分类头和部分参数。



![image-20240416132013141](http://sthda9dn6.hd-bkt.clouddn.com/FkrtZdqFeWsbHP7LvJmmgjWfRd6x)

***预训练的表示与下游分类任务高度相关***





***自监督预训练提高了下游分类器的稳健性***

![image-20240416134716985](http://sthda9dn6.hd-bkt.clouddn.com/Fja8j4BZTAsa91EHE0sUcc3tte6N)

表明在大数据集上预训练，在下游任务中，需要更少的标签数据就可以达到有监督训练的效果





---

### Multi-scale Progressive Gated Transformer 

Multi-scale Progressive Gated Transformer for Physiological Signal Classification



分两步：

- 细粒度捕获局部波形变化
- 粗粒度捕获全局趋势变化



框架：

<img src="http://sthda9dn6.hd-bkt.clouddn.com/FgyMITh5kc3J10LomW2rU9tqJxxD" alt="image-20240615220014092" style="zoom: 67%;" />



- 使用Conv-MaxPool-Conv-AvgPool 实现嵌入
- 堆叠 MHSA-FFN-TCN-FFN 模块 (Temporal Convolution Net)

- 分支融合：x1 ⨂ sigmoid(tanh(fc(x1))) + x2



---

### CLOCS - ICML 2021

*Contrastive Learning of Cardiac Signals Across Space, Time, and Patients*



对比：时间-空间-患者

![image-20240731153848210](http://sthda9dn6.hd-bkt.clouddn.com/FiAKTZPKBclwcqZ5m86_plzRT0wL)



![image-20240731153914807](http://sthda9dn6.hd-bkt.clouddn.com/FrZOif0kPis9FnlpxevBwTlrGRXA)

Figure：（左）对比多段编码、（中）对比多导联编码和（右）对比多段多导联编码中的K个实例的小批量的相似性矩阵。将基于所有应用的变换运算符TA和TB对生成附加矩阵。沿着边缘沿着示出了示例性变换的ECG实例。为了识别阳性对，我们将每个实例与其患者ID相关联。通过设计，对角元素（绿色）对应于同一患者，有助于等式2.类似地，实例1和实例50（黄色）属于同一患者，有助于等式（1）。3.蓝色区域对应于阴性示例，因为它们涉及来自不同患者的实例。



数据划分：

```python
frame = torch.tensor(input_frame,dtype=torch.float)
label = torch.tensor(label,dtype=torch.float)
frame = frame.unsqueeze(0) #(1,5000,12) #SxL = Samples x Leads
frame_views = torch.empty(1,2500,self.nviews*2) #nviews = nleads in this case (1x2500x12*nsegments)
nsegments = frame.shape[1]//2500
fcount = 0
for n in range(self.nviews): #nviews = # of leads
   for s in range(nsegments):
       start = s*2500
       current_view = frame[0,start:start+2500,n]
       current_view = self.obtain_perturbed_frame(current_view)
       current_view = self.normalize_frame(current_view)
       frame_views[0,:,fcount] = current_view  
       fcount += 1
# => (1, 2500, 24) 
```



⭐***CMSMLC(Contrastive Multi-segment Multi-lead Coding)***

Loss分析：

```python
pids = ['1', '2', '1', '3']  # patient id
data = [4, feature_dim, 4]   # batch, dim_feature, nviews

# 1. 先标记相同患者的样本

# 2. 计算各个视图的相似性
view_combinations = combinations(nviews,2)  
# (0,1), (0,2), (0,3)
#        (1,2), (1,3)
#               (2,3)
loss = 0
ncombinations = 0
for combination in view_combinations:
    view1 = data[:, :, combination[0]]
    view2 = data[:, :, combination[1]]
    norm1_vector = view1_array.norm(dim=1).unsqueeze(0)
    norm2_vector = view2_array.norm(dim=1).unsqueeze(0)
    sim_matrix = torch.mm(view1,view2.transpose(0,1))
    norm_matrix = torch.mm(norm1_vector.transpose(0, 1), norm2_vector)
    argument = sim_matrix / (norm_matrix * temperature)  # temperature=0.1
    sim_matrix_exp = torch.exp(argument)
    
    # obtain element
    triu_elements = sim_matrix_exp[rows1,cols1]  # upper triangle
    tril_elements = sim_matrix_exp[rows2,cols2]  # lower triangle
    diag_elements = torch.diag(sim_matrix_exp)   # 主对角
    
    triu_sum = torch.sum(sim_matrix_exp,1)
    tril_sum = torch.sum(sim_matrix_exp,0)
            
    loss_diag1 = -torch.mean(torch.log(diag_elements/triu_sum))  # A => B
    loss_diag2 = -torch.mean(torch.log(diag_elements/tril_sum))  # B => A
            
    loss_triu = -torch.mean(torch.log(triu_elements/triu_sum[rows1]))  # 上三角对应的行
    loss_tril = -torch.mean(torch.log(tril_elements/tril_sum[cols2]))  # 下三角对应的列
    
    loss = loss_diag1 + loss_diag2
    loss_terms = 2

    if len(rows1) > 0:
        loss += loss_triu #technically need to add 1 more term for symmetry
        loss_terms += 1

        if len(rows2) > 0:
            loss += loss_tril #technically need to add 1 more term for symmetry
            loss_terms += 1
            
ncombinations += 1
loss = loss/(loss_terms*ncombinations)  # loss/(4*6) 每个视图4份loss，总共算6对
```

![image-20240731205058921](http://sthda9dn6.hd-bkt.clouddn.com/FjYJ_QqmCMk7qjA-ylYR4ZJlLNsj)

![image-20240731204919944](http://sthda9dn6.hd-bkt.clouddn.com/FqAMVPcFHZsfJvQDYfgwW3l1h1g8)







---

### GUIDING MASKED REPRESENTATION LEARNING  - ICLR 2024



⭐心电图进行简单的数据增强也可能会严重改变病理信息

✔️利用MAE方法，生成式自监督学习



3种嵌入方式，***时间-空间-时空***

![image-20240731152623466](http://sthda9dn6.hd-bkt.clouddn.com/FlMb8O9N6mflZp08gDh4JC6fnKTd)



模型：

![image-20240731152710546](http://sthda9dn6.hd-bkt.clouddn.com/FmsW4LnhaLoqEoKTePQMa4II-s4q)

position embeddings： 使用同一组位置-共享

lead-embeddings:   每个导联专用-标记导联编号

\[SEP]: 区分各个导联Patch



为了增加重建难度：

1. Decoder部分只看同一导联的Patch；=> 确保重建时不会显式使用其他导联的嵌入 => 迫使模型有效地学习时空表示



代码分析：

```python
# x: [batch, 12, 2250]
x = series
# === forward_encoder ===
x = patch_embedding(x) # segment->LN->Linear->LN, => [batch, 12, 30, 75]
x = x + pos_embedding[:, 1:n + 1, :].unsqueeze(1)  # lead-inter shared pos embedding

# mask 
len_keep = int(n * (1 - mask_ratio))
ids_shuffle # [batch, 12, 30] 段编号打乱
ids_restore # 段编号原始顺序

ids_keep = ids_shuffle[:, :, :len_keep]
x_masked = torch.gather(x, dim=2, index=ids_keep.unsqueeze(-1).repeat(1, 1, 1, d))
mask = torch.ones([b, num_leads, n], device=x.device)
mask[:, :, :len_keep] = 0
# === x_masked, mask, ids_restore === 

# embedding 
x = torch.cat([left_sep, x, right_sep], dim=2)
lead_embeddings = lead_embeddings.unsqueeze(2).expand(b, -1, n_masked_with_sep, -1)
x = x + lead_embeddings  # lead-intra shared lead embedding

# Transformer Encoder x 12
x = encoder(x)

# === forward_decoder === 
x = self.to_decoder_embedding(x)  # 维度映射

# 初始化被mask掉的patch
mask_embeddings = self.mask_embedding.unsqueeze(1)
mask_embeddings = mask_embeddings.repeat(b, self.num_leads, n + 2 - n_masked_with_sep, 1)

x_wo_sep = torch.cat([x[:, :, 1:-1, :], mask_embeddings], dim=2)  # [X..,masked,..]
x_wo_sep = torch.gather(x_wo_sep, dim=2, index=ids_restore.unsqueeze(-1).repeat(1, 1, 1, d))  # 恢复位置

x_wo_sep = x_wo_sep + self.decoder_pos_embed[:, 1:n + 1, :].unsqueeze(1)  # 重新添加位置信息
left_sep = x[:, :, :1, :] + self.decoder_pos_embed[:, :1, :].unsqueeze(1) 
right_sep = x[:, :, -1:, :] + self.decoder_pos_embed[:, -1:, :].unsqueeze(1)
x = torch.cat([left_sep, x_wo_sep, right_sep], dim=2)

x_decoded = []
for i in range(self.num_leads):
    x_lead = x[:, i, :, :]
    for block in self.decoder_blocks:  # Transformer Encoder x 6
        x_lead = block(x_lead)
        x_lead = self.decoder_norm(x_lead)
        x_lead = self.decoder_head(x_lead)
        x_decoded.append(x_lead[:, 1:-1, :])
pred = torch.stack(x_decoded, dim=1)

# === loss ===
target = self.patchify(series)

loss = (pred - target) ** 2
loss = loss.mean(dim=-1)  # (batch_size, num_leads, n), mean loss per patch

loss = (loss * mask).sum() / mask.sum()  # mean loss on removed patches
# mask: [batch, 12, 30], 其中1代表被mask，0表示没有被mask
```



*微调时取Encoder部分*



---





[SEP]: SEP	"Segmenting Patch"





---



## 实验

### InceptionFormer

***InceptionNeXt + SMT***

> 图示仿照SMT风格绘制

- 仿照SMT的渐进融合网络结构
- 前半部分使用 InceptionNeXt结构块，DConv卷积代替MHSA

```yaml
# 模型 FLOPs: 2.14G
# 模型参数数量: 12.45M

cnnblock = StemConv()                  # 1, 12, 1000
# 模型 FLOPs: 36.74M
# 模型参数数量: 36.72K
CNNBlock1 = getattr(model, 'block0')   # 1, 64, 1000
# 模型 FLOPs: 35.07M
# 模型参数数量: 35.07K
CNNBlock2 = getattr(model, 'block1')   # 1, 128, 500
# 模型 FLOPs: 135.68M
# 模型参数数量: 271.36K
MixBlock = getattr(model, 'block2')    # 1, 256, 250
# 模型 FLOPs: 793.09M
# 模型参数数量: 3.18M
MHSABlock = getattr(model, 'block3')   # 1, 512, 125
# 模型 FLOPs: 1.05G
# 模型参数数量: 8.41M
```

#### 模型框架



![image-20240307212600434](http://sthda9dn6.hd-bkt.clouddn.com/FsNnhqajz8TUXNFuHkjNO7x2TQ6r)

<center style="font-weight:bolder; color:rgb(4A,4A,4A);">模型框架</center>

**Stem 模块：**

> 参考：A token selection-based multi-scale dual-branch CNN-transformer  一区论文

标准一维卷积，多尺度提取特征，提取ECG形态信息。***全面***提取（携带冗余信息）  # shape: 12, 1000  => 128，1000

![image-20240307212809248](http://sthda9dn6.hd-bkt.clouddn.com/Fu16qcDleveylB7vzaWnL5xUGg9d)

**Block：**

![image-20240307213204707](http://sthda9dn6.hd-bkt.clouddn.com/Fq3DUteSU_WSdplw7JfmABvNFYbc)

**Patch Fusion:**

​	使用Conv1D，kernel_size=3，stride=2, padding=1   => 重叠式嵌入，减少Token数量，增加Token维度(增加Token携带信息的丰富度)



*图示：* ①Token间  ②Token内

![image-20240312132746778](http://sthda9dn6.hd-bkt.clouddn.com/FixTAltVtt8JDqNcAB9Cd44Tk2mV)

#### 自注意力

*在InceptionFormer上实验*

- **Multi Head Self Attention**

```yaml
Test	  Best F1: 0.8410 
Validate  Best F1: 0.8184 
```

- **SR-MHSA**  使用Conv1D stride=2 缩小K，V的TokenMap，减少Token数量

  *sr-ratio = 2*

```yaml
Test	  Best F1: 0.8137
Validate  Best F1: 0.8216
```

- **Bi-Routing Attention**  -- SR与标准的折中

  超参：num_window=25，topk=10

```yaml
Test	  Best F1: 0.8344
Validate  Best F1: 0.8199
```

- **Window Attention**  -- 单纯窗口级	❌多余的版本 - 不如MobileViTBlock

❗没加CNN先局部融合

```yaml
Test	  Best F1: 0.8261
Validate  Best F1: 0.8148
```

- **Shift Window Attention**

***感觉有点问题，效果不如 WA，混合结构的问题？前面进行的交错的CNN和MHSA？***

```yaml
Test	  Best F1: 0.8129
Validate  Best F1: 0.8100
```

- **级联删除Token和修剪Head**

去掉冗余的 ❗***未测试***

#### 位置嵌入

- **前期**直接融进Token中  -- ViT *仅一次*

​		例：data: [batch, num_token, dim_token]    learnable_pos_embed: [batch, num_token, dim_token]

​		data + pos_embed

- 注意力**计算中**引入         -- SwinT  *每次*

​		例： att:[num_token, num_token]  + relative_pos_bias:[num_token, num_token]

​		让相对位置信息(自学习)影响注意力权重

- DWConv 引入局部位置信息 / 在计算完自**注意力前**     -- Bi-Routing-Attention  *每次*	

​		例： DWConv(x) + EncoderBlock(x)

- DWConv增强位置信息 / 在计算完自**注意力中**     -- Bi-Routing-Attention  *每次*	

​		例： SoftMax(Q@K.T/√d)@ (V + DWConv(V) )



#### **现存问题**

- ⭐没有为MHSA计算时**添加位置信息**

  - DWConv 代替位置编码
  - MHSA之前加入可学习的位置向量
- 渐进式模块融合策略

  - 交错
  - 并行？
- 没有优化自注意力的计算
- DConv的优化





---

### InceptionFormer + 卷积调制

***InceptionNeXtBlock => ConvModulation => MHSA***

卷积 => 卷积×调制 => 调制×自注意力 => 自注意力

 x2              x4                       x8                       x4



```python
acc: 0.8299, f1: 0.8418, macro_auc: 0.9679
classes:  ['I-AVB'  'AF'   'LBBB'   'RBBB'  'NORM'  'PAC'   'STD'  'STE'   'PVC']
test f1s: [0.9252, 0.9231, 0.902, 0.9477, 0.8261, 0.6733, 0.8161, 0.7805, 0.782]
```



**全局平均池化 => CLS Token**

❗Validate F1 score only 0.8185	# 稳健性一般

```python
acc: 0.8343, f1: 0.8488, macro_auc: 0.9677 threshold: 0.8
classes:   ['I-AVB'  'AF'   'LBBB'   'RBBB'  'NORM'  'PAC'   'STD'  'STE'   'PVC']
F1 Scores: [0.9028, 0.9587, 0.9412, 0.9444, 0.7861, 0.7308, 0.8046, 0.7895, 0.7813
```

❌**PAC  仍然改善不了**



#### 注意力可视化

![image-20240321214200346](http://sthda9dn6.hd-bkt.clouddn.com/Fr9n4nyTsCBn5iEaSrXymPMghq5e)

![image-20240321214221728](http://sthda9dn6.hd-bkt.clouddn.com/Fq7kGrJi1Zr4ArjVKTra1OBKCO4v)

![image-20240321214245731](http://sthda9dn6.hd-bkt.clouddn.com/Fpzwlg1dLDAUDaNWNS6nednn_qPs)![image-20240323153412375](http://sthda9dn6.hd-bkt.clouddn.com/FlC1-fpMQR9PBZW4hZwHUEUYlkFw)

![image-20240323153505140](http://sthda9dn6.hd-bkt.clouddn.com/FqxORjFqQpnvQPjU5bw4pHZZTWkG)



### ***InceptionFormer + Shunt***

***Shunt Self Attention => InceptionFormer***

**在多头部分嵌入多尺度**。8头自注意力， 1/2头执行像素级自注意力，1/4头执行下采样2倍的块级自注意力，1/4头执行下采样4倍的域级自注意力

![image-20240321215459046](http://sthda9dn6.hd-bkt.clouddn.com/FjM0Qp7pfXVvJTZ7EzvwhckVNV67)

**在计算自注意力时，实现多尺度**

```python
acc: 0.8154, f1: 0.8389, macro_auc: 0.9682
classes:   ['I-AVB'  'AF'   'LBBB'   'RBBB'  'NORM'  'PAC'   'STD'  'STE'   'PVC']
F1 Scores: [0.9065, 0.9283, 0.8511, 0.9296, 0.7865, 0.6531, 0.8193, 0.8421, 0.8333]
```

❗**PAC**

![image-20240323152854941](http://sthda9dn6.hd-bkt.clouddn.com/FpEiiDzmrebCmlBWg6NzE0mvsE0h)

![image-20240321221939637](http://sthda9dn6.hd-bkt.clouddn.com/FvS0TbuQv2iuecYcp_3YCOdaReWG)

![image-20240321221958275](http://sthda9dn6.hd-bkt.clouddn.com/FhOpvWHbC-0OYZbH2e3HeJfOe_0H)

![image-20240321222022287](http://sthda9dn6.hd-bkt.clouddn.com/Frz6qYKV9L3zNIT00capxCx2KJIN)





---

**InceptionFormer: MS-DWConv + MHSA**

```python
验证集: 9-Fold, 测试集: 10-Fold.                                                                               
                                                               
Acc: 0.8328, F1: 0.8530, Auc: 0.9701, Threshold: 0.4, Test_loss: 0.2225                                        
F1s: ['0.8936', '0.9219', '0.9200', '0.9505', '0.8298', '0.6972', '0.8315', '0.8000', '0.8321']                
                                                                                                               
Acc: 0.7918, F1: 0.8247, Auc: 0.9564, Threshold: 0.9, Vali_loss: 0.2764                                        
F1s: ['0.8971', '0.9160', '1.0000', '0.9020', '0.7831', '0.6981', '0.7600', '0.6154', '0.8504']                
  
# 此处，仅取平均，最终应该组合在一起进行测试，因为阈值不一致
Acc: 0.8123, F1: 0.8388, Auc: 0.9633  Average Test and Validate                                                
F1s: ['0.8953', '0.9189', '0.9600', '0.9263', '0.8064', '0.6977', '0.7957', '0.7077', '0.8413']
    
# 模型 FLOPs: 2.14G
# 模型参数数量: 12.45M
```

**InceptionFormer: MS-DWConv + S-MHSA**

❌数据敏感

```python
验证集: 9-Fold, 测试集: 10-Fold.

Acc: 0.8125, F1: 0.8204, Auc: 0.9686, Threshold: 0.6, Test_loss: 0.2426
F1s: ['0.8467', '0.9143', '0.8800', '0.9344', '0.8021', '0.6355', '0.8452', '0.7368', '0.7883']

Acc: 0.7918, F1: 0.8163, Auc: 0.9556, Threshold: 0.8, Vali_loss: 0.2814
F1s: ['0.8841', '0.8934', '0.9778', '0.9091', '0.7340', '0.7037', '0.7925', '0.5946', '0.8571']

Acc: 0.8022, F1: 0.8183, Auc: 0.9621  Average Test and Validate
F1s: ['0.8654', '0.9039', '0.9289', '0.9218', '0.7681', '0.6696', '0.8188', '0.6657', '0.8227']
```

**InceptionFormer: MS-DWConv-SE + MHSA**

自动重新校准  - 通道重要性

```python
Validate: Acc:0.788937409	F1:0.819458854	
Test    : Acc:0.827034884	F1:0.835588466
```

**InceptionFormer: MS-DWConv-CA + MHSA**

同时校准 通道 + 空间 重要性

```python
Validate: Acc:0.7991	F1:0.8315
Test    : Acc:0.8096	F1:0.8194
```

**InceptionFormer: only front 3 stage**

删除第四stage，降低参数

```python
# Best F1: 0.8395  Only front 3 layer
# 模型 FLOPs: 1.04G
# 模型参数数量: 3.64M
```

**InceptionFormer:  stage depth [2, 4, 8, 4] => [2, 4, 8, 2]**

减少stage4层数

```python
# stage depths = [2, 4, 8, 2]
# Best F1: 0.8422
# 模型 FLOPs: 1.61G
# 模型参数数量: 8.25M
```

**InceptionFormer + SparseSemanticToken**

*减少Token*

```python
# Best F1: 0.8312  sparse token -- DW-SpatialPool
# 模型 FLOPs: 1.53G
# 模型参数数量: 17.19M

SparseAttentionBlock:
# 模型 FLOPs: 236.95M
# 模型参数数量: 4.74M
```

**InceptionFormerTiny** 

*减少Token长度  dim_token: [64, 128, 256, 512] => [32, 64, 128, 256]*

```python
# Best F1: 0.8265
# 模型 FLOPs: 536.07M
# 模型参数数量: 3.13M
```

⭐⭐⭐

**Baseline**

```python
# resnet18
模型 FLOPs: 176.57M
模型参数数量: 3.85M
Best F1: 0.8168

# resnet34
模型 FLOPs: 357.74M
模型参数数量: 7.23M
Best F1: 0.8185
    
# mobilenetv2
模型 FLOPs: 96.60M
模型参数数量: 2.19M
Best F1: 0.7967	
```







---

**PTB-XL**  - 推荐9折验证 10折测试

**CPSC**  - (比赛) 9折验证 10折测试



---

**优化器调整**

Adamw + StepLR  => Adamw + CosineAnnealingLR

固定步长衰减学习率  => 余弦退火调整学习率



Adamw + StepLR：

```python
# scheduler
lr = 1e-3
step_size = 20
step_gamma = 0.1
```

Adamw + CosineAnnealingLR

***PASS***





---

### 频域特征

数据的另一种表达形式，或者在这个格式中，某些特征会被放大，使得可以被识别到



### time domain

![image-20240422221233452](http://sthda9dn6.hd-bkt.clouddn.com/Fh5jqjyTnEuzDskq6BThKTGZmZD6)

|        | I-AVB  | AF     | LBBB   | LBBB   | NORM   | PAC    | STD    | PVC    | STE    | 平均       | 描述                |
| :----: | ------ | ------ | ------ | ------ | ------ | ------ | ------ | ------ | ------ | ---------- | ------------------- |
| **F1** | 0.8844 | 0.9231 | 0.8800 | 0.9500 | 0.7514 | 0.6392 | 0.8024 | 0.7692 | 0.7669 | **0.8185** | 时序信号 & 一维模型 |



### frequency domain

![image-20240422221515722](http://sthda9dn6.hd-bkt.clouddn.com/FhOazUABP1FEUBFSsm73IMKpRG1U)

```python
# [batch_size, num_leads, data_length]  =STFT=> [batch_size, num_leads, W, H]
# => resnet34
# => output
```

|        | I-AVB  | AF     | LBBB   | LBBB   | NORM   | PAC    | STD    | PVC    | STE    | 平均       | 描述                                    |
| :----: | ------ | ------ | ------ | ------ | ------ | ------ | ------ | ------ | ------ | ---------- | --------------------------------------- |
| **F1** | 0.7763 | 0.9180 | 0.8511 | 0.8889 | 0.7684 | 0.6393 | 0.7500 | 0.7671 | 0.7805 | **0.7933** | 时序信号通过STFT转为频谱图(汉明窗口100) |



### two-stream: time and frequency domain

![image-20240422214551861](http://sthda9dn6.hd-bkt.clouddn.com/FqxAImdtxGqN53vY_GDxfwBhxXMO)

|        | I-AVB  | AF     | LBBB   | LBBB   | NORM   | PAC    | STD    | PVC    | STE    | 平均       | 描述                                |
| :----: | ------ | ------ | ------ | ------ | ------ | ------ | ------ | ------ | ------ | ---------- | ----------------------------------- |
| **F1** | 0.8993 | 0.9255 | 0.9020 | 0.9326 | 0.7817 | 0.7040 | 0.8000 | 0.7805 | 0.7619 | **0.8319** | 时序信号通过STFT转为频谱图(窗口100) |





### InceptionNeXt + Shunted-Self Attention （Cross-Stack）

1️⃣

![image-20240511210910969](http://sthda9dn6.hd-bkt.clouddn.com/FkLWFrtOPtNHBGo55bV9mAsYVCbv)

***Inception depthwise convlution***

2️⃣ 

![image-20240511211011416](http://sthda9dn6.hd-bkt.clouddn.com/Fjr6plVqNa0GART9yPBiBdJMP-c7)

***Self-Attention 内部实现粗细粒度***

3️⃣ 

![image-20240511211237019](http://sthda9dn6.hd-bkt.clouddn.com/FiKUcUVgiO_MBNzz1xtBXICK4OCN)

***特征图  => 金字塔结构***  （中间CNN和Transformer融合，交叉堆叠）



```python
classes:  
     ['I-AVB'   'AF'      'LBBB'    'RBBB'    'NORM'    'PAC'     'STD'     'STE'     'PVC']

        
Test set:  
Acc: 0.9706, F1: 0.8547, Auc: 0.9666, Threshold: 0.5, Test_loss: 0.1813
Acc: ['0.9797', '0.9782', '0.9913', '0.9738', '0.9506', '0.9549', '0.9593', '0.9913', '0.9564']
F1s: ['0.9028', '0.9388', '0.8846', '0.9511', '0.8132', '0.7207', '0.8391', '0.8500', '0.7917']
    
Validate set:
Acc: 0.9596, F1: 0.8105, Auc: 0.9657, Threshold: 0.5, Vali_loss: 0.2111
F1s: ['0.8750', '0.9333', '0.9787', '0.9016', '0.6989', '0.6504', '0.7545', '0.6818', '0.8201']
Train[650        1099      212       1675      826       554       782       198       629]
Acc: ['0.9738', '0.9767', '0.9985', '0.9476', '0.9185', '0.9374', '0.9403', '0.9796', '0.9636']
```



***CPSC***  - **患者间**

训练集包含 6,877 个（女性：3178 个;男性：3699 个）12 导联心电图记录，持续时间从 6 秒到 60 秒不等。

![image-20240511213646105](http://sthda9dn6.hd-bkt.clouddn.com/FhIixT4TQjVJ9B1r2ZEnuln2pA5b)

论文中是 1-8 训练；9 验证；10 测试；



***PTB-XL***  - 患者间

包含来自 18885 名 10 秒长度患者的 21837 个临床 12 导联心电图

![image-20240511214058996](http://sthda9dn6.hd-bkt.clouddn.com/FiBLlDeJjXN_q5PJnTh5NOaZXFEL)

⭐ 特定患者的所有记录都分配给同一折。折 9 和 10 中的记录至少经过一次人工评估，因此具有特别高的标签质量。因此，我们建议使用 1-8 折 作为训练集，折叠 9 作为验证集，折叠 10 作为测试集。



***Chapman-Shaoxin ECG Data***  - 患者间

包含 10,646 名患者的 12 导联心电图，采样率为 500 Hz，11 种常见心律

![image-20240513120253717](http://sthda9dn6.hd-bkt.clouddn.com/Fgm3PAj0mAdD6TRZ_ykzlYyXSOMJ)

类别分组：

![image-20240513195732015](http://sthda9dn6.hd-bkt.clouddn.com/Fk_uTCl4aFYJqgBY6Be9yjLslIL_)



**Result**

| dataset | Task                      | F1_macro | AUC_macro |
| ------- | ------------------------- | -------- | --------- |
| CPSC    | arrhythmia \| multi-label | 0.8547   | 0.9666    |
| PTB-XL  | arrhythmia \| multi-label | 0.5370   | 0.9552    |

| dataset | Task                      | AUC_macro | F1_macro | Accuracy | Recall | Precision |
| ------- | ------------------------- | --------- | -------- | -------- | ------ | --------- |
| chapman | arrhythmia \| multi-class | 0.9966    | 0.9660   | 0.9699   | 0.9652 | 0.9671    |



multi-label:  sigmoid转0-1区间，用**阈值**进行预测值二值化，生成标签

multi-class:  softmax转概率，取概率最大的下标作为预测标签

 

数据增强：

1. 随机mask某导联

2. 随机mask某段(仅开头或结尾)

3. 增加随机噪声

   *没啥用*



---



















## 汇总

### 实验结果

<img src="http://sthda9dn6.hd-bkt.clouddn.com/FlNas9rDgTfpFaQkuM2RTtL4qJGn" alt="image-20240603201428685" style="zoom:80%;" />

1️⃣ CNN：优势=>强大的提取局部模式的能力； => 提出局部细节，边缘，纹理模式信息。  ⭐ 高频信息

​                  不足=>需要堆叠很深，感受范围才能延申到全局； => 感知全局弱势

2️⃣ Transformer： 优势=>全局感受野，不够整体外貌能力很强;                                              ⭐ 低频信息

​								  不足=>看的太多，干扰太多，对局部细节模式捕获能力较弱。



### 模型插图

- **Block基本结构**

<img src="http://sthda9dn6.hd-bkt.clouddn.com/FrM-MsmNgETP653AMr5X6U3uZeqD" alt="image-20240603173800170" style="zoom:67%;" />



- **InceptionNext**

![image-20240603173849448](http://sthda9dn6.hd-bkt.clouddn.com/FnoK7APToyLRLSNOXXRpdQD5webA)

*DConv: 根据通道分组，每组使用不同卷积核大小的DWConv*



- **InceptionMHSALinear**

![image-20240603140051484](http://sthda9dn6.hd-bkt.clouddn.com/FjOQLnqM8UaLfoCO-p21jBmkb8my)

*Local Block 和 Global Block 线性拼接*



- **InceptionFormer**

![image-20240603140214408](http://sthda9dn6.hd-bkt.clouddn.com/Fo31v6MWxV47NjR4fvosmD_-NzCh)

*Local Block 和 Global Block 交叉堆叠进行融合*



- **InceptionShuntSA**

![image-20240603163710962](http://sthda9dn6.hd-bkt.clouddn.com/Fuv6HWHKdoVULmpy0H2qcFMfaK-C)

<img src="http://sthda9dn6.hd-bkt.clouddn.com/Fk8GpNJFjqD7ISe9cjShaklzAOXx" alt="image-20240603163728815" style="zoom:80%;" />

<img src="http://sthda9dn6.hd-bkt.clouddn.com/Fp25ZJWBfu-R4-2-AZe_RoRAkYmg" alt="image-20240603163748074" style="zoom:80%;" />

*一半注意力头执行细粒度注意力，另一半头执行粗粒度注意力*



- **InceptionHiLoSA**

![image-20240603172844194](http://sthda9dn6.hd-bkt.clouddn.com/Fg2w7c8N-CkizyyzMEuICS2x0JKa)

<img src="http://sthda9dn6.hd-bkt.clouddn.com/FomolO0EP8QM7lp00ZEgOwJgKP0F" alt="image-20240603172708413" style="zoom: 67%;" />



*一半注意力头执行【窗口级注意力】，另一半头执行【池化注意力】*

🧐要调整窗口大小 ！



- **SandwichNet**

<img src="http://sthda9dn6.hd-bkt.clouddn.com/FvjbaounpDcp9XcvNUi0Xcctym7X" alt="image-20240603164045740" style="zoom:80%;" />





- **InceptionFormer, M-DWConv 加入倒残差结构 ** ***(Inverted residual block)***

<img src="http://sthda9dn6.hd-bkt.clouddn.com/Foj-NHXzzmFBZgXmB_JEIfmvb1Wc" alt="image-20240603202132653" style="zoom:50%;" />

*在多尺度DWConv前嵌入Conv1×1 扩张通道数，后嵌入Conv1×1 进行降维。*

***风格非常统一***  ***=> 简单***



- **SandwichProNet**

<img src="http://sthda9dn6.hd-bkt.clouddn.com/Fgs-6JEzvvush1Voc2ORJeK1KHtO" alt="image-20240611122337229" style="zoom:67%;" />



- **Channel Mixer**：GLU， DW-FFN

- **Token Mixer:**  细粒度局部 + 粗粒度全局

- **堆叠方式**





## 数据

***Arrhythmia Task***

*10s + 100Hz sampling rate*



***PTB-XL  - Rhythm Task*** 

```python
['AFIB', 'AFLT', 'BIGU', 'PACE', 'PSVT', 'SARRH', 'SBRAD', 'SR', 'STACH', 'SVARR', 'SVTAC', 'TRIGU'] 
- 12 class

[ 1362  66  74  266  22  695  573  15074  744  143  24  18]  # Train
[ 152    7   8   28   2   77   64   1674   82   14   3   2]  # Test

Train: 18932
Test : 2098
Total: 21030
```



***Chapman***

```python
['AFIB', 'GSVT', 'SB', 'SR'] 
- 4 class

Train [1996. 2069. 3498. 2000.]
Test  [222.  231.  389.  222. ]
    
Train: 9563
Test : 1064
Total: 10627
```



***CPSC-2018***

```python
['I-AVB'  'AF'  'LBBB'  'RBBB'  'NORM'  'PAC'  'STD'  'STE'  'PVC']
- 9 class

Train [ 650 1099 212 1675 826 554 782 198 629]
Test  [  72  122  24  181  92  61  87  22  70]

Train: 6187
Test : 688
Total: 6875
```

