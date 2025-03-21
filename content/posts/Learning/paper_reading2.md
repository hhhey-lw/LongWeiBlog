---
author: "LongWei"
title: "深度学习论文汇总2"
date: "2024-04-19"
tags: ["DL"]
categories: ["Learning"]
series: ["Learning"]
aliases: ["dl_papers2"]
ShowToc: true
TocOpen: true
---



### **CAM**  - CVPR 2015

*Learning Deep Features for Discriminative Localization*  

**弱监督对象定位**  - 仅提供Image level label

期望：每个单元被其感受野内的某种视觉模式激活。因此 fk （表示空间位置 (x, y) 处最后一个卷积层中单元 k 的激活//输出特征图的一个像素）是该视觉模式存在的地图。类激活图只是这些视觉模式在不同空间位置的存在的加权线性和

**计算卷积特征图对于特定输出单元的重要性来实现的**



⭐⭐⭐网络可以保留其卓越的定位能力，直到最后一层   => 深层特征的定位能力

❗❗❗尽管接受了图像级标签的训练，CNN 仍具有出色的对象定位能力



**缺陷**：卷积特征图→全局平均池化→softmax层  // 特定网络结构

![image-20240322134603157](http://sthda9dn6.hd-bkt.clouddn.com/FizSzlSfuX9fZOO_3HduKDclEzcI)

<center style="color: red; font-weight: bold;">做法图示</center>

![image-20240322134441902](http://sthda9dn6.hd-bkt.clouddn.com/FiBRel9zde6IJ3STCIWgQtDspaZx)

<center style="color: red; font-weight: bold;">数学公式</center>

在卷积特征图上执行全局平均池化，并将它们用作全连接层的特征，产生所需的输出分类;

❗❗❗将输出层的权重投影回卷积特征图来识别图像区域的重要性

---





### Grad-CAM  - ICCV 2017

适用CNN模型

但论文提到在CNN+LSTM的也能定位有区别的图像区域

![image-20240322141249072](http://sthda9dn6.hd-bkt.clouddn.com/FjU70VichK5n125Mrm0J9kJnVy43)



![image-20240322141305661](http://sthda9dn6.hd-bkt.clouddn.com/FpssA7XV30OJ4BKmn1aDL-1GNYSa)

α 捕获**特征图** k 对于目标**类** c 的**重要性**  // 与CAM的分类线性层权重作用一致

![image-20240322141314248](http://sthda9dn6.hd-bkt.clouddn.com/FoLeKuhJnfw8veLYf18fRWRhEy09)

**ReLU**的作用，只对对感兴趣的类别有积极影响的特征感兴趣。负像素可能属于图像中的其他类别

*上述操作 => 具有**类别区分性**并且可以很好地定位相关图像区域*   - 最后特征图比较小!

					  但缺乏显示细粒度重要性的能力 （能区分猫狗，但对为什么识别为猫，不够精确）



通过点乘法融合 **引导反向传播** 和 **Grad-CAM** => 可视化

Grad-CAM : 类别区分性

Guided Backprop： 细节纹理重要程序。 做法：将梯度值小于等于零的部分置为零，保留梯度值大于零的部分 => 以突出输入图像中对预测结果有积极影响的区域，来实现对神经网络中每个像素对最终预测结果的影响进行可视化和解释



**浅层卷积关注纹理特征，深层网络关注本质的那种特征**？

---

### DETR  - ECCV 2020

***⭐End-to-End⭐*** Object Detection with Transformers

传统：设置锚框 + 非极大值抑制(去除多余的框)

创新：集合预测(预测分类 + 锚框)



![image-20240322202435323](http://sthda9dn6.hd-bkt.clouddn.com/FuvmZgAnU9ncG2a5qW5EUCNP48H1)

<center style="color: red; font-weight: bold;">前向流程</center>

![image-20240322202514460](http://sthda9dn6.hd-bkt.clouddn.com/FgwVYzzA9oyIF86-03QbkEQU8bms)

<center style="color: red; font-weight: bold;">模型框架</center>

CNN backbone  - local information fusion

Transformer encoder  - global information fusion

object queries  - learnable information vector   作用： => anchor

***FFN:  1. classification => output class vector  2. box => output 4 number [center_x, center_y, width, hight]***



object queries: 作用就是锚框，并且一次性生成100个 >> 图片检测的物体数

如何将object queries 与 groundtrue一一对应？

匈牙利算法 寻找最佳匹配

匹配loss = 分类loss(分类正确率) + 框loss(框的重叠度)

=> 匈牙利算法 => 哪些框与GT最佳匹配(预测框与GT一一对应)

最终回传梯度优化参数LOSS = 分类loss(分类正确率) + [框loss + 与框大小无关的iou loss]

因为框也是生成的，且Transformer容易出大框(全局建模)





---

### Bert  -  2018

**BERT: Pre-training of Deep Bidirectional Transformers for Language Understanding**   *- Computation and Language*

Bidirectional Transformers 意思是

Transformer Encoder中的自注意力计算是全局的，每个Token能观测到其余的Token序列； 而

Transformer Decoder中由于进行的是Masked Multi Head Self Attention，所以序列只能观测到自己与之前的语境；

这对于文本上下文语境建模是有弊端的！

(生成式无监督学习)

**总体结构**

![image-20240327160149024](http://sthda9dn6.hd-bkt.clouddn.com/Fl4-SrUiEyrZ5ep1hsySNk5hJ6Yz)

[CLS] 分类Token，凝聚全局语义信息

[SEP] 分割符，划分句子范围

用的某个语料库进行词嵌入

![image-20240327202952064](http://sthda9dn6.hd-bkt.clouddn.com/FtNYoVsEqoeq74FY2SO4dBDE2WyO)



⭐**训练方法**

- 完型填空：使某个词随机被 [Mask] 字符遮挡；为防止模型对[Mask]字符敏感，遮挡时使用概率遮挡 => 1. 仍替换为[Mask]字符 2.随机替换为其他字符 3. 保持不变 => 迫使模型学习上下文语境   <strong style='color:blue;'>[句子内信息建模]</strong>

- 预测下一句：  <strong style='color:blue;'>[句子间信息建模]</strong>

**丰富的上下文信息**：通过考虑单词的左右上下文，BERT 能够更好地理解词义和句法结构，这对于理解语言的复杂性至关重要。





---

### GPT

generative pretrain transformer

*初略版*



模型图：(Transformer Decoder -仅Masked Attention版)

![image-20240505163406185](http://sthda9dn6.hd-bkt.clouddn.com/FkZKmDcVtOr6qyohAj5rVJdX6s7T)

⭐ *input:*  word => index(查找词汇表) => embedding   +  position embedding

[batch, sequence_length, embedding_dim]	 # sequence_length 可由多个句子组成，可以使用\<s>标识句子结束，\<pad>用来填充，使得sequence_length在batch内长度一致

⭐ *train：* 

x1 => x2

x1, x2 => x3

x1, x2, x3 => x4

因为每个词都要预测下一个词，故使用masked attention （mask-softmax），防止答案泄漏





---

### CLIP  - 2021

**Learning Transferable Visual Models From Natural Language Supervision**

**实现zero-shot，上游大数据集预训练好，下游任务迁移学习无需样本微调**

**多模态模型的总体目标就是：训练一个模型，一方面能统一特征表达，另一方面又能让不同模态特征间学到相关性**

(判别式无监督学习)

#### 工作目的

痛点：

- 在特点数据集上进行标签训练 => 输入没见过的类别，那么模型就不能输出正确的结果

- 数据出现分布偏移，动物图片与卡通动物图片 => 识别不出来



图片 - 文字描述 ， **模型学习配对关系**



*一个对象的不同视角表示：图片和文本描述*

#### **对比学习方法** **（Train）**

![image-20240327200843127](http://sthda9dn6.hd-bkt.clouddn.com/Fu1kXHXwvqyJvl-W0nklKpmttZ44)

1.  Text Encoder (resnet/vit) 学习文本描述的 深度特征  - 单模态内特征 // T_i == 一个文本特征
2.  Image Encoder(transformer) 学习图片的 深度特征  - 单模态内特征  // I_i == 一个图像特征
3.  将多模态特征投影到跨模态空间  // 矩阵映射，(特征向量)到同一纬度
4.  计算余弦相似度，很明显，正确配对的位置为对角线
5.  计算Loss： （相似度logit作为预测分数）
   1.  **按行计算Loss**，在每一行范围内做softmax，然后计算cross_entropy（蓝色格子部分是真值）。这样计算Loss的意义是：对于每一张图片，我们都希望找到和它最相似的文字
   2.  **按列计算Loss**，在每一列的范围内做softmax，然后计算cross_entropy（蓝色格子部分是真值）。这样计算Loss的意义是：对于每一段文字，我们都希望找到和它最相似的图片
   3.  **最后将这两个Loss相加取平均**，代表我们在模型优化过程中**考虑了“图片->文字”和“文字->图片”的双向关系**

encoder均从头开始训练



#### 预测

![image-20240327201936442](http://sthda9dn6.hd-bkt.clouddn.com/FhcxDiV_Ii24QNK9mAIQFKmvrVAK)

- 创建一个标签全集， [f'A photo of a {object}' for object in dataset_labels]
- Text Encoder 学习上述 模板文字描述 的 深度特征
- Image Encoder 学习 待预测图片 的 深度特征
- 计算余弦相似度 => 取最高分作为预测目标

	

#### 缺点

- 每次预测需要构建标签全集描述
- 对抽象任务，性能较差





---

### Two-Stream  - 2014

**Two-Stream Convolutional Networks for Action Recognition in Videos**  



⭐先前工作中通过使用堆叠视频帧作为网络的输入来解决此任务，但结果明显比最好的手工制作的浅层表示差。 => 这可能表明学习的时空特征不能很好地捕捉运动（简单堆叠 => 让模型从庞大的数据中学习❌很难）  => 模型很难识别该类特征 => **预先处理成模型擅长的数据形式**

❗❗❗虽然多帧信息很重要，但以**适当的方式**将其呈现给 ConvNet 也很重要（饱和）

信息显式建模  => 可以简化学习过程



*创新点*

- 空间流从静止视频帧执行动作识别
- 时间流以识别密集光流形式的运动动作

=> 贴合人类视觉：识别物体  +  识别运动  => ⭐[结果角度]表明两个识别流是互补的



光流：帧帧之间像素变化(描述运动变化的数据形式) | 水平方向和垂直方向  **<= 有用的线索**  (与魔改模型不一样的思路)

⭐⭐⭐此类输入**明确描述**了视频帧之间的**运动** => 这使得识别更容易 => 因为网络不需要隐式估计运动  ***（显示建模）***

可以从**光流数据** => **时空局部**特征 || **运动学**特征  || 运动是使用光流位移场**明确表示**的





**模型框架**（双模态）

![image-20240328154319036](http://sthda9dn6.hd-bkt.clouddn.com/Fv9O6s0-ISjzOZYNWYnsWeFDMD9H)



**提高模型迁移能力**   类似于CLIP的目的

考虑将两个数据集合并为一个，由于类集之间的交叉，这并不简单

=>  多任务学习  => 最后生成多个分类头，对应两个数据集分类。混合多个数据集进行实验





---

### MOCO  - CVPR 2020

***Momentum Contrast for Unsupervised Visual Representation Learning***

(判别式无监督学习)

***目标：***

	1️⃣ 构建一个足够大的动态词典，包含足够多的负样本，使模型真能够学到判别式的特征; 在海量数据中学到真正的样本分布
	
	2️⃣ 因为词典是动态变化的，为了使词典中的负样本特征尽可能的保持一致性(模型参数不同，时间维度上，得到的特征向量存在不一致性)，提出动量更新 => 动量模型的缓慢更新确保了字典中的特征相对稳定，从而提供更一致的负样本，提升对比学习的效果。
$$
θ_{k} ←mθ_{k} + (1 −m)θ_{q}
$$
m ∈ [0, 1) 是动量系数。论文中Query Encoder 和Key Encoder是一样配置架构的编码器。  目的，缓慢的更新Key Encoder，构建一个又大又一致的动态词典。



![image-20240408212751319](http://sthda9dn6.hd-bkt.clouddn.com/FlwCsI3Y0anvjZAw_Uhq4q_lirYh)

**框架伪代码**  

pretext task ： 实例判别任务。目标拉近正样本对在特征空间的距离，并使负样本对尽可能的远离。

```python
# f_q, f_k: encoder networks for query and key
# queue: dictionary as a queue of K keys (CxK)
# m: momentum
# t: temperature
f_k.params = f_q.params # initialize
for x in loader: # load a minibatch x with N samples
    x_q = aug(x) # a randomly augmented version
    x_k = aug(x) # another randomly augmented version
    
    q = f_q.forward(x_q) # queries: NxC
    k = f_k.forward(x_k) # keys: NxC
    k = k.detach() # no gradient to keys
    
    # positive logits: Nx1
    l_pos = bmm(q.view(N,1,C), k.view(N,C,1))
    
    # negative logits: NxK
    l_neg = mm(q.view(N,C), queue.view(C,K))
    
    # logits: Nx(1+K)
    logits = cat([l_pos, l_neg], dim=1)
    
    # contrastive loss, Eqn.(1)
    labels = zeros(N) # positives are the 0-th
    loss = CrossEntropyLoss(logits/t, labels)
    
    # SGD update: query network
    loss.backward()
    update(f_q.params)
    
    # momentum update: key network
    f_k.params = m*f_k.params+(1-m)*f_q.params
    
    # update dictionary
    enqueue(queue, k) # enqueue the current minibatch
    dequeue(queue) # dequeue the earliest minibatch
```

两个不同视角的同一张图片为正样本对，不同图片为负样本对。

抽特征：q, k；   最大化q k相似且与负样本远离

更新Q_Encoder, 使用Q_Encoder参数更新K_Encoder，动量缓慢更新



无监督预训练后好，取Q_Encoder作为抽取特征骨干网络，冻结其参数，微调分类头，在进行泛化测试





---

### MAE  - CVPR2022

***Masked Autoencoders Are Scalable Vision Learners***

非对称掩码自动编码器；Encoder和Decoder架构可以不同

(生成式无监督学习)

CV领域的Bert

⭐NLP与CV的不同：

- 信息密度不同

  - NLP：句子信息语义很高，信息密度也高。（人类语言-事先浓缩过的信息）

  - 视觉：信息很冗余，也没高级的语义（自然界），像素可以被相邻的重建恢复

- 恢复难度不一致

  - 恢复高级语义单词和恢复像素级(低级语义)图片，难度也不一样。



***框架：***

![image-20240409131436588](http://sthda9dn6.hd-bkt.clouddn.com/Frag5yZW-mr5iE-Qq9Dg3Mj9TQQ5)

Encoder：位置编码所有Patch都加上。但仅输入未被Mask的Patch。并且Mask比例很高(论文mask75%patch)，迫使模型学到高维的特征，而非捷径。

Decoder：Mask Patch是自学习的向量，并且和Encoded Patch在位置上一致，再次添加位置信息。

简单实现（shuffle，截取前面的作为Encoder输入，后面Patch被Mask；再shuffle逆操作，将Encoded Patch和learnabel patch位置对齐组合好进行Decoder。再重建像素）

通过预测每个屏蔽补丁的像素值来重建输入。解码器的最后一层是线性投影，其输出通道的数量等于补丁中像素值的数量。

**损失函数**计算像素空间中重建图像和原始图像之间的**均方误差**。



问题：

	预训练的输入中具有很大一部分掩码标记，而这在下游任务未损坏的图像中不存在。这种差距可能会降低部署的准确性。





**自监督学习**方法通常侧重于预训练的不同借口任务。

**对比学习**对两个或多个视图之间的图像相似性和相异性进行建模。





---

### wav2vec 2.0  - NeurIPS 2020

***wav2vec 2.0: A Framework for Self-Supervised Learning of Speech Representations***



**时序信号版本的Bert**

[自监督]学习通用特征 => 再微调任务头

(判别式无监督学习)

***总体框架：***

![image-20240411195125065](http://sthda9dn6.hd-bkt.clouddn.com/FqUfVASNdBKkGcmm0F4GiZWIQKjb)

1️⃣ 原始信号[X] =CNN=> 浅在特征表示 [Z]

2️⃣ 浅在特征表示 [Z] =Transformer Encoder=> 全局上下文特征表示 [C]

3️⃣ 浅在特征表示 [Z] =量化器=> 对比目标[Q]

- 把原来连续的特征空间假设是d维，拆分成G个子空间（codebook），每个子空间维度是d/G。
- 然后分别在每个子空间里面聚类（K-mean什么的），一共获得V个中心和其中心特征。
- 每个类别的特征用其中心特征代替。

![image-20240411204733337](http://sthda9dn6.hd-bkt.clouddn.com/FmcVQRp_69PpmPRTef5A9RL5WkXm)

![image-20240411204739181](http://sthda9dn6.hd-bkt.clouddn.com/FjLxT0_K3dukPMzGasHHX8RUkVYI)

量化qt和对应ct

❌ Quantization module 部分不理解



**Mask**

![image-20240411210825518](http://sthda9dn6.hd-bkt.clouddn.com/FlRfun5vPoin9xqspVf4f0lzVOZO)

随机起点，遮挡后面t个时间步

![image-20240411210806211](http://sthda9dn6.hd-bkt.clouddn.com/FnkR4fVzwbvuq6RjaCs_dndb6v5h)



***对比损失***

		![image-20240411210912407](http://sthda9dn6.hd-bkt.clouddn.com/FqYwQ0MDlAXtdrK4q6VHMuWRn3km)

包括 qt 和 K 个干扰项

***❗❗❗理解不太清晰***





---

### **Whisper  - 2022 OpenAI**

***Robust Speech Recognition via Large-Scale Weak Supervision***



大力出奇迹



***模型结构***

![image-20240411211708389](http://sthda9dn6.hd-bkt.clouddn.com/FrJeqrQ_ZkI5Dq1CvF8QkRfMxcaH)



***多任务训练***

- [英文口语=> 英文]                            语音识别 

- [多语言口语 => 英文]                       语音识别 +翻译
- [多语言口语 => 对应语言文字]        语音识别 

- 识别背景音(无内容声音)



⭐⭐⭐**信号 => Log-Mel Spectrogram(频谱图)**

音素



***⭐⭐⭐数据集***

680k小时，超大数据集。 在此基础上预训练，并且0样本迁移(无需特定任务微调)





---

### CPC  - Machine Learning 2018

***Representation Learning with Contrastive Predictive Coding***

*Contrastive Predictive Coding   - 无监督学习*

**通过预测未来，学习特征表示**  (学习对(高维)信号不同部分之间的底层共享信息进行编码的表示  - 局部平滑度)

不预测原始信号，而是对高维嵌入依赖建模

(判别式无监督学习)

***CPC框架***

![image-20240412150754949](http://sthda9dn6.hd-bkt.clouddn.com/Fkg2K2xjA7uBbk6vyr6jA7lWejzG)
$$
g_{enc}:  local\ feature\ learning\\
g_{ar}:  global\ context\ learning
$$

***对比学习***

```python
# 序列： [x1, x2, x3, x4, x5, x6, x7, x8]

# positive sample
# x1, x2, x3, x4 => c <=> x5, x6, x7, x8

# negative sample
# x1, x2, x3, x4 => c <=> xa, xb, xc, xd (同batch中其他的)
```

迫使模型学习序列间的high level feature，学习这种内部的顺序逻辑关系





**互信息公式**  - 衡量两个随机变量之间的相互依赖程度
$$
I(x;c)=\sum_{x,c}p(x,c)\log\frac{p(x|c)}{p(x)}. \\
I(x;c): x 与 c 的互信息\\
p(x,c): x 和 c 同时发生的联合概率分布\\
p(x|c): 给定 c 的条件下，x 发生的条件概率分布\\
p(x): x 的边缘概率分布
$$
**InfoNCE Loss**

**最小化CPC定义的损失 => 实际上最大化 context c_t 和待预测正样本 X_t 之间的互信息**

优化目标：最大化似然概率



*正相关*

![image-20240415213652095](http://sthda9dn6.hd-bkt.clouddn.com/Fld0d7f2o5eYFLT27mG-kwpf69p9)

*近似计算互信息*

![image-20240415213622018](http://sthda9dn6.hd-bkt.clouddn.com/FiE2tChXaZTae_EuLA5UxEl-BwbM)

*Loss*

![image-20240415213722239](http://sthda9dn6.hd-bkt.clouddn.com/Frw_sTw_6jzv57J91HJTXwqKjVN-)

X = {x1, x2, ..., xN} N个负样本，从batch中其他数据中采样

E/X  表示似然概率 



最大化互信息 => 最小化Loss  (互信息下界)

![image-20240415213954622](http://sthda9dn6.hd-bkt.clouddn.com/Fmr-VnocMhPJJtXfiZGyk_JrBuo1)



**完整InfoNCE-Loss**
$$
\text{InfoNCELoss} = -\frac{1}{N} \sum_{i=1}^{N} \log \frac{\text{exp}(s(x_i, x_i^+))}{\text{exp}(s(x_i, x_i^+)) + \sum_{j=1}^{K} \text{exp}(s(x_i, x_j^-))}
$$



*不太了解这个最大化互信息的Loss*





---

### **DALL·E 2**  - 2022.3 OpenAI

*Hierarchical Text-Conditional Image Generation with CLIP Latents*

***层级式图生文***



**模型架构**

![image-20240416192447568](http://sthda9dn6.hd-bkt.clouddn.com/Fl5NOQ7na3jY5w-4oMbQClRzwjgn)

***描述：***

- 虚线上是CLIP架构(文本和图像的联合表示空间)，学习图文对的关联信息
- 虚线下是生成框架，prior模型根据Text Embedding生成出CLIP对应的Image Embedding， decoder(diffussion model)根据Image Embedding进行重建



*⭐分步训练*

***prior*** 

生成CLIP image Embedding (diffusion model)

![image-20240416194435205](http://sthda9dn6.hd-bkt.clouddn.com/FlbLpzwV8mvw-N7XcRUxXpKN_tVX)



***decoder***

是根据image Embedding生成图片 -- 并且这个decoder是多个堆叠，先生成低分辨率，再高清化  -  (diffusion model)





***简洁表示：***

![image-20240416193905706](http://sthda9dn6.hd-bkt.clouddn.com/FkLqbQZZdzkbNQsmfckYUFfbO7OI)

*y - 文字*

*x - 图片*

*zi - 图片嵌入  （显式生成图像表示可以提高图像多样性，同时将照片真实性和标题相似度的损失降至最低）*





---

### diffusion model  - NeurIPS 2020

***Denoising Diffusion Probabilistic Models***



**框架：**

***加噪 + 去噪(还原)***

![image-20240417165835467](http://sthda9dn6.hd-bkt.clouddn.com/Fq1ieN5eRAPoPKJz9gRb3UUVXuEa)



***正向过程：***

1️⃣ Xt 是 前一张图片加噪生成的，Z1是服从正太分布的噪声

![image-20240417170233229](http://sthda9dn6.hd-bkt.clouddn.com/Fu4Ef01QR3UNTEVo22lrQcOAktpQ)

![image-20240417170238304](http://sthda9dn6.hd-bkt.clouddn.com/Fi78LTSVdR9ki7Tkg_P3-MXWSP-U)

βt是超参数=范围为[0.0001,0.02]递增，则αt 是随时间减少， 表示公式一中原图信息越来越少，噪声越来越重

2️⃣ 递推带入一下

![image-20240417170436248](http://sthda9dn6.hd-bkt.clouddn.com/FiHTiSdVmNrgDsNfKd8o8ajxz5g3)

![image-20240417170509368](http://sthda9dn6.hd-bkt.clouddn.com/FjGR-X-OPfIzkMdjwnKR3-BwDzMq)

最后可得···

![image-20240417170704121](http://sthda9dn6.hd-bkt.clouddn.com/FhFVZkEeHGk4XHiNcobYmz_Kyx1p)

Zt_hat 是一个服从正太分布的随机噪声，at_hat = at\*at-1\*···*a1, **可由X0直接产生任意时间步的加噪图片**



**反向去噪过程：**

*核心基础*

![image-20240417171006179](http://sthda9dn6.hd-bkt.clouddn.com/FuLyZjUK_C7OzaR_Sx5izsv8qVfm)



1️⃣ 用Xt生成Xt-1 ，按贝叶斯公式转换

![image-20240417171023360](http://sthda9dn6.hd-bkt.clouddn.com/FlPyQq-wecfnCuAKSYOq9yDXtvWy)



2️⃣ q(Xt|Xt-1) == q(Xt|Xt-1, X0)， 而q(Xt-1) == q(Xt-1|X0) **任意步加噪图可由原图直接产生**

![image-20240417171223592](http://sthda9dn6.hd-bkt.clouddn.com/FrLTV45PiHS6n-plx9FCssOtXc8E)

3️⃣ 反解公式7， X0可由Xt进行估计

![image-20240417171845271](http://sthda9dn6.hd-bkt.clouddn.com/FhkEASm1Lj9tDI7fDu3WnhzbDeTS)

4️⃣ 带入并整理

![image-20240417172918885](http://sthda9dn6.hd-bkt.clouddn.com/Fp8r3kgTbQNu6B5nUdKDOQt0Lzfj)



··· 因此可根据Xt => Xt-1， 下式为最终的**去噪公式**
$$
x = \frac{1}{\sqrt{\alpha}} \left( x - \frac{1 - \alpha}{\sqrt{1 - \alpha_{\text{hat}}}} \cdot \text{predicted\_noise} \right) + \sqrt{\beta} \cdot \text{noise}
$$
β noise 保证多样性



*Ɛθ表示预测模型*

![image-20240417175432578](http://sthda9dn6.hd-bkt.clouddn.com/FiJbpIpLJ6S2emRh_Rh6kg19Dhyp)

***代码逻辑***

训练：

```python
for i, images in enumerate(pbar):
    images = images.to(device)
    t = diffusion.sample_timesteps(images.shape[0]).to(device)	# 采样几个时间步进行训练
    x_t, noise = diffusion.noise_images(images, t)	# @ 公式-7
    predicted_noise = model(x_t, t)	# Unet
    loss = mse(noise, predicted_noise)

    optimizer.zero_grad()
    loss.backward()
    optimizer.step()

sampled_images = diffusion.sample(model, n=images.shape[0]) # @ 去噪公式
```



*预测模型*

Unet 带时间点嵌入(用余弦-位置嵌入实现的)

```python
def forward(self, x, t):
    t = t.unsqueeze(-1).type(torch.float)
    t = self.pos_encoding(t, self.time_dim)		# 嵌入时间步顺序

    x1 = self.inc(x)	    # cnn
    x2 = self.down1(x1, t)  # pooling
    x2 = self.sa1(x2)	    # self attention
    x3 = self.down2(x2, t)
    x3 = self.sa2(x3)
    x4 = self.down3(x3, t)
    x4 = self.sa3(x4)

    x4 = self.bot1(x4)	
    x4 = self.bot2(x4)
    x4 = self.bot3(x4)

    x = self.up1(x4, x3, t)	# 插值上采样，再拼接skip connect
    x = self.sa4(x)
    x = self.up2(x, x2, t)
    x = self.sa5(x)
    x = self.up3(x, x1, t)
    x = self.sa6(x)
    output = self.outc(x)
    return output
```

p(Xt-2|Xt-1, t, y)  t是时间步嵌入，y是条件嵌入

最简单的融合方法就是相加



？？？ 疑惑点 - 反向过程求的t时刻的均值方差用在哪了？



2025/1/14 更新：

// X => X_noise_t  可以一步生成，可以时间步t可以直接生成对应t时间步的噪声图。

// 使用U-net预测时间步t的噪声 使用MSE-Loss进行训练

```python
# 训练循环
for epoch in range(num_epochs):
    for x_0 in dataloader:  # x_0 是原始数据
        # 1. 随机选择一个时间步 t
        t = torch.randint(0, T, (x_0.size(0),))  # 为每个样本随机选择一个时间步

        # 2. 前向过程：添加噪声
        epsilon = torch.randn_like(x_0)  # 采样噪声
        alpha_bar_t_t = alpha_bar_t[t].view(-1, 1, 1, 1)  # 选择对应时间步的 alpha_bar_t
        x_t = torch.sqrt(alpha_bar_t_t) * x_0 + torch.sqrt(1 - alpha_bar_t_t) * epsilon  # 添加噪声

        # 3. 反向过程：预测噪声
        predicted_epsilon = model(x_t, t)  # 模型预测噪声

        # 4. 计算损失函数
        loss = F.mse_loss(predicted_epsilon, epsilon)  # 均方误差损失

        # 5. 反向传播和优化
        optimizer.zero_grad()
        loss.backward()
        optimizer.step()
```

// 串行，一步一步去噪

```python
# 生成过程
def generate_samples(model, num_samples, T, alpha_bar_t):
    x_T = torch.randn(num_samples, ...)  # 从标准正态分布中采样初始噪声
    for t in range(T-1, 0, -1):  # 从 T-1 到 1
        # 预测噪声
        epsilon = model(x_T, t)
        
        # 计算去噪后的数据   <= 消除噪声
        alpha_t = alpha_bar_t[t] / alpha_bar_t[t-1]
        x_T = (x_T - torch.sqrt(1 - alpha_t) * epsilon) / torch.sqrt(alpha_t)
    
    return x_T
```



---

### Time-Frequency Consistency  - NeurIPS 2022

***Self-Supervised Contrastive Pre-Training for Time Series via Time-Frequency Consistency***



期望同一示例的基于**时间**和基于**频率**的表示在**时频空间中靠近**在一起

(判别式无监督学习)

pretext task：实例判别 (一对正样本，其余负样本)

***框架***

![image-20240423132039482](http://sthda9dn6.hd-bkt.clouddn.com/FiQwWmZiTaA80GpxZ_cP6LoWQBnD)

时域增强：基于时间特性从 xi 扩充，包括抖动、缩放、时移和邻域分段；

频域增强：频谱特征扰动 xFi 的增强，添加或删除频率分量来扰动频谱 (确保扰动时间序列仍然与原始样本在频域和时域仍相似)



Loss：

![image-20240423133548048](http://sthda9dn6.hd-bkt.clouddn.com/Fv5GblVwR6sTQrzRmi3mkPCEiVeA)

余弦相似度：衡量两个向量之间相似性，范围[-1, 1]



**// 补充 2025/1/14**

<img src="http://sthda9dn6.hd-bkt.clouddn.com/Fhfw9iENvupdYltnGP1TnIoanttM" alt="image-20250114163629763" style="zoom:40%;" />

样本越相似  => sim(i, j) 越大 => exp(sim) 越大 =>  exp(sim)/\sum(exp) 越接近于1 => log(·)越接近于0 

-log(·) 将图像倒置，样本越**不**相似  => exp(sim)/\sum(exp) 越接近于0 => loss 越大



---

### COMET  - NeurIPS  2023

***Contrast Everything A Hierarchical Contrastive Framework for Medical Time-Series***

(判别式无监督学习)

- 我们的方法旨在弥合标记数据的有限可用性与医疗时间序列分析中稳健且可概括的模型的需求之间的差距
-  对比表示学习背后的关键思想是通过将相似的数据靠近在一起并将不相似的数据进一步分开来挖掘数据一致性

 关键是要利用所有可用的信息；除了样本标签之外，数据集是否还有其他信息？(补充信息)



**患者间、患者内进行测试(定义打乱规则)**



**⭐⭐⭐多级信息**

![image-20240424172452759](http://sthda9dn6.hd-bkt.clouddn.com/FszIdIHqJYTLaVVMaDJ3SiLhuUQ4)



**多级对比学习框架**

![image-20240424172549571](http://sthda9dn6.hd-bkt.clouddn.com/FgCOlLyo72rghf4TuKsLoY_a4KtI)



***代码分析总结***

```python
X_train.size == [Batch, channels, segment]  # segment length 330, sampling_rate = 250Hz
y_train  # 心肌梗塞二分类标签， patient_id， segment_id
```

*细分为样本级别(心跳)，故图示Encoder不同级别对比学习可复用*



**关键，计算不同级别的对比Loss**

***Patient-Level Loss***

![image-20240424205635784](http://sthda9dn6.hd-bkt.clouddn.com/FmojCp5VGez0QwxsX3dFyUHUCK__)

```python
x = [Batch, channels, segment]

out1 = net(x)
out2 = net(x)  # ? 模拟数据增强后的不同视角或版本，以便在对比学习中生成有效的正例和负例

# 根据y_train 病人id，构建mask矩阵
pid1, pid2 = np.meshgrid(str_pid, str_pid)	
pid_matrix = pid1 + '-' + pid2		#  每项为 id_x - id_y	
# 目标位置 id_x - id_x
pids_of_interest = np.unique(str_pid + '-' + str_pid)  # unique combinations of pids of interest i.e. matching 
bool_matrix_of_interest = np.zeros((len(str_pid), len(str_pid)))
for pid in pids_of_interest:
    bool_matrix_of_interest += pid_matrix == pid

# 上三角和下三角目标row，col
rows1, cols1 = np.where(np.triu(bool_matrix_of_interest, 1))  # upper triangle same patient combs
rows2, cols2 = np.where(np.tril(bool_matrix_of_interest, -1))  # down triangle same patient combs
    
out1, out2  => sim_matrix_exp  # 余弦相似度矩阵

# @@ 对比学习：不考虑对角线元素的原因主要是因为对角线元素表示的是特征向量与其自身的相似度
# @@ 我们关注的是如何使具有相同标签的不同特征向量之间更相近，而使不同标签的特征向量之间更疏远

# Loss 分母， 某样本与其他样本相似度之和，  @分上三角和下三角
triu_sum = torch.sum(sim_matrix_exp, 1)  # add column
tril_sum = torch.sum(sim_matrix_exp, 0)  # add row

loss_terms = 0  # 取平均 
if len(rows1) > 0:
    triu_elements = sim_matrix_exp[rows1, cols1]  # row and column for upper triangle same patient combinations
    loss_triu = -torch.mean(torch.log(triu_elements / triu_sum[rows1]))
    loss += loss_triu  # technically need to add 1 more term for symmetry
    loss_terms += 1
...

loss = loss/loss_terms
```

***Trial-Level Loss***

同上，只不过patient_id => segment_id



❌❌❌*Sample-Level Loss & Observation-Level Loss*

看不懂源码~ 😎😭





---

### TS2Vec  - AAAI 22

***TS2Vec: Towards Universal Representation of Time Series***

(判别式无监督学习)

现存工作局限：

- 它们都没有以不同尺度的时间序列为特征来捕获尺度不变的信息，而这对于时间序列任务的成功至关重要。多尺度特征可以提供不同级别的语义并提高学习表示的泛化能力
- 粗粒度表示 - 整个时间序列，可能没那么细致

之前工作的问题

![image-20240425195237484](http://sthda9dn6.hd-bkt.clouddn.com/Fl7FFxGPcEP-7YFmsNf9PYHpZVDj)

正样本对会误判

- 当存在水平偏移时，子系列一致性很容易受到攻击 (左图)
- 当出现异常时，时间一致性可能会引入误报对 (右图)

	

创新：

- TS2Vec 中的对比目标基于**增强上下文视图**，即相同子系列在两个增强上下文中的表示应该是一致的 **(上下文语境下)** 
- 层级式对比Loss，由细粒度=>粗粒度，局部到全局  # 学习各种语义级别的任意子系列的上下文表示，灵活且通用的表示。 顶级语义级别的对比使模型能够学习实例(样本)级表示



***框架***

![image-20240425171329665](http://sthda9dn6.hd-bkt.clouddn.com/FmF0LTUTFGsYXsZSjO1Es3CsVaFK)

流程：

- 实例(一段信号)，分两个重叠的子序列,  [batch, channel, dim_feature]
- 投影，[batch, channel, dim_feature] => [batch, channel, dim_hidden]
- 随机Mask掉部分信号
- CNN-Encoder

⭐⭐⭐  Hierarchical Contrasting Loss

- ***instance & temporal contrastive loss***  

		![image-20240425172445164](http://sthda9dn6.hd-bkt.clouddn.com/FjeDH748BsrYQtTOEOmSjYW_SEMr)
		
		*用 -F.log_softmax(x, dim=-1) 实现*

- z1 = F.max_pool1d(z1.transpose(1, 2), kernel_size=2).transpose(1, 2)
  z2 = F.max_pool1d(z2.transpose(1, 2), kernel_size=2).transpose(1, 2)

  编码特征浓缩，**多尺度**的Contrast loss



***图示Loss过程***

![image-20240425193417634](http://sthda9dn6.hd-bkt.clouddn.com/Fu8OeKZvY8TQ-hCVmTs7laVuLNnM)

子序列是从原始信号中裁剪下来的，并且**有重叠部分**

![image-20240425175053049](http://sthda9dn6.hd-bkt.clouddn.com/Fk0G8jM8ufq4FezR7WIn1MKEPHow)





---

### TimesURL  - AAAI 2024

***TimesURL: Self-Supervised Contrastive Learning for Universal Time Series Representation Learning***

代码在TS2Vec上修改

=> 在这里，我们必须提到，重要的时间变化信息，例如趋势和季节，在多次最大池化操作后会丢失，因此顶层对比实际上无法为下游任务捕获足够的实例级信息

=> 掩码重建进行学习实例级信息

(判别式+生成式 混合 无监督学习)

*观察*

![image-20240427213825901](http://sthda9dn6.hd-bkt.clouddn.com/FlxcLu-nrG0hZ5m6BeQMnfJk4BjQ)



**简单负样本：**大多数时间序列片段可以被视为容易负样本。 这些片段往往表现出与锚点的语义差异，并且仅贡献较小的梯度，因此无法提供有用的区分信息

**硬负样本：** 硬负样本就是离正样本很近，并且模型很难区别的

正样本-**硬负样本**-负样本：，这些样本如果让其远离正样本可以大大提高模型性能。 其有效性被大量的简单负样本所掩盖。（现存框架没有特点显示的指出）

由于时间序列中的局部平滑性和马尔可夫特性，大多数负样本很容易不足以捕获时间信息，因为它们从根本上缺乏驱动对比学习所需的学习信号。 作为图 2 中真实示例，对于每个正锚点（红色方块），相应的负样本（灰色标记）包含许多简单的负样本和很少的困难负样本，即许多负片太远，无法造成对比损失。



对比学习的一个关键组成部分是选择适当的增强，这些增强可以施加一些先验来构建可行的正样本，以便编码器可以被训练来学习鲁棒和有区别的表示

**频率混合**用于通过将通过快速傅里叶变换（FFT）运算计算出的一个训练实例 xi 中的一定比例的频率分量替换为同一批次中的另一个随机训练实例 xk 的相同频率分量来生成新的上下文视图 （保持病理相同）

**随机裁剪**。 ***重叠两个子序列*** - 随机裁剪是**上下文一致性策略**的关键步骤。 它可以保持时间序列的重要时间关系和语义一致性



***创新点：***

- 提出双Universum概念，就是利用Mix-up增强F(x)，追加硬负样本。
- 对比学习+自监督掩码重建，联合优化，来捕获段级和实例级信息， 实现通用表示



⭐ 第一类包括预测、异常检测和插补，它们更多地依赖于在分段级别捕获的细粒度信息，因为这些任务需要推断特定的时间戳或子序列。细粒度(局部)

⭐ 第二类包括分类和聚类优先考虑实例级信息（即粗粒度信息），旨在推断整个系列的目标。粗粒度(全局)

<em style="color:red; font-weight:bolder">实现 : 分段(对比学习，学习片段)，整句(掩码重建，学习整体)</em>



***框架：***

![image-20240427213747206](http://sthda9dn6.hd-bkt.clouddn.com/FiK4E0Rba4I2lpP2fIpxQ8rFyuFb)



***AUG：***

	频率增强，**x => fft() =mask+fusion=> ifft() => y**，①该篇论文fusion是融合同batch其他信号的某些部分 => 领域创新 （fusion的信号应该和这个信号相同病理，扩张数据分布）



***DualConv：***

	原始信号的两个增强子视图，z1 = Encoder(x1),  	z1' = Encoder(x1'),  => mix-up option => **z1_mix = α × z1 + (1-α) × z1[torch.randperm(z1.shape[0])]**，z1'_mix。  
	
	[z1, z1', z1_mix, z1'mix] @ [z1, z1', z1_mix, z1'mix].T   => sim => **-F.log_softmax(sim)**
	
	loss:  俩视图对应段为正样本(分子)

![image-20240427220130014](http://sthda9dn6.hd-bkt.clouddn.com/FqUjTPyOQ_WkWJxxkJ0GpzCsSAAB)

负样本在母分，x_mix做负样本增强。







---

### Mixup  - 2017 Machine Learning

***mixup: BEYOND EMPIRICAL RISK MINIMIZATION***



一种简单并且不需要专业领域知识的数据增强



现存问题讨论：

- 过拟合(大模型，直接记忆Train data，走捷径)                                          - overfitting
- 精心设计样本(对抗性例子，人难以察觉，但模型会给出错误的答案)      - generalize

👇

**ERM**(经验风险最小化原则)问题

- 一方面，即使存在强正则化，ERM 也允许大型神经网络记忆（而不是归纳）训练数据
- 另一方面，使用 ERM 训练的神经网络在对训练分布之外的示例进行评估时会极大地改变其预测。



CV: 图像的邻近区域定义为其水平反射、轻微旋转和轻微缩放的集合

=>  数据增强始终可以提高泛化能力, 但该过程依赖于数据集，因此需要使用专家知识 （不同领域增强不一定通用）

数据增强假设邻近的示例共享同一类，并且不会对不同类的示例之间的邻近关系进行建模。(聚类)



=> 邻近风险最小化 (**VRM**) 原则

⭐=> 从训练样例的邻近分布中提取额外的虚拟样例，以扩大训练分布的支持度



***方法：***

![image-20240428175535852](http://sthda9dn6.hd-bkt.clouddn.com/FqaLaOdoIr-cpS9l6zmtNSzR-Vqq)

***框架伪代码*：**

![image-20240428175500700](http://sthda9dn6.hd-bkt.clouddn.com/FpPy-mOfvDy30xYu8K1sC1Q5zOAp)

*图例：*

![image-20240428175951150](http://sthda9dn6.hd-bkt.clouddn.com/Fg1UhIVdFoUyTZUNKCNOy5NVzP-o)

虚拟数据，让数据边界过渡； 当不在Train数据的分布出现时，降低不确定性。 稍微清晰化边界





---

### YOLO-v1  - CVPR 2016 

***You Only Look Once: Unified, Real-Time Object Detection***

 ⭐将目标检测视作回归问题  => 预测出来



***前向推理***

*Model:*

![image-20240429203859825](http://sthda9dn6.hd-bkt.clouddn.com/FhxSE95ivo_u62hu7k5pAhAamgQZ)

*input: [3, 448, 448]  => output: [30, 7, 7]*

![image-20240429204129869](http://sthda9dn6.hd-bkt.clouddn.com/FlwDkypmqQ3dpDIVV3Dvc4rqnrcd)

*置信度(confidence): 这个值代表了模型认为预测的边界框内存在对象的概率*

*框的中心坐标 + 宽高*

*框中的物体是什么类*



![image-20240429210124912](http://sthda9dn6.hd-bkt.clouddn.com/FtrEL1xVvt9z-KAIvxuRVYLmLS_J)

***非极大值抑制***  （最佳：每个类别独立执行非极大值抑制，从而更精确地处理多类别情况）

1. **置信度排序**：首先将所有的预测边界框按照它们的置信度（confidence scores）进行降序排序。
2. **选择最高置信度边界框**：从排序后的列表中选择置信度最高的边界框作为参考框（reference box）。
3. **计算IOU**：计算选中的参考框与列表中其他所有边界框的交并比（IOU）。交并比是两个边界框的交集面积与它们的并集面积的比值。
4. **抑制**：如果参考框与任何其他边界框的IOU超过预先设定的阈值（通常设置为0.5），那么这些边界框会被认为是多余的，并从列表中删除。
5. **重复步骤**：从剩余的边界框列表中再次选择置信度最高的边界框，重复上述过程，直到所有的边界框都被处理完毕。
6. **最终结果**：经过非极大值抑制后，剩余的边界框被认为是对目标位置的最佳预测，它们将被用于最终的目标检测输出。



***训练：***

![image-20240429210707202](http://sthda9dn6.hd-bkt.clouddn.com/FoYarO_WymLCMWJzy9ShWN6l4FAd)

λcoord = 5,   λnoobj = 0.5,  调整各个部分的重要性
$$
1_{ij}^{obj}: 表示第ij个格子有对象 \\
1_{ij}^{noobj}: 表示第ij个格子没有对象\\
S^{2}: 图片划分格子 \\
B: 每个格子预测多少个框
$$
*bounding box loss : 中心点 + 框宽高*

*confidence: 格子是否有对象*

*classes：格子分类是否正确*





---

### Semi-Supervised Hybrid Loss  -  Machine Learning 2023

***Semi-Supervised End-To-End Contrastive Learning For Time Series Classification***

1️⃣无标签数据对比学习(增强视图一致性)，2️⃣有标签对比学习(相同种类一致性)，3️⃣有标签分类监督学习

(判别式无监督学习 + 有监督学习)

**框架对比**

![image-20240504141408686](http://sthda9dn6.hd-bkt.clouddn.com/FvRDJcCTiZLB2TSscRE047d6YZYv)

⭐ ***End to End***



**框架**

![image-20240504141522823](http://sthda9dn6.hd-bkt.clouddn.com/FntDu8y-SPWRFtWIEWiahhJImAax)

Unlabeled Sample : 使用两个增强视图作为positive pair，与其他sample为negative pair (标准的对比学习)

Labeled Sample：1️⃣ 同类型的sample为positive pair，不同类型的sample为negative pair.  2️⃣过分类头，计算分类Loss

❤️ 混合上述三个Loss，联合优化**Encoder**





---

### SimCLR  - 2020

***A Simple Framework for Contrastive Learning of Visual Representations***



*贡献：*

- **数据增强**对于对比学习至关重要 (裁剪缩放，翻转，颜色紊乱，旋转， 掩盖， 高斯噪声， 高斯模糊，Sobel 滤波)  - 裁剪缩放+颜色紊乱 比较好

- 在经过Resnet编码器后，追加MLP能增强模型性能

- 样本x，增强视图xi和xj(正样本)，batch size =N，一共2N的增强视图，对于某个样本x，xi和xj为正样本，和batch中剩余的样本的增强为负样本  

  (大batchsize, 性能更好， 全局BN)

***样本自成一类***，来尽可能地让编码器找到图像中最重要的特征



*框架*

![image-20240504192858744](http://sthda9dn6.hd-bkt.clouddn.com/FtXMJSSYglENTuecQD7bhMyV1xVo)

共享参数 shared weight

*Loss*

![image-20240504192936596](http://sthda9dn6.hd-bkt.clouddn.com/FnYcuZnuiDcvvR3ajLOAq4DShL2R)

上面是正样本对，下面是负样本对
-log_softmax() => 挑选出需要的值



*算法*

![image-20240504193222346](http://sthda9dn6.hd-bkt.clouddn.com/FtXUBR7a_alxoHOMOf_A6CuFi92w)





---

### ViLT  - 2021

***ViLT Vision-and-Language Transformer Without Convolution or Region Supervision***

极简结构的图片文多模态融合



速度限制-问题分析

![image-20240506202622982](http://sthda9dn6.hd-bkt.clouddn.com/FophrEolg_pY22h3f06TApCOn7vU)

归纳总结：

![image-20240506201201840](http://sthda9dn6.hd-bkt.clouddn.com/Ft9OjzVqwvKABcvbEdV0-NdHWT3y)

(a) vision embedding 参数量 > Text Embedding > Modality Interaction  # 缺点，视觉嵌入太重(比重太大)，并且融合非常简单即点乘算相似度

(b) vision embedding和Text embedding 占比差不多  > Modality Interaction   # 模态融合之前，工作太繁杂，而且前抽取特征不好，限制后面融合，并且不重视后面的模态融合操作。

(c) 重视visual  Embed和后期的modality interaction，# text 和 vision不均等，重要性不平衡

=> 简单框架，Text词嵌入(bert中的BertEmbeddings加载训练后的权重)，vision用patch projection，都很快

⭐ 1. 图像和文本前期嵌入应该有相似均匀的表达能力 2. 这两种模态是否在深层网络中相互作用。



模型框架：

![image-20240506202930648](http://sthda9dn6.hd-bkt.clouddn.com/Fk4qZVe2Ed-ezQCm2RoOZuE3J0Ha)

*初始化参数-ViT，而不是bert*



优化目标(主要)：

- Image Text Matching：0.5概率将图片替换为与文本不匹配的图片，预测一致性(二分类问题)
- Masked Language Modeling：预测被遮掩的词



text cls: 预测图文是否一致，二分类

text token set： 全局上下文=>预被掩词

text token set 和 visual token set：进行对齐Loss



---

### BEIT-v3 2022

***Image as a Foreign Language: BEIT Pretraining for All Vision and Vision-Language Tasks***



统一Vision 和 NLP

⭐ 核心思想是图像可以被建模为一门外语，这样我们就可以对图像、文本和图文对进行统一的掩码“语言”建模。



**结果非常好**

![image-20240506221847766](http://sthda9dn6.hd-bkt.clouddn.com/FvdT24-GqGPwRnm3KHtiyZK8rTtO)



*基础块*

![image-20240506221730469](http://sthda9dn6.hd-bkt.clouddn.com/FlsZ16a9K6-0aI8vE7dcRGRmqc4V)

共享注意力矩阵(都是一个物体的不同视角)，但是最后的FFN各个模态专享



*拓展到不同的模态*：

![image-20240506221937618](http://sthda9dn6.hd-bkt.clouddn.com/FvuKOj42D1UKUyfgc9DhAB_kHSUm)



*任务*：

**图像字幕任务**：采用了特殊的自注意力掩模。 ***图像标记（即图像块）只能在图像序列内双向相互关注***。 ***标题的标记可以关注图像标记、它们的左侧标题标记以及它们本身***。 在微调过程中，我们随机屏蔽一定比例的标题标记。 该模型经过训练，可以根据图像的线索及其左侧标题上下文来恢复这些标记。 我们还屏蔽了特殊的边界标记 [SEP]，以帮助模型学习终止生成



**视觉问答：** 将任务表述为分类问题。 该模型经过训练，可以从训练集中 3129 个最常见的候选答案中预测答案。我们将给定问题和图像的嵌入连接起来，然后将输入嵌入输入多路转换器以联合编码图像-问题对。 最终的池化输出被输入到分类器层来预测答案。 



**图像文本检索任务：**是测量图像和文本之间的相似度。 根据检索目标的模态，有两个方向：图像到文本检索和文本到图像检索。 *双编码器模型*分别对图像和文本进行编码以获得它们的表示。 然后我们计算这些表示的余弦相似度分数。



**图像分类：** 将该任务制定为图像到文本检索任务。 我们使用类别名称作为文本来构建图像-文本对。 BEIT-3 被训练为双编码器，以找到图像最相关的标签。 在推理过程中，我们首先计算可能的类名的特征嵌入和图像的特征嵌入。 然后计算它们的余弦相似度分数以预测每个图像最可能的标签。





---

### ALBEF  - 2021

***Align before Fuse**: Vision and Language Representation Learning with Momentum Distillation*



现存问题：大多数现有方法采用基于变压器的多模态编码器来联合建模-视觉Token（基于区域的图像特征）和文本Token。 **由于视觉标记和单词标记未对齐**，因此多模态编码器学习图像文本交互具有挑战性。

（1）图像特征和文本符号映射仍然停留在他们自己的空间，使得多模态编码器很难学习建模他们之间的交互；

（2）物体检测器 --- 标注费钱，使用费算力 --- 在预训练阶段需要标注矩形框，在推理阶段高分辨率图像，如 600*1000，速度较慢；

（3）广泛使用的 image-text 数据集均是从网上搜集的带有严重噪声的数据，现有的预训练目标，如 MLM 可能过拟合到文本数据，降低了模型的泛化性能。



*框架：*

![image-20240507195518743](http://sthda9dn6.hd-bkt.clouddn.com/Fm8rrHfMoDMFD44WkrXWbLfBZWsO)

image encoder: ViT(ImageNet预训练参数)  - CLS token

text encoder: Bert(预训练参数)  - CLS token

multimodal encoder: Bert(预训练参数) + cross-attention



⭐ **在传入multi-modal encoder前，使用ITC迫使模型进行对齐 ** *align before fuse的align*

- 对齐图像特征和文本特征，使多模态融合编码器更容易执行跨模态学习
- 改进了单模态编码器，以更好地理解图像和文本的语义
- 它学习一个共同的低维空间来嵌入图像和文本，这使得图像文本匹配目标能够通过我们的对比硬负挖掘找到更多信息样本。



**Image-Text Contrastive Loss：**

正样本对：配对的Image-Text

负样本对：Queue存储着的样本表示

Image => 匹配Text-Queue(Momentum)

Text => 匹配Image-Queue(Momentum)

(代码中利用了Momentum Distillation ...)



**Image-Text Matching:**

有了multi-modal encoder输出的 embed token (正确样本的表征) => 即喂入Multi-modal encoder的 text embed = (positive), Image embed = (positive), 拿**融合的CLS**作为最终表征，过MLP => 二分类预测是否匹配；这部分Targets=(1, 1, ..., 1)

从同batch中，按相似性大小随机挑选一个(hard)负样本，

然后，text embed = (positive, ..., negetive, ...) , Image embed = (negetive, ..., positive)

multi-modal encoder => CLS => MLP => two probability

Targets = (0, 0, ..., 0)

// 重新梳理如下：

```python
# 图像i-文本j => 多模态编码器 => 是否匹配；
图像 1-文本 1 : 匹配对
图像 2-文本 2 : 匹配对
图像 1-文本 2 : 不匹配  // 这里使用余弦相似度选取最困难的样本
图像 2-文本 1 : 不匹配  // 这里使用余弦相似度选取最困难的样本
```



**Masked Language Modeling**:

屏蔽掉一些词，通过从图片模态信息中预测掉被屏蔽的词(多分类Loss)

这里也借助了图像的信息去更好的恢复被mask掉的单词

【这里只对匹配的对计算 掩码Loss】



![image-20240507203423762](http://sthda9dn6.hd-bkt.clouddn.com/Fo5QVCCNEHnTtRVHiD9VTTtG2NhB)

目的：缓解noisy web data的不足，真正的label不一定有momentum的好

![image-20250115224719284](http://sthda9dn6.hd-bkt.clouddn.com/FlfwuaqgM0F6HxRI8NLy9yX0qlBH)

真实label不一定比momentum model给出的predict label好，=> 使用KL散度进行约束 一致性



**最大化互信息视角解释：**

在自监督学习中，a 和 b 是同一图像的两个增强。 在视觉语言表示学习中，我们将 a 和 b 视为捕获其语义的图像文本对的不同变体。 我们的目标是学习对观点变化不变的表征。



最小化Loss => 最大化互信息的下限(最大化了图像-文本对的**不同“视图”**之间的互信息（MI）的下限)

InfoNCE Loss

![image-20240507201135974](http://sthda9dn6.hd-bkt.clouddn.com/Fn6k4nYEeCC98gdG9m20Ny9q11Kt)

***Image-Text Contrastive Loss***

![image-20240507201202181](http://sthda9dn6.hd-bkt.clouddn.com/Foi0D0DMVUtxzsHTL76RXZKMh3jo)

最大化Text和Image中的互信息， ITC 将两个单独的模态（即 I 和 T）视为图像-文本对的两个视图

***MLM:***

![image-20240507201419353](http://sthda9dn6.hd-bkt.clouddn.com/FhfVA9APbraPAhcN-2iDpbkpMlSA)

MLM 将图像-文本对的两个视图视为：(1) 随机选择的单词标记，以及 (2) 图像 + 带有该单词屏蔽的上下文文本。





---

### Instance discrimination  - 2018

***Unsupervised Feature Learning via Non-Parametric Instance Discrimination***

首次提出个体判别任务！

观察：

![image-20240508125442186](http://sthda9dn6.hd-bkt.clouddn.com/FvT1Zpo3nQTgy2ZEOa9t67cXuVDQ)

在监督学习中。在预测'花豹'时，预测概率除了'花豹'，**剩余**预测得分比较高的是'美洲虎'、'猎豹'； **最不相似**的是'救生艇'、'购物车'、'书柜';

***最高响应的类都是视觉相关的***

⭐ 并不是语义标签，**而是数据本身的明显相似性使某些类比其他类更接近**；

❗❗❗ **个体判别：将类监督发挥到了极致，并学习了区分各个实例的特征表示。**



这些观察结果表明，典型的判别学习方法可以自动**发现语义类别之间的明显相似性，而无需明确指导这样做**。

我们能否通过纯粹的判别学习来学习反映实例之间明显相似性的有意义的度量？ **图像本身就是独特的，并且每个图像都可能与同一语义类别中的其他图像显着不同**。如果我们学会**在没有任何语义类别概念的情况下区分各个实例，我们最终可能会得到一个捕获实例之间明显相似性的表示，就像类明智的监督学习如何仍然保留类之间的明显相似性一样**。



**目标**:

	在没有监督的情况下学习嵌入函数 v = fθ(x)。 fθ 是一个具有参数 θ 的深度神经网络，将图像 x 映射到特征 v。这种嵌入将在图像空间上产生一个度量，对于实例 x 和 y, dθ(x, y) = |fθ(x) − fθ(y)|。 良好的嵌入应该将视觉上相似的图像映射得彼此更接近。 我们新颖的无监督特征学习方法是实例级区分。 我们将每个图像实例视为其自己的不同类，并训练分类器来区分各个实例类。



**方法：**

![image-20240508130437824](http://sthda9dn6.hd-bkt.clouddn.com/FgTn1HJ9-x4YmL1t8ikulVrn2ZpY)

用一个memory bank存储4096个样本embed feature(128-dimention) 随着网络更新, 目的是让特征在嵌入空间中远离(每一个样本都是一个类)，学习那种有监督时类和类之间相似聚集的现象。





---

### BYOL  - 2020/6

***Bootstrap Your Own Latent A New Approach to Self-Supervised Learning***



```python
无监督学习： {
    判别式：从增强视图的表示中，他们学会区分同一图像的另一个增强视图的表示和不同图像的增强视图的表示 => 这种判别方法通常需要将增强视图的每个表示与许多反例进行比较。
    生成式：通过预测同一图像的不同视图（例如，不同的随机裁剪）来学习表示 => 图像的增强视图的表示应该能够预测同一图像的另一个增强视图的表示。
}
```



***方法：***

![image-20240508172636506](http://sthda9dn6.hd-bkt.clouddn.com/Fgnq_P3obfLx4Ys4EVFMcEvtdv14)

在线网络θ + 目标网络γ(提供回归目标，γ = α×γ + (1-α)×θ ，指数移动平均 )

**yθ**是目标编码器，其余的训练好后丢掉

 ⭐一张图片的两个增强表示的相同的语义 => 在高维的嵌入表示中，应该可以预测对方(相近)

```python
yθ：Encoder
zθ：Projection head
qθ：Prediction head
```



***Loss:***
$$
注意图像增强t(x)和t^{`}x会对等的传给online\ net 和 target\ net\\
Loss: 1/2 × ( || q_{θ}(z_{θ}) - z^{`}_{ξ}||^{2}  + || q_{θ}(z_{θ}) - z^{`}_{ξ}||^{2}  )
$$
***Train:***
$$
θ：online\ net\ parameters\\
ξ：target\ net\ parameters\\
θ <- optimizer(θ),\ \ \ ξ <- αξ + (1-α)θ
$$




---

### DINO - 2021

***Emerging Properties in Self-Supervised Vision Transformers***



![image-20240508213644391](http://sthda9dn6.hd-bkt.clouddn.com/FgmNTLztdwi5Fz9jueQRIveYY2PT)

ViT最后的CLS注意力图示⭐⭐⭐



探讨：*质疑自监督学习是否为 Vision Transformer (ViT) 提供了比卷积网络 (convnets) 更突出的新属性？*



框架：(借鉴BYOL)

![image-20240508213533118](http://sthda9dn6.hd-bkt.clouddn.com/FoMugd-rgr0cSAJqhDni4V7Xx0mV)

教师是在训练过程中动态构建的。知识蒸馏就不再被用作自监督预训练的后处理步骤，而是直接作为自监督目标。

其中学生和教师具有相同的架构并在训练期间使用蒸馏。

教师在我们工作中用学生的动量平均值进行更新。

增强策略：

	1. 多裁剪策略构建图像的不同扭曲视图或裁剪。 更准确地说，根据给定的图像，我们生成一组 V 的不同视图。 该集合包含两个全局视图 xg 1 和 xg 2 以及几个分辨率较小的局部视图。
	1.  所有的裁剪都通过学生传递，而只有全局观点通过老师传递，因此鼓励“局部到全局”的对应。



伪代码：

![image-20240508213511774](http://sthda9dn6.hd-bkt.clouddn.com/FpwbfqXQctKIAjN6nzEdScMiH1Tv)

防止模型坍塌：

 	1. *对动量教师输出进行居中和锐化，以避免模型崩溃。*
 	2. **居中（Centering）**：对动量教师的输出进行居中操作是为了减少批次之间的偏差，增加输出的稳定性。具体做法是从每个输出中减去其均值，确保输出围绕零分布，这有助于避免网络输出在特征空间内偏向某一方向，从而降低了模型坍塌的风险。
 	3. **锐化（Sharpening）**：锐化是通过增加输出分布的峰值来实现的，目的是使模型的输出更加区分明显，即使不同类别之间的区别更加清晰。这通常通过提高输出概率分布的熵来实现，比如可以采用温度调整（temperature scaling）等方法来调整概率分布，使得主要的概率值更加突出，而其他的概率值则相对降低。



图示：

![image-20240508215946714](http://sthda9dn6.hd-bkt.clouddn.com/Fm-JcH7RKbGtr4JmV6THi9pBZpzJ)

*不同颜色是不同的注意力头*



![image-20240508220018510](http://sthda9dn6.hd-bkt.clouddn.com/FpiUx2RoGk6ES3YqU1cacRQmhQug)

无监督注意力更能学到本质！





---

### SimSiam - CVPR 2021

***Exploring Simple Siamese Representation Learning***

简单设计的Siamese(孪生)网络。 我们的极简主义方法的竞争力表明

![image-20240509130843870](http://sthda9dn6.hd-bkt.clouddn.com/Fuh4i3VZlmz43UK5fCBeVdASTTui)

1️⃣ “没有动量编码器的 BYOL”

2️⃣ “没有负样本的 SimCLR“ + stop-grad(⭐这个非常关键， 这个对防止模型坍塌很关键)



*方法：*

![image-20240509131933070](http://sthda9dn6.hd-bkt.clouddn.com/Fjpwyna05L85B-6wGDYZmz_Qw0MA)

一幅图像的两个增强视图由同一编码器网络 f（主干网络加投影 MLP）处理。 然后在一侧应用预测 MLP-h，在另一侧应用停止梯度操作。 该模型最大化了双方之间的相似性。 它既不使用负对也不使用动量编码器。



*伪代码：*

![image-20240509132032286](http://sthda9dn6.hd-bkt.clouddn.com/Fn0nqir1DgxbNEJEZxTwwyAticNl)







<em style="font-weight:bold; font-size: 24px">消融</em>

***Loss:***

*负的余弦相似度 和 交叉熵*

相似度：![image-20240509132907059](http://sthda9dn6.hd-bkt.clouddn.com/Fvq2llOYgXiD8my85xmHV4Qfciy2)

交叉熵：![image-20240509132841007](http://sthda9dn6.hd-bkt.clouddn.com/Fo-Wyh7xSpg3IJLeTKwtQKrrnJg0)

![image-20240509132922553](http://sthda9dn6.hd-bkt.clouddn.com/Fi1FIOIJnexoaKUFVFunkW1ACLD0)

![image-20240509132830913](http://sthda9dn6.hd-bkt.clouddn.com/FqfhYzetpsM5M-7uRQ67CWI2eeol)



***BatchNorm的影响：***

![image-20240509132146667](http://sthda9dn6.hd-bkt.clouddn.com/Fovin_S7qt5lFb027We7eEvPRQPd)

***BatchSize的影响：***

![image-20240509132226252](http://sthda9dn6.hd-bkt.clouddn.com/FohojIt8WTiIjRV1Pg1dgQVdX22J)

***预测头的影响：***

![image-20240509132304829](http://sthda9dn6.hd-bkt.clouddn.com/Fr8ykXKTMCL8sJr1AioPfrb10cgi)

***Loss的对称性：***

![image-20240509132550886](http://sthda9dn6.hd-bkt.clouddn.com/Fn0qB0BgCd8qAYGtw508QP7TYPGY)

sym对称；asym非对称；asym. 2×(每个图像采样两对来粗略地补偿对称性)





---



---

### Segment Anything

*即时分割*



*模型组件*

![image-20240615220259186](http://sthda9dn6.hd-bkt.clouddn.com/Fo1ZZ7uXEYg4FF6zhXP22Wb00GRI)



模型框架：

![image-20240615220352268](http://sthda9dn6.hd-bkt.clouddn.com/Fr4-lV5r1l6wvGHaV5X7ttwK-v-B)

*prompt encoder：*

- 	Sparse prompts: 
  - point: point => 256 dimensional vectorial embedding. 这个使用index去索引位置嵌入像Swin-T， foreground or backgroud embedding（自学习）. to add together.
  - box: 左上角位置编码 + 左上角的学习嵌入；左上角位置编码+“右下角”的学习嵌入
  - text: clip的text encoder.
- dense prompts:
  - mask: CNN => 256 特征向量。有则加mask，没有就加可学习的表示无mask的学习嵌入



*mask decoder：*

![image-20240615220440257](http://sthda9dn6.hd-bkt.clouddn.com/FjIJbCLSSY9RpRdpfUBwRHKDZ11H)







---

### KNN(K-Nearest Neighbors)

*K-近邻算法*

核心思想：**相似的样本具有相似的输出**。

=> KNN通过计算输入样本与训练数据集中所有样本的距离，找到距离最近的K个样本，然后根据这些样本的类别来决定输入样本的类别



主要步骤:

- **选择K值**：选择一个正整数K，代表你要比较的邻居数量。

- **计算距离**：对每个待分类样本，计算它与训练数据集中所有样本的距离。常用的距离度量有欧氏距离、曼哈顿距离和余弦相似度等。

  - $$
    \textbf{欧氏距离}: d(x, x_i) = \sqrt{\sum_{j=1}^{m}(x_j-x_{ij})^2}
    $$

- **选择最近的K个邻居**：根据计算得到的距离，从训练数据集中选择距离待分类样本最近的K个样本

- **投票或加权**：在分类任务中，K个邻居中最多的类别即为待分类样本的预测类别。在回归任务中，可以对K个邻居的数值进行平均或者加权平均。
- **输出结果**：输出投票或加权后的结果作为待分类样本的预测结果。





---

### PatchTST - ICLR 2023

1. Patchify .  
   1. 有监督 => 可以重叠
   2. 自监督 => 不可以重叠，避免网络可以从重叠区域走捷径学习

2. 多变量独立：

   1. 每个时间序列将有自己的潜在表示，通过共享权重机制交叉学习 ？？？ 

      共享Encoder权重，不同通道使用相同的模型参数。这种方法允许模型在不同的任务之间共享知识

**Overview**

![image-20240801210057622](http://sthda9dn6.hd-bkt.clouddn.com/Fq0nT1xy9-5ka1qxWndHierUZxTr)



![image-20240801205504650](http://sthda9dn6.hd-bkt.clouddn.com/FoubmWQSlQXo0ibNAHq6LdsmBlGT)



```python
x = [batch, channel, length]
x = [batch, channel, num_token, len_token]
x = [batch*channel, num_sample, dim_hidden]

=> Transformer Encoder but residual attn # 

=> Linear head =>
```





---

### CrossFormer - ICLR 2023

TRANSFORMER UTILIZING CROSSDIMENSION DEPENDENCY FOR MULTIVARIATE TIME SERIES FORECASTING



创新点：

- 显示建模时间依赖关系 + 通道依赖关系
- 两阶段注意力 （时间：MHSA，通道：Router MHSA）



嵌入方式 and 依据：

![image-20240802201801644](http://sthda9dn6.hd-bkt.clouddn.com/FmcGiYT9kC7WvvSf5RRFZuugHNPj)

自注意力呈现小局部一致性，一坨而不是一个。



- 保持通道独立 ✔️
- 注意力优化     ✔️ 



***TwoStageAttention***

```python
# Step 1. Time Dependency
x = [batch, channel, length]
x = [batch, channel, num_patch, dim_patch] # <= DSW (Dimension-Segment-Wise)
x = [batch*channel, num_patch, dim_patch]  
y = TransformerEncoer(x, x, x) # <= capture time dependency

# Step 2. Channel Dependency
y = [batch, channel, num_patch, dim_patch]
y = [batch*num_patch, channel, dim_patch]
router = [1, num_router, dim_patch] # <= router
router => repeat => [*, num_router, dim_patch]
z = TransformerEncoder(router, y, y) # <= capture channel dependency   
z = TransformerEncoder(y, router, router)

# save per stage output
=> Unet Decoder => to predict
```



ECG与多变量的异同：

- 相似点
  - 不同通道贡献不同 => DSW-patch, 能够更加细粒度编码局部波形  == 多变量(通道)
  - patch化的成功！！！
- 不同点
  - 由于是对心脏电活动的同一时间不同角度的观察 => 病理位置相同 =>是否能够通过**共享**策略 降低计算成本🤔❓





---

### Informer - AAAI 2021 Best 

-- TopK-Q

观察：(Q&K 是等价的)

![image-20240803141041758](http://sthda9dn6.hd-bkt.clouddn.com/FsuQLyF7-jRZhDX_VBbBjyVuOJYE)

![image-20240803141402493](http://sthda9dn6.hd-bkt.clouddn.com/FrWzYw4K5yiCpTHywdjy8lbdsTJc)

注意力呈现长尾分布：

Query分为活跃于惰性Token



衡量指标：

![image-20240803141306326](http://sthda9dn6.hd-bkt.clouddn.com/FnyVbctz6tBWyu_6v-9eFxtgAmsL)



注意力优化：

```python
Q, K, V
# probe
K = [batch, num_head, len_token, dim_head]
K_sample = [batch, num_head, random_len, dim_head]
Q_K_sample = [batch, num_head, len_token, random_len]
M = Q_K_sample.max(-1)[0] - torch.div(Q_K_sample.sum(-1), L_K)  # 衡量指标
M_top = M.topk(n_top, sorted=False)[1]

Q_reduce = Q[:, :, M_top, :]
attn_active = softmax(torch.matmul(Q_reduce, K.transpose(-2, -1))*scale)

contex = V.sum(-2).expand() => [batch, num_head, len_token, dim_head]  # 均匀分布的就直接取V的均值
contex[:, :, M_top, :] = attn_active@V
```



结构优化：

![image-20240803142458572](http://sthda9dn6.hd-bkt.clouddn.com/FjJea7jjH25S7tZe7BywKypEuXlv)

```python
# Encoder:
for num_layer ...:
    x = attn(x)
    x = conv_layer(x) # <- maxpool(act(norm(conv())))
return enc_out
```

```python
# Decoder:
cross = enc_out
x = # 预测引导
for num_layer ...:
    x = layer(x, cross, x_mask=x_mask, cross_mask=cross_mask)  # Note：Masked MHSA-ProbeAttn
```



框架逻辑

```python
Class Exp_Basic(Object):
    def __init__():
    def _build_model():
    def _acquire_device():
    def _get_data():
    def train():
    def vali():
    def test(self):
        
Class Exp_Model(Exp_Basic):
    def _select_optimizer():
    def _select_criterion():
    def _process_on_batch():
```

```python
# data_loader.py
Class Dataset_XXX(Dataset):
    def __init__():
    def __read_data__(self):
    def __getitem__(self, index):
    def __len__(self):
```

```python
# main.py

parser = argparse.ArgumentParser(description='[Model] Task')
...
args = parser.parse_args()
setting = ...args

exp = Exp_Model(args)
exp.train(setting)
exp.test(setting)
```

<img src="http://sthda9dn6.hd-bkt.clouddn.com/Fr4gDWDQ5ks4NCuXHRCQ-UOBW3f6" alt="image-20240817204741361" style="zoom: 50%;" />



---

### Adaptive Token Dictionary  - CVPR2024

Transcending the Limit of Local Window: Advanced Super-Resolution Transformer with Adaptive Token Dictionary



扩展局部窗口的限制

![image-20240919153745713](http://sthda9dn6.hd-bkt.clouddn.com/FvL2v53zt2HNJxfaf6DoqKEZxUQv)

1. window-based self-attention
2. token dictionary cross-attention => Attention(Q(XW),K(TW), V(TW))
3. 基于2的Attn，将token map排序分group(类)，进行group内部的Attention



![image-20240919154126696](http://sthda9dn6.hd-bkt.clouddn.com/Fm6YWCM2vD3r3UYGENn7woDqo_2r)



**Architecture**

![image-20240919154258809](http://sthda9dn6.hd-bkt.clouddn.com/FsliGkn-Ia9KlymRQrFUZZaff_H5)





---

### Poly Kernel Inception  - CVPR 2024

Poly Kernel Inception Network for Remote Sensing Detection

遥感图像中的目标检测面临着多种挑战，包括**目标尺度变化大**、测距环境多样等。现有的方法试图通过大核卷积或扩张卷积来扩展脊柱的空间感受野来解决这些挑战。然而，前者通常会**引入相当大的背景噪声**，而后者则有生成**过度稀疏的特征表示**的风险。本文提出了一种多核初始化网络（PKINet）来解决上述问题。PKINet采用无膨胀的多尺度卷积核来提取**不同尺度**的对象特征并捕获**局部上下文**。此外，一个上下文锚注意（CAA）模块并行引入捕获远程上下文信息。

1. **不同尺度-局部上下文**
2. **并行引入捕获远程上下文信息**



![image-20240919154538153](http://sthda9dn6.hd-bkt.clouddn.com/FjXAvi5BOhnK1vn-qVoFbI57Ajvo)

```python
# 十字架型汇聚 => 近似标准的DWConvKxK => 降低参数量
agg = Conv1x1(AvgPool(X))
agg = Conv1x1(DWConvKx1(DWConv1xK(agg)))
attn = Sigmoid(agg) 
```





---

### DANet  - CVPR 2019

Dual Attention Network for Scene Segmentation



![image-20240925140803581](http://sthda9dn6.hd-bkt.clouddn.com/Fv2w3idnvTYq2ouUvrQMtxqlfucH)

At the end of the model, we use the dual attention mechanism to explicitly capture position and channel dependencies.

```python
class PAM_Module(Module):
    """ Position attention module"""
    #Ref from SAGAN
    def __init__(self, in_dim):
        super(PAM_Module, self).__init__()
        self.chanel_in = in_dim

        self.query_conv = Conv2d(in_channels=in_dim, out_channels=in_dim//8, kernel_size=1)
        self.key_conv = Conv2d(in_channels=in_dim, out_channels=in_dim//8, kernel_size=1)
        self.value_conv = Conv2d(in_channels=in_dim, out_channels=in_dim, kernel_size=1)
        self.gamma = Parameter(torch.zeros(1))

        self.softmax = Softmax(dim=-1)
    def forward(self, x):
        """
            inputs :
                x : input feature maps( B X C X H X W)
            returns :
                out : attention value + input feature
                attention: B X (HxW) X (HxW)
        """
        m_batchsize, C, height, width = x.size()
        proj_query = self.query_conv(x).view(m_batchsize, -1, width*height).permute(0, 2, 1)
        proj_key = self.key_conv(x).view(m_batchsize, -1, width*height)
        energy = torch.bmm(proj_query, proj_key)
        attention = self.softmax(energy)
        proj_value = self.value_conv(x).view(m_batchsize, -1, width*height)

        out = torch.bmm(proj_value, attention.permute(0, 2, 1))
        out = out.view(m_batchsize, C, height, width)

        out = self.gamma*out + x
        return out


class CAM_Module(Module):
    """ Channel attention module"""
    def __init__(self, in_dim):
        super(CAM_Module, self).__init__()
        self.chanel_in = in_dim


        self.gamma = Parameter(torch.zeros(1))
        self.softmax  = Softmax(dim=-1)
    def forward(self,x):
        """
            inputs :
                x : input feature maps( B X C X H X W)
            returns :
                out : attention value + input feature
                attention: B X C X C
        """
        m_batchsize, C, height, width = x.size()
        proj_query = x.view(m_batchsize, C, -1)
        proj_key = x.view(m_batchsize, C, -1).permute(0, 2, 1)
        energy = torch.bmm(proj_query, proj_key)
        energy_new = torch.max(energy, -1, keepdim=True)[0].expand_as(energy)-energy
        attention = self.softmax(energy_new)
        proj_value = x.view(m_batchsize, C, -1)

        out = torch.bmm(attention, proj_value)
        out = out.view(m_batchsize, C, height, width)

        out = self.gamma*out + x
        return out
```





---

### MixNet 

*MixConv: Mixed Depthwise Convolutional Kernels*



![image-20240925145109345](http://sthda9dn6.hd-bkt.clouddn.com/FtRY7ZtihlPgnE0gXKTYRzWioi9l)

convulution => capture local pattern

- early stages: edges
- later stages: objects⭐



这项研究表明了单个内核大小的局限性：我们既需要大内核来捕获高分辨率模式，也需要小内核来捕获低分辨率模式，以获得更好的模型精度和效率

在单一机制下实现多种效果，进行增强



---

### Multimodal Learning 

Multimodal Learning With Transformers: A Survey



**融合策略**

![image-20240925151957579](http://sthda9dn6.hd-bkt.clouddn.com/Fpkl7hX0gQn4hTn-9pCn7v0eSDsx)





---

### Dual Aggregation Transformer

Dual Aggregation Transformer for Image Super-Resolution

Motivation: 现有方法利用自我注意沿着不同的维度，空间或通道，并取得了令人印象深刻的性能。这启发我们将Transformer中的两个维度结合起来，以获得更强大的表示能力。

![image-20241004172905131](http://sthda9dn6.hd-bkt.clouddn.com/FkgqHcOiU-RhuZ2OuevAk2UbTecu)

![image-20241004172825801](http://sthda9dn6.hd-bkt.clouddn.com/Fr9yfe5d0MlT-gv6julT2CWe0wme)





---

### Dual Vision Transformer

研究全局语义和更精细的像素级特征之间的依赖关系 => pixel-level token  &  semantic token

分解和集成的全局语义和本地功能

![image-20250114144508261](http://sthda9dn6.hd-bkt.clouddn.com/FioBD2_JVUt1b_3OHzNhyt1y4Qyx)

![image-20241030220700886](http://sthda9dn6.hd-bkt.clouddn.com/FuAjYWuMPbxeo1UKHwCopbn3dyf8)



---

### Fish-Speech Tech-report

Text-to-Speech End2End Model



**两阶段训练策略：**

<img src="http://sthda9dn6.hd-bkt.clouddn.com/Fl0WmE6EneKMjE67yM20TWC7qsrz" alt="image-20250215131815505" style="zoom: 50%;" />

*Stage 1:*

	Audio:Mel Spectrogram => 【Encoder】 => Embedding => Quantize Tokens => 【⭐Decoder⭐】=> Audio
	
	**⭐重构目标⭐**

*Stage 2:*

	Text:Quantize Tokens => 【✨AR Model✨】=> Quantize Tokens  
	
	⭐**Text:Audio一致性 + 自回归预测Next**⭐

*Inference:*

	Text:Prompt-Tokens => 【✨AR Model✨】=> Quantize Tokens  => 【⭐Decoder⭐】=> Audio



*Vector Quantize Tech:*

Example：

```python
有一组连续的温度数据（如 20.3°C, 21.7°C, 22.5°C, 19.8°C），你想将其离散化为几个类别:
1.低温: 15°C - 20°C
2.中温: 20°C - 25°C
3.高温: 25°C - 30°C
=>
20.3°C → 中温
21.7°C → 中温
22.5°C → 中温
19.8°C → 低温

假设编码本有 512 个向量 Shape:[512, dim]
Encoder得到的Embedding Shape:[T, dim]
对于 Encoder 输出的每个时间步的特征向量，VQ 会找到编码本中与之最接近的向量，并用其索引表示。
最终输出是一个离散的索引序列，例如 [42, 123, 87, ...]，每个索引对应编码本中的一个向量。

编码本随机初始化，在训练过程中，编码本会通过梯度下降和优化算法（如 Adam）不断更新：
最近邻搜索 => 量化误差计算 => 梯度更新,BP
编码本的作用 => 降维与压缩 + 离散化表示 + 提升生成质量(通过离散化减少生成过程中的模糊性，提升生成语音的自然度s)
```



---

### Blip

- Image-2-Text 任务之一

Bootstrapping Language-Image Pre-training for Unified Vision-Language Understanding and Generation Architecture



<img src="http://sthda9dn6.hd-bkt.clouddn.com/FpPP6kQ3PE7BEtpByp9YYxMRML6v" alt="image-20250215133226745" style="zoom:50%;" />

相同颜色共享参数



Stage 1：

Image => 【Image Encoder:ViT】 => image Embedding

Text => 【Text Encoder:Bert】 => text Embedding

目标：图文一致性, 训练Encoder



Stage 2：

Image => 【Image Encoder:ViT:Freeze🥶】 => image Embedding 

Text => 【Text Encoder:Bert:Freeze🥶 + Cross-Attention:image Embedding🥵】 => Linear:2class

目标：图文是否匹配-2分类, 训练Cross-Attention部分



Stage 3:

Image => 【Image Encoder:ViT:Freeze🥶】 => image Embedding 

Text => 【Text Decoder:GPT🥵 + Cross-Attention:image Embedding:Freeze🥶】 => Linear:multi-class

目标：Image Embedding + text 自回归预测Next



Inference

Image => 【⭐Image Encoder⭐】 => image Embedding 

Prompt-Text => 【⭐Text Decoder:GPT⭐ + Cross-Attention:image Embedding:⭐】 => Linear:multi-class  => Next-Token

实现Image => Text



---

### BEVFormer

BEVFormer: Learning Bird’s-Eye-View Representation from Multi-Camera Images via Spatiotemporal Transformers

网络架构信息流思路：

![image-20250310223908226](http://sthda9dn6.hd-bkt.clouddn.com/FhrytoVkAZHPxv74ssV8XYRfxwe-)

具体：

![image-20250310223507001](http://sthda9dn6.hd-bkt.clouddn.com/FiHzv4f6t8ftO97JsQ2A8Kqg4ad_)

- 一组可学习的BEV Queries，二维网格，模拟鸟瞰图；
- Spatial Cross Attention，每个视图经过backone提取，拿其中多个层级的输出，拼接为多尺度特征(多个层的特征图，校准通道)。然后每个位置的q，只查询对应几个视图的周边几个k；
- Temporal Attention，t时刻的BEV中的q，查询t-1时刻，相应位置周边的几个k。 这里的t-1时刻的BEV特征，需要通过一个角度还是啥校准空间对齐；



---

### Deformable DETR

architecture figure：

<img src="http://sthda9dn6.hd-bkt.clouddn.com/FsCz6WFOAG-lyH--baQzssLcXHQl" alt="image-20250310230354425" style="zoom:50%;" />

Attn figure：

<img src="http://sthda9dn6.hd-bkt.clouddn.com/Fj3SPOZCjcvybFaLv1hoUNeGlail" alt="image-20250310230305133" style="zoom: 50%;" />

```python
# 伪代码 - 单尺度的
import torch
import torch.nn as nn
import torch.nn.functional as F

class DeformableAttention(nn.Module):
    def __init__(self, embed_dim, num_heads, num_points):
        super().__init__()
        self.embed_dim = embed_dim
        self.num_heads = num_heads
        self.num_points = num_points

        # 用于预测采样偏移的线性层
        self.offset_proj = nn.Linear(embed_dim, num_heads * num_points * 2)

        # 用于计算注意力权重的线性层
        self.attn_proj = nn.Linear(embed_dim, num_heads * num_points)

        # 输出投影层
        self.out_proj = nn.Linear(embed_dim, embed_dim)
	
    # reference_points, 每个采样点初始，共用同一个reference_point，靠offset进行局部位置偏移
    def forward(self, query, reference_points, value):
        B, N, C = query.shape
        H, W = value.shape[-2:]

        # 预测采样偏移
        offsets = self.offset_proj(query).view(B, N, self.num_heads, self.num_points, 2)

        # 生成采样点
        sampling_points = reference_points.unsqueeze(2) + offsets

        # 双线性插值采样特征值
        sampled_value = F.grid_sample(
            value,
            sampling_points.view(B, -1, H, W, 2),
            mode='bilinear',
            align_corners=True
        ).view(B, N, self.num_heads, self.num_points, C // self.num_heads)

        # 计算注意力权重
        attn_weights = self.attn_proj(query).view(B, N, self.num_heads, self.num_points)
        attn_weights = F.softmax(attn_weights, dim=-1)

        # 加权求和
        output = (sampled_value * attn_weights.unsqueeze(-1)).sum(dim=3)
        output = output.view(B, N, C)

        # 输出投影
        output = self.out_proj(output)

        return output
```