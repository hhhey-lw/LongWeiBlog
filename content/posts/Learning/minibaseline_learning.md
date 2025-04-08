---
author: "LongWei"
title: "DL基准模型训练伪代码"
date: "2024-10-31"
tags: ["DL"]
categories: ["Learning"]
series: ["Learning"]
ShowToc: true
TocOpen: true
---

### minGPT

*author: karpathy Andrej*



**work flow**

```python
String: ['X1', 'X2', ..., 'Xn'] =[dictionary]=> [k1, k2, ..., kn]  # word2index

# Train:    
X = [X1, X2, X3, Y1, Y2], Y = [-1, -1, Y1, Y2, Y3]
Model = {(WordEmbed+PosEmbed) => (CausalSelfAttention+FFN)xN => (ClassHead)}
#  attn =   A B C
#         A √ × ×       uni-directional
#         B √ √ ×  
#         C √ √ √

Z = Model(X) => [Z1, Z2, Z3, Z4, Z5]
Y             = [-1, -1, Y1, Y2, Y3]
# Equal to        Parallel Train
(X1,X2,X3)        =predict=> Y1  
(X1,X2,X3,Y1)     =predict=> Y2
(X1,X2,X3,Y1,Y2)  =predict=> Y3

loss = crossentropy(Z, Y, ignore=-1)  # Zi => {P1,...,Pclass} multi-class task

# Test:  Serial generation
give inp = [X1, X2, X3]
Step 1: 
    Z = Model(inp) = [Z1,Z2,Z3]
    Y              = [-1,-1,Y1]
    inp = inp + Y[-1] = [Z1,Z2,Z3,Y1]
Step 2:
    Z = Model(inp) = [Z1,Z2,Z3,Z4]
    Y              = [-1,-1,-1,Y2]
    inp = inp + Y[-1] = [Z1,Z2,Z3,Y1,Y2]
Step 3:
    Z = Model(inp) = [Z1,Z2,Z3,Z4,Z5]
    Y              = [-1,-1,-1,-1,Y3]
    inp = inp + Y[-1] = [Z1,Z2,Z3,Y1,Y2,Y3]
    
out: [Z1,Z2,Z3,Y1,Y2,Y3]
```



---

### minBert

**work flow**



*pre-trainning*

```python
# Data preparation
data = [{"S1", "S2", ..., "Sn"},                   [{k1, k2, ..., kn},
        {"S1", "S2", ..., "Sn"},    =vocabulary=>  [{k1, k2, ..., kn},
        {"S1", "S2", ..., "Sn"}]                   [{k1, k2, ..., kn},

# Mask-Word & Predict isNext
while positive != batch_size / 2 or negative != batch_size / 2: # isNext,notIsNext sample number is equality
	random-sentence-A, random-sentence-B => tokens_a, tokens_b
    input_ids = [word_dict['CLS']] + tokens_a + [word_dict['SEP']] + tokens_b + [word_dict['SEP']]
    seg_ids = [0] * (1 + len(tokens_a) + 1) + [1] * (len(tokens_b) + 1)
    
    # fixed input length => padding ['Pad']
    input_ids extend [word_dict['Pad']] * num_pad  
    seg_ids extend [0] * num_pad
                                                    
    => masked_pos, masked_tokens
    => isNext
=> [{input_ids,seg_ids,masked_pos,masked_tokens,isNext},
    {...}...]
                                                    
attn_pad_mask = [0,..,0,1,..,1]

Bert = {(Token_embed+Pos_embed+Seg_embed) => {MHSA[attn_pad_mask]=>FFN}xN 
        => {head1(dim_token->2)-o1:isNext, head2(dim_token->vocab_len)-o2:predict_masked_word}}
#  attn =   A B P       P->Padding
#         A √ √ ×       bi-directional
#         B √ √ ×  
#         P √ √ ×
                                                    
loss1 = CrossEntropyLoss(o1,isNext)
loss2 = CrossEntropyLoss(o2,masked_tokens)
```

*fine-tuning*

- input: [CLS] + Token-A + [SEP],  get [CLS]  # 单独
- input: [CLS] + Token-A + [SEP] + Token-B + [SEP], get => [SEP] + Token-B + [SEP]  # 关联关系



---

### minUnet



*work flow*

```python
# Model  U-shape feature-flow
input => [Enc1] => [Dec6] => output  # Pixel-Level semantic token => binary classification
         [Enc2] => [Dec5]
         [Enc3] => [Dec4]
```

```python
# Training:
for data in dataloader:
    inp, mask = data
    oup = model(inp)
    
    loss = BCELoss(oup, mask)
    ...
```



---

### minYolo



```python
X:(1,3,H,W) => Model => Y:(K,K,B*5+C)
        
[k1,k2,k3]
[k4,k5,k6] =>  [X,Y,W,H,Confidence,C1,...,Cn]
[k7,k8,k9]

# Inference  (x, y) (w, h) confidence prob-class[0,...,79]{80-category}
# Step 1
conf_mask = (prediction[:, :, 4] > confidence).float().unsqueeze(2)  # 根据confidence判断该锚框是否存在对象
prediction = prediction * conf_mask
# Step 2
# 处理x,y,w,h => (x1,y1,x2,y2)

# 若B>1: => 重塑为 (KK+N,5+C)

# 整理成 [X1,Y1,X2,Y2,idx_class,class_score] <= 存在对象

# 进行NMS非极大值抑制
for img : per_img:
    # 根据 class_score 排序
    # 计算当前预测框与后续预测框的IoU
    # 根据 iou ? nms_conf 进行去除冗余的框

锚框数据：[X1,Y1,X2,Y2,idx_class,class_score]    


# Inference
label: [[idx_class,x,y,w,h],[...]]  => [[X,Y,W,H,Confidence,C1,...,Cn],[...]]
# Yolo-v1: 将锚框均匀的等分为7×7的Cell

X:(1,3,H,W) => Model => P:(K,K,5+C)  Y:(K,K,5+C)
 
# Loss
分三步：
1. 锚框Loss : x,y and w,h
2. Confidence-Loss: obj and noobj
3. class-loss: Ci

# 具体做法就是 
1. 按照Confidence>0.5 => 是否存在对象
2. 收集所需要的Cell => 分别计算Loss
```

![image-20241102213134197](http://sthda9dn6.hd-bkt.clouddn.com/Fkz-1AneHRs4XMsndjM2zkbfmd8K)





---

