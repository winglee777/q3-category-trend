# {{节令×品类}} 爆品趋势报告

> 视频号小店 {{年份}} {{节日}}周期（{{开始日期}} — {{结束日期}}）{{行业}}赛道爆品洞察。
> 基于 `品类趋势解读` skill v2 生成。

---

## 项目信息

- **取数周期**：{{开始日期}} — {{结束日期}}（{{周数}} 周节日周期）
- **数据来源**：视频号小店全域 TOP 商品（友望导出）
- **品类范围**：{{N}} 大品类（{{品类列表}}）
- **owner**：{{owner}}

---

## 目录结构（v2）

```
.
├── README.md                  本文件
├── source.xlsx                原始数据（不入 git）
│
├── 【数据加工层】
├── gen.py                     步骤 1：xlsx → records.json + stats.json
├── category-rules.json        品类清洗规则（继承自 template，1100+ 关键词起步）
├── cross_aggregate.py         步骤 3：基于 attrs 算交叉表（口味×品类、场景×品类等）
├── verify_attrs.py            质量校验：检查属性抽取覆盖率
├── insights_brief.py          数据简报：把所有聚合表浓缩成 LLM 可读输入
│
├── 【数据产物层】data/
│   ├── records.json           商品级明细（含 cat + attrs 11 维度）
│   ├── stats.json             品类基础聚合（SKU/销量/GMV）
│   ├── flavor_stats.json      🆕 口味×品类交叉
│   ├── scene_stats.json       🆕 场景×品类交叉
│   ├── ingredient_stats.json  🆕 原料×品类交叉
│   ├── brand_stats.json       🆕 品牌集中度
│   ├── selling_point_stats.json 🆕 卖点频次
│   ├── insights_brief.md      🆕 LLM 输入简报
│   ├── insights.json          各品类洞察文案（LLM 草稿 + 人工审核）
│   └── overall.json           5 大整体趋势（LLM 草稿 + 人工审核）
│
└── 【可视化层】
    ├── index.html              主报告（多品类切换 + TOP 排行）
    └── onepage.html            一页速览版（趋势 + CTA）
```

---

## 完整工作流（6 步）

### Step 1 · 数据加工（脚本，<10 秒）

```bash
cp /path/to/导出.xlsx ./source.xlsx
python3 gen.py
```

输出 `data/records.json` + `data/stats.json`，规则覆盖率应 ≥ 95%。

### Step 2 · 品类规则补丁（Claude 重判，~5 分钟）

如果 origin 残留 > 5%，调用 Claude：

> "读 data/records.json 里 cat_reason='origin' 且 GMV>=1000 的记录，按节令专属词补充 category-rules.json 的关键词，目标覆盖率 99%+"

完成后再跑一次 `python3 gen.py`。

### Step 3 · 11 维度属性抽取（Claude 内嵌 LLM，~30 分钟）

调用 Claude，按批处理（每批 100 条）抽取以下 11 维度：

```
brand          品牌
product        产品（具体品名/品类细分）
flavor[]       口味
ingredient[]   原料
spec           规格
scene[]        场景
audience       人群
selling_point[]卖点
promo          促销钩子
timing         时令
sub_category   子类目
```

完成后跑校验：

```bash
python3 verify_attrs.py
```

### Step 4 · 交叉聚合（脚本，<10 秒）

```bash
python3 cross_aggregate.py
```

生成 5 张交叉表（flavor/scene/ingredient/brand/selling_point × 品类）。

### Step 5 · 洞察文案生成（Claude，~10 分钟）

```bash
# 先生成数据简报，节省 LLM token
python3 insights_brief.py
```

调用 Claude：

> "读 data/insights_brief.md，写出 overall.json（5 大整体趋势）和 insights.json（各品类子赛道 chips + ADQ 关键词），先出草稿我审核"

### Step 6 · 部署（git push，<2 分钟）

```bash
git init && git add .
git commit -m "init: {{节令}}趋势报告"
git remote add origin git@github.com:USER/REPO.git
git push -u origin main
# 在仓库 Settings → Pages 启用部署
```

---

## 数据访问统计

`index.html` 和 `onepage.html` 的 `</body>` 前加：

```html
<script async src="//busuanzi.ibruce.info/busuanzi/2.3/busuanzi.pure.mini.js"></script>
<div style="text-align:center;color:#7A8C7F;font-size:12px;padding:20px">
  本页累计访问 <span id="busuanzi_value_site_pv">…</span> 次 ·
  独立访客 <span id="busuanzi_value_site_uv">…</span> 人
</div>
```

---

## LLM 双轨策略

| 路径 | 适用场景 | 推荐 |
|---|---|---|
| **路径 B：CodeBuddy 内嵌 Claude** | 日常协作、质量优先 | ✅ 主推 |
| **路径 A：DeepSeek API（llm_classify.py）** | 无人值守、批量化、跨设备 | 备用 |

详见 `技能库/品类趋势解读/SKILL.md` 第七节。

---

## 与 template 的差异

每次新建项目时，从 template 复制完整骨架，只需在本文件修改：

- {{节令}} / {{年份}} / {{开始日期}} / {{结束日期}}
- {{行业}} / {{品类列表}}
- {{owner}}
- 主题色（参考 SKILL.md 第八节的节令配色库）

---

## 已知坑 & 预防

参见 `SKILL.md` 第十节"已知坑清单"（5 大类，已验证）。

---

> 全量数据 / 报告定制 / 跨节令对比，请联系 **{{owner}}**
