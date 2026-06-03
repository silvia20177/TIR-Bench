# TIR-Bench 详细解读：专为"以图推理"能力设计的多模态评测基准

> **导读**：TIR-Bench（Thinking-with-Images Reasoning Benchmark）是一个专门评测多模态大模型"图像推理"能力的基准，包含 1215 个样本、13 项任务，核心亮点是引入工具调用评测维度——模型不仅要"看懂图"，更要"边用工具边推理"。目前最强结果由 o3-TU（o3 + 代码解释器）以 **46.0%** 的准确率刷新，绝大多数主流模型仍低于 30%。

---

## 一、为什么需要 TIR-Bench？

现有多模态评测基准（如 MMBench、MMMU）主要考察模型对图像内容的静态理解，而忽视了一种关键能力：**模型能否主动调用工具对图像进行操作、分析，并在多轮工具交互中完成复杂推理任务**。

TIR-Bench 将这一能力定义为 **Thinking with Images（TIR）**， 区别于传统的 Chain-of-Thought（只用文字推理，图像是静态输入），Thinking-with-Images 让模型在推理链中主动操作图像：

  - 写代码生成图像处理工具（如 OpenAI o3）
  - 执行代码修改图像（裁剪、旋转、增强对比度、画辅助线等）
  - 将处理后的新图像重新注入上下文，继续推理
为此精心设计了 13 项评测任务，覆盖从基础感知到多步工具链推理的完整能力谱系。 
---

## 二、基准概览

| 维度 | 数值 |
|------|------|
| 总样本数 | 1215 |
| 任务数 | 13 |
| 选择题 | 665 道 |
| 开放作答题 | 550 道 |
| 问题平均长度 | 46.29 字符 |
| 答案平均长度 | 3.41 字符 |

各任务样本量分布如下：

| 任务 | 样本量 |
|------|--------|
| Color（颜色感知） | 100 |
| Low-Light（低光图像问答） | 50 |
| Instrument Reading（仪器读数） | 80 |
| Jigsaw（拼图复原） | 120 |
| Math（数学几何推理） | 120 |
| Maze（迷宫路径规划） | 120 |
| Rotated OCR（旋转图像文字识别） | 60 |
| Proportion（目标占比估算） | 120 |
| Rotation（图像旋转矫正） | 75 |
| Spot the Difference（找不同） | 100 |
| Symbolic Reasoning（符号逻辑推理） | 50 |
| Visual Search（高分辨率视觉检索） | 120 |
| Word Search（字符网格单词搜索） | 100 |
| **Total** | **1215** |

---

## 三、13 项任务详解

### 3.1 Color（颜色感知）
**核心挑战**：模型需通过编写并运行代码，对图像进行程序化处理，统计特定颜色的像素占比后才能作答。

- 示例问题：`What is the number in the center of this image? Select from the following choices.`
- 考察能力：编程辅助视觉分析、颜色统计

---

### 3.2 Low-Light（低光图像问答）
**核心挑战**：面对曝光不足或细节被遮挡的图像，模型需先识别画质缺陷，通过编写代码提升对比度、亮度，再精准回答图像内容相关问题。

- 示例问题：`How many ducks are in the image?`
- 考察能力：图像增强、低质量视觉理解

---

### 3.3 Instrument Reading（仪器读数）
**核心挑战**：模型需先定位仪器的关键读数区域，再通过程序化裁剪/放大特定区域以提升清晰度，最终从增强后的视图中精准提取数值。

- 示例问题：`What is the stopwatch second reading in the image? Provide an integer.`
- 考察能力：多轮工具辅助精细分析

---

### 3.4 Jigsaw（拼图复原）
**核心挑战**：模型需将图像程序化分割为若干块，通过反复尝试重组、持续的行动与自我修正循环，评估每次组合是否已复原原图。

- 示例问题：`Determine the correct arrangement to restore the original image.`
- 考察能力：基于迭代工具流程的复杂空间推理

---

### 3.5 Math（数学几何推理）
**核心挑战**：模型需使用工具绘制辅助构造线、建立坐标系，进而计算长度、比例等几何属性，解决图形中的数学问题。

- 示例问题：`Which arch is the longest? A. AB  B. BC  C. CD  D. DE  E. EG  F. GA`
- 考察能力：程序化扩充视觉输入、几何计算

---

### 3.6 Maze（迷宫路径规划）
**核心挑战**：模型需分析迷宫视觉结构，利用图像处理工具设计求解方案，应用寻路算法破解迷宫，并在图像上绘制求解路径。

- 示例问题：`Please complete a maze game starting from the red ball to the green ball.`
- 考察能力：高级空间规划与算法执行

---

### 3.7 Rotated OCR（旋转图像文字识别）
**核心挑战**：模型需先识别文本图像的方向错误，再使用工具旋转至正确姿态，最后通过光学字符识别精准读取内容。

- 示例问题：`What is written in the image?`
- 考察能力：多步视觉推理（方向感知 → 图像矫正 → OCR）

---

### 3.8 Proportion（目标占比估算）
**核心挑战**：需要调用高性能外部分割模型获取目标掩码，再通过程序计算目标相对于整张图像的像素占比。

- 示例问题：`Which of the following values is the closest to the proportion of the image occupied by chair in blue bottom right corner?`
- 考察能力：工具链调度、图像分割与面积计算

---

### 3.9 Rotation（图像旋转矫正）
**核心挑战**：模型需以迭代方式测试各种旋转角度，每次变换后视觉评估姿态是否正确，通过工具行动与视觉判断的循环完成求解。

- 示例问题：`How many degrees should you rotate this image CLOCKWISE to restore it to its original orientation?`
- 考察能力：迭代式方向矫正

---

### 3.10 Spot the Difference（找不同）
**核心挑战**：识别两张近乎一致图像的差异时，模型需采用工具化策略（如像素级差分运算），高亮显示差异区域，再定位并标记包含差异的图像分块编号。

- 示例问题：`Output the list of patch indices where the two images differ.`
- 考察能力：精准程序化视觉比对

---

### 3.11 Symbolic Reasoning（符号逻辑推理）
**核心挑战**：统计复杂多边形的边数时，模型无法仅凭猜测作答，必须系统性地识别并枚举每一条独立边，可能需要借助顶点、线条检测算法完成计数。

- 示例问题：`How many edges are there in this polygon?`
- 考察能力：视觉信息应用抽象规则逻辑

---

### 3.12 Visual Search（高分辨率视觉检索）
**核心挑战**：在复杂高分辨率图像中定位特定目标，需要开展迭代式搜索，策略性地使用工具反复放大、分析不同区域，直至定位目标或关键信息。

- 示例问题：`Is a brown sheep present to the left of the central white horse?`
- 考察能力：多轮深度视觉推理

---

### 3.13 Word Search（字符网格单词搜索）
**核心挑战**：图像中布满高度相似的字符，仅有少量细微差异，标准 OCR 在此场景下失效，迫使模型通过像素级比对、目标视觉检索等方式定位异常字符。

- 示例问题：`In the figure, in which row and column does the number 8 appear?`
- 考察能力：细粒度视觉判别与异常检测

---

## 四、数据格式说明

每条样本为一个 JSON 对象，字段含义如下：

```json
{
  "meta_data": {},          // 元信息（部分任务含 difficulty 字段）
  "task": "maze",           // 任务类型
  "answer": "D",            // 标准答案
  "prompt": "...",          // 问题文本
  "image_1": "data/maze/17.png",   // 主图路径
  "image_2": null,          // 辅助图路径（找不同任务使用）
  "id": 10                  // 样本 ID
}
```

**各任务典型样本示例：**

<details>
<summary>Proportion（目标占比估算）</summary>

```json
{
  "task": "refcoco",
  "answer": "A",
  "prompt": "Which of the following values is the closest to the proportion of the image occupied by man?\nA: 65%\nB: 73%\nC: 81%\nD: 89%\nE: 49%\nF: 57%",
  "image_1": "data/refcoco/COCO_train2014_000000305267.jpg",
  "id": 1
}
```
</details>

<details>
<summary>Jigsaw（拼图复原，含难度标注）</summary>

```json
{
  "meta_data": {"difficulty": 3},
  "task": "jigsaw",
  "answer": "[4, 1, 8, 9, 3, 6, 2, 7, 5]",
  "prompt": "Instructions: Please complete the jigsaw puzzle shown in the image. The original image has been divided into {n}×{n} pieces and scrambled...",
  "image_1": "data/jigsaw/shuffled_puzzle_3.jpg",
  "id": 8
}
```
</details>

<details>
<summary>Spot the Difference（找不同，需要两张图）</summary>

```json
{
  "task": "spot_difference",
  "answer": "7, 11, 12, 13, 14, 15, 19",
  "prompt": "Output the list of patch indices where the two images differ. Note that in this image, n is 4 and m is 5",
  "image_1": "data/spot_difference/79_left.png",
  "image_2": "data/spot_difference/79_right.png",
  "id": 11
}
```
</details>

---

## 五、评测流程

评测分为三个阶段，各阶段对应独立脚本：

```
TIRBench.json (1215 条样本)
        │
        ▼
[1] run_inference.py         → responses.json
        │   • 图像转 base64
        │   • 8 workers 多线程并发调用 MLLM
        │   • 支持 GPT-4o、Qwen 等主流模型
        ▼
[2] extract_answer.py        → answers.json
        │   • few-shot demo prompt + LLM
        │   • 从冗长模型回复中提取结构化答案
        ▼
[3] calculate_score.py       → results.json + scores.txt
            • 按任务类型使用差异化评分策略
            • 每条样本输出 true/false（0/1）
            • 汇总总分 + 分任务分数
```

---

## 六、主流模型评测结果

> **缩写说明**：SR=符号推理, WS=单词搜索, LL-VQA=低光VQA, IR=仪器读数, SD=找不同, JG=拼图, VS=视觉搜索, RG=旋转游戏, Pro.=目标占比
>
> **后缀说明**：`-TU` 表示 Tool-Using（配合代码解释器），无后缀为普通推理模式

### 综合排名

| 分组 | 模型 | 总分 | Color | Pro. | OCR | SR | Maze | Math | WS | LL-VQA | IR | SD | JG | VS | RG |
|------|------|------|-------|------|-----|----|------|------|----|--------|----|----|----|----|----|
| 随机基线 | Random Guess | 13.5 | 28.0 | 6.7 | - | 14.0 | 13.3 | 15.8 | 0.0 | 4.0 | 8.8 | 22.6 | 5.8 | 22.5 | 16.0 |
| **开源模型** | Llava-1.6-M-7B | 11.3 | 27.0 | 7.5 | 3.3 | 16.0 | 4.2 | 16.7 | 0.0 | 14.0 | 6.3 | 18.0 | 0.0 | 22.5 | 12.0 |
| | Qwen2.5-VL-72B | 19.7 | 37.0 | 15.0 | 33.3 | 24.0 | 35.0 | 22.5 | 3.0 | 32.0 | 12.5 | 14.1 | 0.0 | 25.8 | 12.0 |
| | InternVL3-78B | 21.4 | 25.0 | 21.7 | 3.3 | 26.0 | 32.5 | 23.3 | 8.0 | 28.0 | 16.3 | 18.9 | 5.8 | 39.2 | 26.7 |
| **闭源模型** | GPT-4.1 | 18.8 | 36.0 | 7.5 | 11.7 | 12.0 | 17.5 | 25.0 | 4.0 | 24.0 | 11.3 | 30.9 | 5.1 | 34.2 | 22.7 |
| | GPT-4o | 17.3 | 26.0 | 22.5 | 10.0 | 10.0 | 20.0 | 15.8 | 0.0 | 26.0 | 7.5 | 19.4 | 6.2 | 35.0 | 20.0 |
| | Gemini-2.5-Pro | 28.9 | 44.0 | 21.7 | 25.0 | 34.0 | 24.2 | 30.8 | 12.0 | 42.0 | 20.0 | 28.5 | 10.4 | **58.3** | 30.7 |
| | Grok-4 | 22.5 | 35.0 | **53.3** | 6.7 | 20.0 | 25.8 | 19.2 | 2.0 | 22.0 | 12.5 | 27.0 | 10.0 | 25.8 | 18.7 |
| | o3 | 26.9 | 36.0 | 34.2 | 8.3 | 34.0 | 29.2 | 24.2 | 4.0 | 28.0 | 17.5 | 37.2 | 10.8 | 47.5 | 33.3 |
| **工具增强** | PyVision | 31.8 | 53.0 | 26.7 | **63.3** | 54.0 | 15.8 | 25.8 | 10.0 | 32.0 | 17.5 | 36.4 | 7.6 | 55.0 | 46.7 |
| | o4-mini-TU | 37.5 | 53.0 | 21.7 | 53.3 | 58.0 | 34.2 | 31.7 | 55.0 | **44.0** | 13.8 | 38.9 | 11.8 | 47.5 | 52.0 |
| | **o3-TU** | **46.0** | **55.0** | 31.7 | 53.3 | **66.0** | **42.5** | **50.0** | **64.0** | 42.0 | 21.3 | **41.0** | **16.4** | 57.5 | **77.3** |

**关键结论**：
- 工具增强（Tool-Using）模式带来显著提升，o3-TU（46.0%）比 o3 无工具（26.9%）**高出近 20 个百分点**
- 绝大多数主流模型（含 GPT-4o、Qwen2.5-VL-72B）得分在 **20%~30%** 区间，说明该基准具有较高区分度
- 低光图像理解（LL-VQA）是 Gemini-2.5-Pro 的相对优势（42.0%），但仪器读数（IR）仍是普遍短板

---

## 七、置信度验证方法

在正式使用 TIR-Bench 评测自己的模型之前，**强烈建议先做置信度校验**：选择一个论文中有公开得分的参照模型（如 o3），在本地复现其评测，若最终得分与论文数据基本吻合，则说明评测环境配置正确、结果可靠。

以下是两个已验证的参照数据点：

### Qwen3.5-122B-A10B

| 任务 | 自测得分 | 技术报告得分 |
|------|---------|-------------|
| **综合（All）** | **42.24%** | **42.5%** |
| Color | 41.94% | — |
| Contrast | 42.00% | — |
| Instrument | 22.78% | — |
| Jigsaw | 15.45% | — |
| Math | 47.50% | — |
| Maze | 55.83% | — |
| OCR | 25.00% | — |
| Proportion | 56.67% | — |
| Rotation | 27.03% | — |
| Spot Difference | 34.64% | — |
| Symbolic | 48.00% | — |
| Visual Search | 73.28% | — |
| Word Search | 40.23% | — |

### o3

| 任务 | 自测得分 | 技术报告得分 |
|------|---------|-------------|
| **综合（All）** | **23.34%** | **26.9%** |
| Color | 37.23% | — |
| Contrast | 28.00% | — |
| Instrument | 16.25% | — |
| Jigsaw | 5.44% | — |
| Math | 24.17% | — |
| Maze | 26.67% | — |
| OCR | 6.67% | — |
| Proportion | 35.00% | — |
| Rotation | 34.67% | — |
| Spot Difference | 21.43% | — |
| Symbolic | 32.00% | — |
| Visual Search | 35.29% | — |
| Word Search | 1.00% | — |

> 自测与报告数值存在小幅差异属正常现象（通常在 ±3% 以内），若偏差超过 5%，建议检查图像读取、并发调用或答案提取逻辑。

---

## 八、评测结论生成 Prompt 模板

完成评测后，可使用以下 prompt 快速生成规范化结论文案。

### 版本一：简洁版（适合公众号推文、简报）

```
你担任多模态模型评测总结助手，我会给出TIR-Bench总体准确率和各任务得分。
请生成一段精简短结论：
1. 一句话定整体水平、对标基准顶尖模型位置；
2. 简明列出核心优势任务、核心短板任务；
3. 一句话概括模型能力特点：静态视觉强弱、工具图像推理强弱；
4. 文案精炼简短，不用展开细节、不用列表。
```

### 版本二：学术版（适合技术报告、论文附录）

```
你担任多模态大模型专业评测分析师，我会提供模型在TIR-Bench的总体准确率、各子任务名称、样本量、得分。
请按固定逻辑生成正式评测结论：
1. 开篇给出总体准确率，对标论文中 o3-TU、o4-mini-TU 做档位排名与水平定位；
2. 自动划分优势强项、中等水平、明显短板三类任务并罗列关键项；
3. 归纳模型核心能力特征：静态视觉理解表现、工具式图像智能体推理表现；
4. 分析短板根源，并给出后续迭代优化重点；
5. 行文学术正式、段落连贯，直接输出可落地到报告的结论文本，不口语、不额外解释、不列表格。
```

### 结论模板示例

> 本次模型在 TIR-Bench 基准上综合准确率为 **X%**，对照论文中的模型排名第 **X**，第一是 o3-TU（46.0%）。
>
> **模型静态视觉理解能力突出**，其中 Visual Search 高分辨率视觉检索（**X%**）、Maze 迷宫路径规划（**X%**）、Proportion 目标占比估算（**X%**）优势明显。
>
> **工具驱动的图像智能体推理能力偏弱**，其中 Jigsaw 拼图复原（**X%**）、Rotated OCR 旋转图像文字识别（**X%**）、Instrument 仪器读数（**X%**）、Rotation 旋转角度判别（**X%**）得分普遍偏低。

---

## 九、总结

TIR-Bench 的核心价值在于将"工具调用"纳入多模态评测体系，揭示了一个被传统基准长期忽视的能力维度：**当前主流 MLLM 在静态视觉理解上已有不错表现，但在需要主动调用工具、多步迭代推理的场景下仍有显著差距**。

这对模型开发者的启示也很明确：未来多模态模型的能力上限，很可能不在于"更大的视觉编码器"，而在于**更强的工具感知、更稳定的多轮规划与自我修正能力**。


