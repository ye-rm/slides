---
# You can also start simply with 'default'
theme: neversink
layout: cover
routerMode: hash
color: light
---

# InfiniGen

**Efficient Generative Inference of Large Language Models  with Dynamic KV Cache Management**


---
layout: default
---

# Background

![20250507143854](https://markdown-1308105459.cos.ap-beijing.myqcloud.com/Typora/20250507143854.png)

LLM推理过程中需要KV-Cache存储大量的中间计算结果➡️提高上下文长度/批处理需要占用大量显存

---
layout: two-cols-title
---

::title::

# Prior works

Quantization/Eviction

::left::

## Quantization

![20250507145021](https://markdown-1308105459.cos.ap-beijing.myqcloud.com/Typora/20250507145021.png)

::right::

## Eviction

![20250507145154](https://markdown-1308105459.cos.ap-beijing.myqcloud.com/Typora/20250507145154.png)


---
layout: two-cols-title
---

::title::

# Limitations of Prior Approaches

::left::

![20250507150806](https://markdown-1308105459.cos.ap-beijing.myqcloud.com/Typora/20250507150806.png)

::right::

![20250507150857](https://markdown-1308105459.cos.ap-beijing.myqcloud.com/Typora/20250507150857.png)




---
layout: two-cols-title
---

::title::

# Limitations of Prior Approaches

::left::

![20250507151739](https://markdown-1308105459.cos.ap-beijing.myqcloud.com/Typora/20250507151739.png)

::right::

<br/>
<br/>
<br/>

 - KV-Cache仍然线性增长
 - 精度丢失
 - 资源浪费

🤔考虑把KV-Cache搬到内存


---
layout: two-cols-title
---
::title::

# KV Cache Offloading

::left::

<br/>
<br/>

![20250507152408](https://markdown-1308105459.cos.ap-beijing.myqcloud.com/Typora/20250507152408.png)

::right::

<br/>
<br/>

### GPU memory: Expensive & Small

💵：
$3 \times RTX 3090 \approx RTX A6000$ 

The same `GA102` core, only more 24GB memory

<br/>

### CPU memory: Cheap  & Large

Easy to expand

---
layout: default
---

# Problem of Offloading

![20250507155025](https://markdown-1308105459.cos.ap-beijing.myqcloud.com/Typora/20250507155025.png)

---
layout: two-cols-title
columns: is-7
---

::title::
# InfiniGen

::left::
![20250507213108](https://markdown-1308105459.cos.ap-beijing.myqcloud.com/Typora/20250507213108.png)

::right::

<br/>
<br/>
<br/>

### **Dynamic KV Cache Management**

 - **Transfer Less**: Load and compute only with a few important tokens
 - **Transfer Early**: Prefetch essential KV entries in the preceding layer

---
layout: two-cols-title
---

::title::

# Prefetching Opportunities
### Attention Input Similarity
::left::

![20250507213728](https://markdown-1308105459.cos.ap-beijing.myqcloud.com/Typora/20250507213728.png)

::right::

<br/>
<br/>

![20250507213808](https://markdown-1308105459.cos.ap-beijing.myqcloud.com/Typora/20250507213808.png)

::default::

layer `i-1` 的输入经过`layer-normalized`作为 layer `i` 的输入，layer `i` 的很大程度受到layer `i-1` 的影响

---
layout: two-cols-title
---

::title::
# Prefetching Opportunities
### Skewed Partial Weight

::left::

![20250507214626](https://markdown-1308105459.cos.ap-beijing.myqcloud.com/Typora/20250507214626.png)

::right::

<br/>

the attention score highly depends on a few columns in the query and key matrices

<br/>
make a few columns in the query and key matrices have much larger magnitude than others, a much smaller number of columns significantly affects the attention pattern.

---
layout: two-cols-title
---

::title::
# Prefetching Opportunities
### Skewed Partial Weight

::left::
multiplying the query and key weight matrices with the same orthogonal matrix A

![20250507220724](https://markdown-1308105459.cos.ap-beijing.myqcloud.com/Typora/20250507220724.png)

::right::
![20250507222819](https://markdown-1308105459.cos.ap-beijing.myqcloud.com/Typora/20250507222819.png)

::default::

## To make it align with the direction that Q stretches the most

---
layout: default
---

# Efficiently Prefetching KV Cache

![20250507225142](https://markdown-1308105459.cos.ap-beijing.myqcloud.com/Typora/20250507225142.png)

Prefill 阶段对LLM每层的Query矩阵进行SVD分解，乘上矩阵A并保存调整后的模型权重（如上一页）

---
layout: default
---

# Prefill Stage

![20250507225634](https://markdown-1308105459.cos.ap-beijing.myqcloud.com/Typora/20250507225634.png)

### 对Query和Key矩阵取绝对值相加，取出Top-K大的列，得到Partial Query矩阵

---
layout: default
---

# Decoding Stage

![20250507230142](https://markdown-1308105459.cos.ap-beijing.myqcloud.com/Typora/20250507230142.png)

### 使用`Partial Query`矩阵进行运算，根据`i-1`层的输入选择`i`层预取的Token

---
layout: default
---

# Experimental Setup

![20250507230758](https://markdown-1308105459.cos.ap-beijing.myqcloud.com/Typora/20250507230758.png)

