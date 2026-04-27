---
name: sci-psychiatry-digest
description: 收集 SCI 一区（JCR Q1）心理学/精神病学领域的最新高影响因子文献
version: "2.5"
author: Judy (朱迪)
license: MIT
---

# SCI 一区心理精神科文献速递 Skill (v2.5)

## ⚠️ 飞书文档权限处理（必读！）

创建文档时必须确保用户（曹俊，open_id: `ou_13198dff86df39f443084c8dc460d89f`）能够编辑，**二选一**：

### 方案一：以用户身份创建（推荐）

```bash
lark-cli docs +create \
  --api-version v2 \
  --title "文档标题" \
  --doc-format markdown \
  --as user \
  --content "$(cat content.md)"
```

**`--as user`** 标志让文档以用户身份创建，用户直接获得所有者权限，可正常编辑和分享。

### 方案二：机器人身份创建后授予编辑权限

```bash
# Step 1: 创建文档（机器人身份）
lark-cli docs +create \
  --api-version v2 \
  --title "文档标题" \
  --doc-format markdown \
  --content "$(cat content.md)"

# Step 2: 授予用户编辑权限
lark-cli docs +update-perm \
  --doc DOC_ID \
  --actor ou_13198dff86df39f443084c8dc460d89f \
  --perm full_access
```

> 🚫 **禁止**：仅用机器人身份创建文档而不授予权限——用户只能查看，不能编辑。
> ✅ **正确做法**：始终使用 `--as user` 或创建后立即授予 `full_access` 权限。

---

## 目标
为用户一次性产出一份结构化、可复制的 **SCI 一区（JCR Q1 / 中科院 1 区）心理学/精神病学** 高影响因子文献清单，每篇文章包含：
- 基本信息（标题 / 期刊 / IF / 发表时间 / 链接）
- 中文简介

> ### 🚫 绝对禁止事项
> 1. **禁止虚假信息**：不得编造文章标题、作者、期刊名、IF、分区、链接或内容简介。
> 2. **禁止无效链接**：所有链接在输出前必须验证，200 OK 方可写入文档。失效链接（404 / Embargo 无法替换为 DOI 时）必须删除。
> 3. **禁止内容冲突或无关**：写入的汇总信息必须与检索到的原始内容一致；如发现期刊名、发表时间、研究结论等与原文不符，必须修正或删除该条目。

> ⚠️ **核心原则**：宁缺毋滥。如无法确认真实性，宁可删除也不能写入虚假或冲突内容。

---

## 触发条件
当用户表达以下意图时启用：
- "给我 SCI 一区心理精神科的文章"
- "心理学 / 精神病学 Q1 / 一区 期刊最新论文"
- "高影响因子 精神病学 / 心理学 文献速递"
- "中科院一区心理精神科文献"

---

## 执行流程

### Step 1 · 明确用户偏好（若未给出）

| 参数 | 默认值 | 可选值 |
|------|--------|--------|
| 细分方向 | 综合（不限） | 抑郁 / 双相 / 精神分裂 / 儿童青少年 / 神经影像 / 心理治疗 / 成瘾 / 创伤 / 焦虑 / 认知障碍 |
| 分区体系 | 双体系（均收录） | 仅 JCR Q1 / 仅中科院 1 区 / 双体系均收录 |
| 时间窗 | 近 1 年 | 近 3 个月 / 近半年 / 近 1 年 / 近 2 年 |
| 每模块数量 | 15 篇 | 每模块 10 / 15 / 20 篇 |

---

### Step 2 · 检索顶刊列表

| 期刊 | IF (2024 JCR) | JCR 分区 | 中科院分区 |
|------|--------------|---------|-----------|
| World Psychiatry | ~73 | Q1 | 1区 Top |
| The Lancet Psychiatry | ~30 | Q1 | 1区 |
| JAMA Psychiatry | ~25 | Q1 | 1区 |
| American Journal of Psychiatry | ~14 | Q1 | 1区 |
| Molecular Psychiatry | ~10 | Q1 | 2区参考 |
| Neuropsychopharmacology | ~7 | Q1 | 1区 |
| Psychological Medicine | ~7 | Q1 | 1区 |
| Biological Psychiatry | ~9 | Q1 | 1区 |
| Translational Psychiatry | ~6.8 | Q1 | 2区参考 |
| Schizophrenia Bulletin | ~5 | Q1 | Q1 |

---

### Step 3 · EUtils API 检索命令模板

```bash
# 检索某期刊近1年文章
TERM="JOURNAL_NAME[journal]%20AND%202025[pdat]"
curl -s "https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esearch.fcgi?db=pmc&term=$TERM&retmax=20&retmode=json"

# 获取文章摘要
curl -s "https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi?db=pmc&id=PMCID&retmode=text&rettype=medline"
```

---

### Step 4 · 采集文章信息

| 字段 | 要求 |
|------|------|
| 标题 | 英文原题，不得简写 |
| 期刊 | 官方刊名 + 缩写 |
| IF | 2024 JCR 版本，注明"IF ~xx" |
| 分区 | JCR Q1 + 中科院分区（如有） |
| 发表时间 | 期刊官方出版日期（年/月） |
| 源链接 | PMC 链接 或 DOI，格式必须可点击 |
| 中文简介 | 含研究设计、样本、主要发现、意义，100-180 字 |

---

### Step 5 · ⚠️ 验证所有链接

```bash
for url in "PMC_URL_1" "DOI_URL"; do
  code=$(curl -s -o /dev/null -w "%{http_code}" -L --max-time 10 "$url")
  echo "$code $url"
done
```

| HTTP 状态码 | 处理方式 |
|-------------|---------|
| 200 OK | ✅ 直接使用 |
| 403 Forbidden | ⚠️ PMC Embargo，必须替换为 DOI |
| 404 Not Found | ❌ 删除或标注"需手动访问" |

---

### Step 6 · 双输出（飞书文档 + 直接回复）

#### 6A · 飞书文档格式

**标题**：`SCI 一区心理精神科文献速递 · {YYYY-MM-DD}`

**文档结构**：模块一（中科院 1 区）+ 模块二（JCR Q1）+ 附录（链接验证结果 + 期刊 IF 参考）

**⚠️ 重要**：创建时必须使用 `--as user` 参数，或创建后授予用户 `full_access` 权限（见上文「飞书文档权限处理」章节）。

#### 6B · 直接回复

每篇文章：标题 / 发表时间 / 期刊 / IF / 链接 / 中文简介

---

## 质量要求

1. **IF 版本一致**：同一次输出使用同一 JCR 年度，并在附录注明
2. **链接 100% 验证**：不合格链接不得写入正文
3. **简介与原文一致**：发现冲突必须修正或删除
4. **分区标注清晰**：JCR Q1 + 中科院分区，两个体系独立成模块
5. **双输出强制**：必须同时执行飞书文档创建 + 直接回复
6. **权限必处理**：文档创建时必须使用 `--as user` 或创建后授予权限

---

## 技术备注

- EUtils esearch：`https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esearch.fcgi?db=pmc&term=TERM&retmax=20&retmode=json`
- EUtils efetch：`https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi?db=pmc&id={PMCID}&retmode=text&rettype=medline`
- PubMed 链接：`https://pubmed.ncbi.nlm.nih.gov/{PMID}/`
- 链接验证：`curl -s -o /dev/null -w "%{http_code}" -L --max-time 10`
- 用户 open_id：`ou_13198dff86df39f443084c8dc460d89f`（曹俊，固定值）
