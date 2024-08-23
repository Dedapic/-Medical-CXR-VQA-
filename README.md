# 对Medical-CXR-VQA 数据集的调研内容

### 1）获取该数据集的相关地址：
- https://github.com/Holipori/Medical-CXR-VQA
（Medical-CXR-VQA数据集正在Physionet中进行审查中，目前在Physionet上还没能查询到。）

### 2）数据集规模、内容、特点：
- **规模、内容：**
The dataset contains 780,014 question–answer pairs, which they categorized into six types: abnormality (190,525 pairs), location (104,680 pairs) ， type (69,486 pairs), level (111,715 pairs), view (92,048 pairs), and presence (211,560 pairs).
- **特点：**
  - 提供了详细的临床信息，如异常情况、位置、严重程度和类型等。
  - 问题和答案的设计模拟了实际的医学诊断流程。

###  3）数据集的构建：
从源数据库 MIMIC-CXR 的自由文本报告中构建了 Medical-CXR-VQA 数据集。【 MIMIC-CXR数据集——①包含 227,835 个研究和 377,110 张图像的大规模数据集；②下载地址：https://physionet.org/content/mimic-cxr/2.0.0/ 】
- **step1：**
  - 概述：为了从 MIMIC-CXR 数据集中提供更深入的信息，使用 LLMs 创建了一个中间 KeyInfo 数据集（the KeyInfo dataset contains the key information of each report, such as abnormalities, and their corresponding locations, types, and levels），其中包含更细致的信息。
  - 当中的细节（KeyInfo extraction）：
  ①结构化的关键信息对于从原始 MIMIC-CXR 数据集构建 VQA 数据集至关重要。为了收集此类信息，需要收集异常名称及其相应的属性关键词。然而，基于规则的方法存在局限性，因此尝试利用LLM从医学报告中提取关键信息，这不仅能够简化流程、节省时间，还能增强提取信息的多样性，以适应各种场景。
  ②有两种方法可以指导大型语言模型（LLM）执行特定任务。第一种方法是使用提示，第二种是LLM微调。然而，第一种方法由于 LLM 的长度限制，生成的输出可能没有足够的空间，且使用高性能的 LLM（如 GPT-4）对于大规模应用来说成本过高。因此，LLM微调变得尤为重要，但LLM 微调依赖于高质量的数据集作为训练数据。所以作者决定在应用中结合这两种方法——先利用GPT-4生成训练数据。然后使用这些数据来微调 Llama 2。
- **step2：**
  从 KeyInfo 数据集中提取问题-答案对，以构建 Medical-CXR-VQA 数据集。为了满足放射科医生在疾病诊断方面的兴趣，问题设计是在VQA-RAD 问题设计上做扩展，构建完 KeyInfo 数据集后能够获取生成六种类型问题所需的所有信息。
- **step3（后处理）：**
  - 发现幻觉是微调后不可避免的问题。
  - 因此，使用 follow-up questions 来增强 LLM 的原始生成输出。
  - 接着，应用后处理代码以进一步标准化格式，这包括统一复制名称、拆分带有异常名称的属性、移除不需要的发现以及重新分配属性。
- **step4（评估）：**
  Conducted a comparison of the correctness rates between 100 random samples extracted from the LLM method and the rule-based method.
 
### 4）Multi-modal relationship graph
在Medical-CXR-VQA数据集中起到了桥接图像特征、医学知识和问答对之间联系的作用，为构建一个可解释、多模态的医学VQA模型提供了基础。
- **三种不同的关系来构建多模态关系图:**
  - 1）基于 ROI-wise 空间位置的空间关系图。【根据之前的研究（Li et al., 2019）定义空间关系】
  - 2）基于医学专家知识的语义关系图。语义关系是基于解剖结构和疾病的知识图谱构建的。 作者在语义关系图中定义了两种类型的语义关系：①解剖知识图【遵循之前的研究（Zhang et al., 2020），构建了一个解剖知识图谱，用于模拟身体部位及其相应的疾病关系】，②共现知识图【通过计算和归一化 MIMIC-CXR-JPG 数据集中不同疾病标签的出现频率来提取共现关系】。
  - 3）隐含关系以发现额外的潜在关系图。【利用已被证明在一般领域视觉问答问题中有效的隐式关系来发现潜在关系 (Li et al., 2019)。遵循 Li et al.（2019）的设计，使用完全连接的图来学习图顶点之间的隐式关系。】

### 5）贡献和创新点：
- 通过应用LLMs，增强了从医学报告中提炼关键信息的精确度，相比于传统的基于规则的方法，实现了62%的准确度提升。
- 数据集融合了多模态关系图学习方法，为医学图像VQA任务提供了丰富的上下文信息。
