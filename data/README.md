# data 目录说明

本目录是昇腾算力专区（AI 算力专区）的结构化数据，按语言分 `zh/`、`en/` 两个目录，文件名、主键、行数一一对应。

- 字段统一用 **camelCase**
- 每个文件是一个 JSON 数组（`manifest.json` 是对象）
- 与 `数据库与接口设计-2.0.md` 的差异，见仓库根目录的 `接口口径变更说明.md`

## 一、基础枚举

| 文件 | 行数 | 主键 | 说明 |
|---|---:|---|---|
| `frameworks.json` | 4 | `id` | `vllm` / `mindie` / `mindspeed` / `mindspore`（MindSpore 数据源是 MindFormers 文档） |
| `quant_levels.json` | 5 | `id` | `bf16` / `w8a8` / `w4a8` / `w8a16` / `mxfp8` |

`mxfp8` 是 OCP 微缩放格式（元素 FP8 E4M3，每 32 元素共享 E8M0 缩放），950 系列官方权重用它，常见组合是 MXFP8 激活 + MXFP4 权重（即 W4A8），**不等同于普通 FP8**。

## 二、算力卡

| 文件 | 行数 | 主键 | 说明 |
|---|---:|---|---|
| `ai_cards.json` | 7 | `id` | 卡型号；`primaryOs` 是该卡 guide 绑定的主 OS，`recordedOn` 本轮为空 |
| `ai_card_os.json` | 18 | `(cardId, osVersion)` | 卡已认证的 OS |
| `ai_card_products.json` | 7 | `cardId` | 卡所属产品线 |
| `card_model_supports.json` | 860 | `(cardId, modelId)` | 卡 × 模型支持关系 |
| `card_frameworks.json` | 28 | `(cardId, frameworkId)` | 卡上有数据的框架 |
| `card_framework_os.json` | 15 | `(cardId, frameworkId, osVersion)` | 每组合只写 1 行（guide 绑定的主 OS） |

## 三、模型

| 文件 | 行数 | 主键 | 说明 |
|---|---:|---|---|
| `model_series.json` | 161 | `id` | 模型系列 |
| `models.json` | 223 | `id` | 模型；`id` 为 slug（如 `qwen3-8b`） |

## 四、命令与安装部署指导

| 文件 | 行数 | 主键 | 说明 |
|---|---:|---|---|
| `infer_commands.json` | 212 | 自增 | 推理命令（短文本） |
| `train_commands.json` | 728 | 自增 | 训练命令（短文本） |
| `infer_deploy_commands.json` | 448 | `(modelId, cardId, quantId, mode, scene)` | 推理安装部署指导 |
| `train_deploy_commands.json` | 2912 | `(modelId, cardId, mode, scene)` | 训练安装部署指导（训练无量化维度） |

### `guideMd` 的约定

固定四节顺序：

```
## 前提条件   3–5 条要点，一句话一条
## 安装       关键步骤 + 命令
## 部署       命令（训练另附启动脚本）
## 验证       可缺；训练通常没有
```

- 命令**逐字来自官网原文**，不做改写、不补参数
- 原文没有某节时，该节直接不出现（不写「暂无」「未提供」）
- 不出现「见第 X 节」「见上文」这类引用——需要的内容直接贴过来

### 空 `guideMd` 的含义

`guideMd` 为空字符串**不是抓取失败**，表示「官方文档没有该卡 × 该量化 × 该安装方式 × 该场景的来源」。

这类记录额外带一个 `guideEmptyReason` 字段说明原因，例如：

- 该卡在该量化下没有官方权重
- 源码安装小节只有文字、没有可引用的代码块
- 文档只提供另一种部署场景
- 特性矩阵不支持该卡

前端建议显示「暂无指导」，不要显示空白区块。

### 计数口径

- 一条记录只有在 `guideMd` 非空时才计入「部署套数」
- `osOptions` 只收「至少有一条非空 guideMd」的 OS
- 推理 448 行中，本轮 174 条有指导、274 条为空

## 五、场景镜像（Ascend Hub）

| 文件 | 行数 | 主键 | 说明 |
|---|---:|---|---|
| `images.json` | 32 | `id` | 镜像仓库 |
| `image_tags.json` | 561 | `(imageId, tag)` | 镜像 tag，只保留 openEuler |
| `card_images.json` | 106 | `(cardId, imageId)` | 镜像适用于哪些卡 |
| `image_variants.json` | 66 | `(imageId, os, arch)` | 变体 |
| `image_build_steps.json` | 158 | `(imageId, os, arch)` | 构建步骤（仓库级，复制到各 variant） |
| `image_verify_steps.json` | 12 | `(imageId, os, arch)` | 验证步骤，只有少数仓库有 |
| `image_dockerfiles.json` | 52 | `(imageId, tag)` | Dockerfile 原文 |
| `image_env_vars.json` | 6 | 自增 | 环境变量说明 |
| `image_build_params.json` | 52 | 自增 | 构建参数 |
| `image_layers.json` | 2805 | 自增 | 镜像层 |

### `image_dockerfiles` 的来源字段

| 字段 | 说明 |
|---|---|
| `source` | 原始来源（官网页面 URL） |
| `sourceUrl` | Dockerfile 文件地址 |
| `sourceRepo` | 仓库，如 `Ascend/cann-container-image` |
| `sourceKind` | `cann-baseline`（官方 CANN 基线，4 条）或 `product`（业务仓库自建镜像，48 条） |

这 52 条**不是同一镜像的多个版本**：4 条来自官方 CANN 基线镜像仓库（每个芯片 1 条，选完芯片 + OS 唯一），其余 48 条来自 mind-cluster、DrivingSDK、IndexSDK 等业务仓库。

选片逻辑**保持原样**（卡 + OS + tag），新增字段只用于追溯来源，不参与筛选。

## 六、驱动

| 文件 | 行数 | 主键 | 说明 |
|---|---:|---|---|
| `driver_versions.json` | 51 | `id` | Ascend HDK 解决方案版本，状态只有 `certified` |
| `driver_version_os.json` | 111 | `(driverVersionId, osVersion)` | 版本与 OS 的对应 |

## 七、其他

| 文件 | 说明 |
|---|---|
| `manifest.json` | 版本号、生成时间、各表行数、数据来源 |
| `QA_REPORT.md` | 数据质量自检结果 |
| `ai_card_planned_os.json` | 本轮无数据（兼容性查询助手没有规划状态） |
| （未生成）`roadmap_items` | 适配进展不在本轮范围 |

## 八、已知限制

1. 卡的认证 OS 与镜像 tag 只保留 openEuler，接口本身仍接受任意 OS
2. `recordedOn` / `certifiedOn` 全为空——官网没有认证日期，不拿快照日期冒充
3. MindIE 没有英文版，逐模型 README 为机器翻译（代码块原样保留）
4. 训练指导的内容粒度到「框架 × 卡 × 安装方式 × 场景」，同一组合下不同模型的指导相同（官方文档只到框架级）
