# LangExtract ✕ DeepSeek：高精度结构化信息提取完全指南

本教程专为希望使用 **DeepSeek** 作为底层模型，结合 **LangExtract** 进行高精度信息提取的开发者编写。

LangExtract 并非简单的 LLM 套壳调用，而是一个**系统级的信息提取工程框架**。它内置了 Prompt 编排、多轮召回、智能分块和字符级溯源等机制，解决了传统 LLM 提取中常见的“幻觉”、“漏提”和“格式错误”问题。

---

## 一、 快速开始与基础配置

DeepSeek 的 API 完全兼容 OpenAI 格式。我们推荐直接复用内置的 `OpenAILanguageModel` Provider。

### 1. 安装依赖

请确保安装时带上 openai 相关的依赖包：

```bash
pip install "langextract[openai]"
```

### 2. 配置 DeepSeek 客户端

```python
import os
import langextract as lx

# 配置你的 DeepSeek API Key
os.environ["OPENAI_API_KEY"] = "sk-your-deepseek-api-key"

# 初始化 DeepSeek 模型配置
# 推荐使用 deepseek-chat (V3) 进行常规提取，或 deepseek-coder 获取更强的逻辑推理能力
config = lx.factory.ModelConfig(
    model_id="deepseek-chat",
    provider="OpenAILanguageModel",
    provider_kwargs={
        "base_url": "https://api.deepseek.com/v1",
        "temperature": 0.0, # 提取任务建议设置为 0，保证结果稳定性
    }
)
```

---

## 二、 核心功能与代码实战

以下示例展示了 LangExtract 的各项杀手锏功能，并附带针对 DeepSeek 的最佳实践配置。

### 核心功能 1：声明式 Few-shot 引导 (免写复杂 Prompt)

你不需要手写冗长的正则表达式或复杂的 Prompt 模板。只需用自然语言描述目标，并提供 2-3 个标准样例，LangExtract 会自动在底层将其编排为极具引导性的 Prompt。

```python
# 1. 准备你要提取的原始文本
source_text = """
2023年10月15日，张三（Zhang San）与北京字节跳动科技有限公司签订了一份价值500万人民币的技术服务合同。
项目负责人是李四，预计在2024年底前交付。
"""

# 2. 声明式 Few-shot：告诉模型“你想要什么格式的数据”
# 提供几个标注好的样板（字典或 Pydantic 模型均可）
examples = [
    {
        "text": "马化腾创立了腾讯控股有限公司，总部位于深圳。",
        "entities": [
            {"name": "马化腾", "type": "人物"},
            {"name": "腾讯控股有限公司", "type": "公司"}
        ]
    },
    {
        "text": "史蒂夫·乔布斯（Steve Jobs）是苹果公司（Apple Inc.）的联合创始人。",
        "entities": [
            {"name": "史蒂夫·乔布斯", "type": "人物"},
            {"name": "苹果公司", "type": "公司"}
        ]
    }
]

# 3. 执行提取
result = lx.extract(
    text_or_documents=source_text,
    prompt_description="提取文本中出现的所有【公司】和【人物】实体。",
    examples=examples,
    config=config,
    # ⚠️ DeepSeek（OpenAI 兼容系）必填参数：
    fence_output=True,            # 开启 Markdown 代码块包裹，稳定解析 JSON
    use_schema_constraints=False  # 关闭原生 Schema 强约束（目前仅 Gemini 等原生支持）
)

print(result.extracted_data)
# 预期输出: 
# [
#   {"name": "张三", "type": "人物"}, 
#   {"name": "北京字节跳动科技有限公司", "type": "公司"},
#   {"name": "李四", "type": "人物"}
# ]
```

---

### 核心功能 2：字符级溯源（Character-level Grounding）

**这是 LangExtract 的最大杀手锏。**
每一个提取出的结果，不仅是一段文本，而是精确映射回原文的 **字符偏移量 (offset)**。这使得结果可以被程序**反向验证**，彻底拒绝 LLM 的“幻觉”。

```python
# 接上文的代码
for entity in result.grounded_entities:
    # 打印实体及其在原文中的精确起止位置
    print(f"提取实体: {entity.text}")
    print(f"原文位置: 字符 {entity.start_char} 到 {entity.end_char}")
    
    # 你甚至可以反向从原文本切片来验证提取结果是否属实：
    original_slice = source_text[entity.start_char : entity.end_char]
    print(f"原文反向切片验证: '{original_slice}'")
    print("-" * 20)

# 输出示例：
# 提取实体: 北京字节跳动科技有限公司
# 原文位置: 字符 26 到 38
# 原文反向切片验证: '北京字节跳动科技有限公司'
# --------------------
```

*优势：如果模型凭空捏造（幻觉）了一个不存在的公司名，系统将无法将其映射到原文本的 offset，从而被底层直接拦截或打上非溯源标记。*

---

### 核心功能 3：长文档“大海捞针”（智能分块 + 多轮提取）

当你需要处理长达几万字的 PDF 解析文本或研报时，单次喂给大模型极易遗漏信息（Lost in the Middle）。LangExtract 提供了自动化解决方案：

1. **智能分块 (Chunking with Overlap)**：自动对超长文本切片，并保留切片间的重叠区间，防止关键信息刚好卡在边界被截断。
2. **多轮提取 (Multi-Pass)**：对同一个切片进行多轮“复审”，每次提示模型寻找上一轮遗漏的信息，最后自动去重合并，极大提升**召回率（Recall）**。

下面是一个从长篇财务报告中提取“风险提示”的**完整、可运行代码示例**：

```python
import os
import langextract as lx

# (假设 config 已经配置好，参考第一节)

# 1. 模拟一篇很长很长的文档（例如：年度财务报告）
# 这里我们通过字符串乘法，故意将关键信息分散在很长的文本中，模拟“大海捞针”场景
long_financial_report = """
【2023年度财务报告及管理层分析】
第一节：公司业务概况。本年度公司在新能源汽车赛道持续发力，实现了营收的稳健增长...（此处省略一万字套话）...
风险提示一：市场竞争加剧的风险。近年来，行业内新进入者不断增加，如果公司不能在技术创新、产品质量上保持优势，可能面临市场份额下降的风险。
第二节：核心财务指标分析。本年度实现净利润 5.6 亿元，同比增长 12%...（此处省略一万字财务数据分析）...
风险提示二：原材料价格波动风险。公司主要原材料为锂矿石等大宗商品，受国际宏观经济影响较大。若未来价格大幅上涨，将对公司毛利率产生直接的不利影响。
第三节：未来展望与战略规划。公司计划在欧洲市场增设三个研发中心...（此处省略一万字战略规划）...
特别风险说明：核心技术人员流失风险。公司所处行业属于技术密集型，核心技术人员对研发至关重要。若发生大规模核心人才被竞对挖角的现象，将严重阻碍新一代产品开发。
""" * 5 # 乘以5，模拟成一篇非常长的文档，确保触发底层分块机制

# 2. 准备 Few-shot 样例，规范输出结构
examples_for_risks = [
    {
        "text": "本公司面临汇率波动风险。由于公司大量业务集中在海外，人民币升值将直接导致汇兑损失，影响当期净利润。",
        "entities": [
            {
                "risk_name": "汇率波动风险", 
                "description": "由于海外业务集中，人民币升值会导致汇兑损失。"
            }
        ]
    }
]

print("开始处理长文档，启动智能分块与多轮提取引擎...")

# 3. 执行长文档深度提取
result = lx.extract(
    text_or_documents=long_financial_report,
    prompt_description="提取文档中明确列出的所有【风险提示】条款，必须包含风险名称和具体描述。",
    examples=examples_for_risks,
    config=config,
    
    # ----------------------------------------------------
    # 🔥 长文档“大海捞针”核心优化配置 🔥
    # ----------------------------------------------------
    chunk_size=1500,      
    # 【智能分块】：将长文本按 1500 字符为单位切块。
    # 根据 DeepSeek 的上下文能力，实际生产中可设为 3000-8000。
    
    chunk_overlap=200,    
    # 【重叠区间】：相邻块之间保留 200 字符的重叠。
    # 核心作用：防止关键句子刚好被切到两个不同的 chunk 中导致信息丢失。
    
    max_passes=3,         
    # 【多轮召回】：对切分好的每个 chunk，让模型“审阅” 3 次！
    # 核心作用：第一遍模型可能只提取了最明显的风险；第二遍框架会提示模型“仔细看看还有没有漏掉的”；
    # 第三遍继续深挖。随后框架会在底层将 3 遍的结果去重合并。这是提升长文本 Recall 召回率的绝对关键。
    # ----------------------------------------------------
    
    fence_output=True,
    use_schema_constraints=False
)

# 4. 打印最终合并与去重后的结果
print(f"\\n提取完成！跨越 {len(long_financial_report)} 字符的长文档，共提取到独立风险因素：")
for i, item in enumerate(result.extracted_data, 1):
    risk_name = item.get("risk_name", "未命名风险")
    desc = item.get("description", "无描述")
    print(f"【风险 {i}】: {risk_name}")
    print(f" 描述: {desc}\\n")
```

---

## 三、 高阶建议与架构选型

### 1. 为什么不用自定义 Provider？

如果你只是日常的信息提取，直接修改 `base_url` 复用 `OpenAILanguageModel` 已经可以覆盖 **95% 的场景**。
除非你需要：

- 植入企业内部特殊的流量网关认证逻辑
- 实现特殊的重试/熔断机制 (Retries/Circuit Breaker)

*此时，你可以通过 `@registry.register` 注册一个继承 `BaseLanguageModel` 的自定义类并重写 `infer` 方法。*

### 2. DeepSeek 参数调优指南

- **Temperature**：强制设为 `0.0`，降低生成多样性，确保相同的文本每次提取的结果一致。
- **并发与缓存**：LangExtract 默认开启并发请求。因为 DeepSeek V3/R1 响应极快，处理几百页的长文档时，框架会自动切块并并发发送，同时内置的缓存能避免相同的 Chunk 被重复扣费。

### 3. Schema 约束说明

由于 OpenAI 兼容接口（目前包含 DeepSeek）的原生 `json_schema` 强制约束在某些复杂嵌套场景下，表现可能不如纯文本 Prompt 稳定。因此，使用 **`fence_output=True` 搭配 `use_schema_constraints=False` 是 DeepSeek 的黄金组合**。框架会要求模型输出 Markdown 包裹的 JSON 代码块，并在本地以高容错的方式完成解析。