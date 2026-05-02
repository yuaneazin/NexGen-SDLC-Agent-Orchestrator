# Geo-Sentinel: Neuro-Symbolic SDLC Orchestrator for GIS

<div align="center">
  <p><strong>基于神经符号架构的地理空间任务自动化与逻辑审计系统</strong></p>
  <p>Bridging the gap between LLM probabilistic generation and strict physical spatial constraints.</p>
</div>

---

## 📖 1. 项目愿景与背景 (Vision & Background)

在当前的地理信息科学（GIS）与大型语言模型（LLM）交叉领域，存在一个致命的“语义-物理”断层：LLM 擅长生成代码和处理模糊逻辑，但缺乏对物理空间和拓扑常识的硬性感知（例如：不理解投影形变、无法识别异常高程）。这导致在复杂的 3S 集成实验中，传统的 Auto-Agent 极易产生 **GIGO (Garbage In, Garbage Out)** 问题。

**Geo-Sentinel** 并不是一个简单的“代码生成器助手”，而是一个基于 **SDLC (Software Development Life Cycle)** 的全链路调度器。它利用国产大模型（MiMo-v2.5-pro）作为认知核心，同时将 **ArcPy/GDAL** 等传统确定性引擎降维成“物理裁判（Domain Auditor）”，通过不断的反向询问与沙箱碰撞，实现高可靠性的地理任务自动化闭环。

---

## 🏗️ 2. 核心架构设计 (Core Architecture)

本系统由四个高度解耦的 Agent 模块协同工作，形成非线性的自愈闭环：

### Module A: 需求解构与对齐层 (Reasoning Engine)
*   **机制:** 接收用户的模糊自然语言输入，放弃线性的单步生成，采用 **ToT (Tree of Thoughts)** 进行任务树展开。
*   **反向询问 (Reverse Prompting):** 发现缺失因子时，主动向用户进行对齐（如：“发现目标区域缺少 30m 精度的 DEM 数据，是否允许调用 GEE API 进行在线裁剪补充？”）。

### Module B: 架构与生成代理 (Architecture & Generation Agent)
*   **机制:** 将任务树翻译为具体的空间计算图（Spatial Computation Graph），并调用 `MiMo-v2.5-pro` 生成底层处理脚本（Python/ArcPy）。
*   **隔离:** 代码不直接运行，而是被送入审计队列。

### Module C: 领域逻辑审计层 (Domain Auditor) —— [核心壁垒]
本系统的核心创新点，超越常规的代码语法检查，进行“物理常识约束”。
*   **投影与坐标系预审:** 强制检测生成代码中对 `SpatialReference` 的定义，防止跨带投影或非等面积投影导致计算失真。
*   **空值与边界处理 (NoData Handling):** 审查栅格代数运算前，是否包含数据清洗与空值掩膜逻辑。

### Module D: 自愈沙箱与结果评价 (Healing Sandbox & Evaluation)
*   **执行与捕获:** 在隔离环境中运行脚本，捕获底层引擎抛出的 `Stack Trace`。
*   **结果统计溯源:** 脚本成功运行后，Agent 将读取输出成果（如 `.tif`）的统计直方图（Histogram）。若发现极值异常（如土壤侵蚀模数出现负值），触发“物理法则违背”警报，强制打回 Module A 重构。

---

## 🌍 3. 实验场景落地：汶川县 RUSLE 自动化测算

本项目以经典的修正通用土壤流失方程（RUSLE）为测试用例，验证系统处理复杂科学计算的能力。

$$A = R \times K \times LS \times C \times P$$

### 3.1 传统工作流的痛点
在传统的 ArcGIS Pro 实验中，完成上述模型需要手动进行数十次数据重采样、投影转换、表面分析（提取坡度/坡长）和栅格计算器操作，极易产生误差累积。

### 3.2 Geo-Sentinel 自动化流
1.  **Input:** "请读取本地的汶川县 DEM、降雨量、土壤类型和 NDVI 影像，利用 RUSLE 公式测算水土流失分布。"
2.  **Audit Collision:** 在生成 $LS$（地形因子）提取代码时，系统审计代理检测到输入的 DEM 边缘存在无效值（NoData），主动注入了 `Focal Statistics`（焦点统计）平滑修复代码。
3.  **Healing:** 栅格计算器因分辨率不统一报错，沙箱捕获异常，并指挥大模型自动补充 `Resample`（重采样）对齐步骤。
4.  **Output:** 最终输出通过物理极值校验的侵蚀模数分布图，并生成对应的 Markdown 分析简报。

---

## ⚙️ 4. 技术栈说明 (Technology Stack)

*   **大语言模型 (LLM):** Xiaomi MiMo-v2.5-pro (推理与生成主干)
*   **地理引擎基座 (GIS Engine):** ArcPy / GDAL / Geopandas (充当物理定律校验器)
*   **开发语言:** Python 3.9+ 
*   **自动化调度:** 自研轻量级 Agent Orchestrator (基于 Pydantic 规范化数据流)

---

## 🚀 5. 快速开始 (Getting Started)
### 5.1 环境配置

```bash
conda create --name geo-sentinel --clone arcgispro-py3
conda activate geo-sentinel
pip install -r requirements.txt
```

### 5.2 环境变量设置

在根目录创建 `.env` 文件，注入你的模型额度凭证：

```env
MIMO_API_KEY=your_mimo_api_key_here
AGENT_MODE=STRICT_AUDIT
WORKSPACE_PATH=./data/wenchuan_county
```

### 5.3 运行沙箱测试

```bash
python scripts/setup_env.sh
python core/agent_manager.py --task "run_rusle_pipeline"
```

---

## 🛣️ 6. 演进路线图 (Roadmap)

*   [x] 构建基础的 Prompt-to-Code 工作流。
*   [x] 引入基于 ArcPy 的底层异常捕获与重试机制。
*   [ ] **(WIP)** 完善 `Domain Auditor` 的领域知识图谱规则库。
*   [ ] 探索基于 Geohash 的表征重构机制（对齐 ReaGeo 等前沿学术架构），提供端到端的语义空间测试平台。

---

## 🎓 7. 作者 (Author)

**Yuan Zhang**

*   **Chengdu University of Technology**, GIS Major.
*   **Focused on**: Spatial Data Logic, Neuro-symbolic GIS Architectures.
*   **Contact / Contributions**: Please submit PRs or open issues for academic discussion.
