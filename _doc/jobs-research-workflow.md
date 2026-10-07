# 就业调研页（/jobs/）更新工作流

> 本文件给 AI 助手（Claude Code 等）阅读执行用。目标：把 `/jobs/` 页面的就业数据更新到最新。
> 本文件位于 `_doc/`（下划线目录），**不会被 Jekyll 发布到线上**，可以放心写内部细节。

## 一、产物结构

| 文件 | 作用 | 更新时是否改动 |
|---|---|---|
| `_data/jobs.yml` | **全部调研数据**（岗位、公司、来源） | ✅ 唯一需要改的文件 |
| `jobs.html` | 页面渲染逻辑（Liquid 模板，读取 yml 循环渲染） | 一般不动；只有结构变化（加字段、加分区）才改 |
| 线上地址 | https://yhwu0423.github.io/jobs/ | push 后 GitHub Pages 自动构建 |

## 二、`_data/jobs.yml` 数据结构

```yaml
updated: "2026-10"            # 调研时间（年月），显示在页面顶部——每次更新必改
audience: "2027 届校招（2026 年秋招季）"   # 面向人群——跨校招季时改
categories: [...]             # 8 个大类，卡片按此分组渲染；job.category 必须是其中之一
jobs:                         # 细分岗位列表（当前 21 个）
  - id: "java-backend"        # 锚点 slug（英文 kebab-case），总览表跳转用，新增岗位必须给唯一 id
    name: "Java 后端开发"
    category: "后端开发"       # 必须与 categories 中某项完全一致
    salary: "白菜 30–40 万 · SP 40–50 万"    # 年包档位
    salary_note: "..."        # 薪资出处/口径（必写，注明来源与届别）
    headcount: "大 —— ..."    # 需求量（总览表只显示空格前第一个词）
    popularity: "高 —— ..."
    difficulty: 4             # 1-5 整数（渲染星级；不能是小数）
    value: 4.5                # 性价比 1-5，可 .5；总览表按它降序排
    tech_stack: [...]         # 技术栈标签
    learning: [...]           # 要学什么（列表）
    notes: "..."              # 大厂 JD 关键要求与备注（要求引用 JD 原文名词）
    jds:                      # 代表性官方 JD / 官方渠道（2-5 条）
      - company: "得物"
        position: "Java 开发工程师"
        url: "https://campus.dewu.com"
companies:                    # 大厂全景表（当前 54 家）
  - name: "字节跳动"
    point: "一句话校招看点"
    site: "https://jobs.bytedance.com/campus"   # 必须经 curl 实测可访问
sources: [...]                # 数据来源（名称 + URL）
```

## 三、更新触发

- 秋招季（每年 8–10 月）、春招季（3–4 月）各值得更新一次；
- 或用户直接说「更新就业调研页」。

## 四、执行步骤

### Step 0 ｜ 确认时间锚点
先跑 `date` 拿真实当前日期（不要凭印象），据此判断当前是哪个校招批次（如 2026 年秋招 = 2027 届），相应调整 `updated` 与 `audience`。

### Step 1 ｜ 逐家调研公司校招动态
- 工具：**WebSearch**。**一次只发一个查询，严禁并发**（用户 API 套餐有并发限制，并发会 403）。
- 查询词模板：`{公司名} {届别}届校园招聘 启动 技术岗位`；可 2-4 家同梯队公司合并为一个查询。
- **查询词里不要带 URL**（带网址的查询会返回 "No links found"）。
- 信源优先级：官方公告 > 权威媒体（新华网 / 界面 / 财联社 / IT之家）> 高校就业网转载 > 牛客/爆料平台。返回结果里 "No links found" 后面模型自己编的内容**不可信**，必须重搜换关键词。

### Step 2 ｜ 官方入口核验（必做）
WebFetch 对大量域名被拦（报 "Unable to verify if domain is safe"），**改用本机 curl 实测**：

```bash
curl -s -o /dev/null -w '%{http_code}' --noproxy '*' --max-time 8 -A 'Mozilla/5.0' '<url>'
```

- 只收录返回 **2xx/3xx** 的入口；404/000 的换备用域名重试（常见 ATS 域名模式见下）；都失败则该公司不收录，或链接降级为公司官网首页并在看点中注明。
- SPA 页面 curl 拿到空壳 HTML 是正常的，**只看状态码**，不要试图解析内容。
- 常见 ATS 域名模式（猜入口时用）：
  - 飞书招聘：`{公司拼音}.jobs.feishu.cn/index` 或 `/campus`
  - 自建校招站：`campus.{公司域名}`、`hr.{公司域名}`、`join.{公司域名}`、`careers.{公司域名}`
  - 北森：`{公司}.zhiye.com/campus`；Moka：`app.mokahr.com/campus_apply/{公司}`

### Step 3 ｜ 岗位与 JD 调研
- 每个细分岗位至少配 **2-4 条代表性官方 JD / 官方渠道**。
- `notes` 里的技术要求要**摘自 JD 原文**（框架名、技术名词），禁止凭印象写。
- 岗位是否需要新增/合并/删除，以当期各厂官方在招职位为准（例如 2027 届新出现了「AI 全栈工程师」「Agent 开发工程师」「AI SRE」）。

### Step 4 ｜ 薪资数据
- 来源优先级：官方报告（前程无忧 / 智联 / 脉脉）> Offershow / 牛客爆料 > 媒体整理。
- **必须标注口径与届别**（写进 `salary_note`）；找不到可靠数据就写「暂无公开数据」，**禁止编造**。
- 关键事实需 ≥2 个来源一致才写（见用户记忆 blog-writing-rigor）。

### Step 5 ｜ 写入 yml
- 保持 schema 不变；新增公司追加到 `companies`；新增岗位给唯一 `id` 并归入某个 `category`。
- 各处的公司/岗位数量统计（yml 注释、jobs.html 标题与说明框、sources 最后一条）要同步改。

### Step 6 ｜ 本地验证

```bash
bundle exec jekyll build
grep -c 'job-card"' _site/jobs/index.html    # 应等于 jobs 数量
grep -c '官方入口' _site/jobs/index.html      # 应等于 companies 数量
grep -o '{%' _site/jobs/index.html           # 应无输出（无 Liquid 残留）
```

### Step 7 ｜ 提交发布

```bash
git add _data/jobs.yml jobs.html
git commit -m "add 更新就业调研数据（YYYY-MM）"   # 提交风格：add + 中文简述，带 Co-Authored-By trailer
git push 2>&1 || git -c http.proxy= -c https.proxy= push   # 代理失效时的回退
```

- 完成后提醒用户 **Ctrl+Shift+R 强刷**（sw.js 缓存，普通刷新可能看到旧版）。

## 五、质量红线（不可违反）

1. 官方 JD / 官网优先，第三方盘点仅作补充；
2. 所有链接必须 curl 实测可访问；
3. 数据必须带出处（写进 `sources` 或 `salary_note`）；
4. 搜不到就写「暂无公开数据」，禁止编造；
5. 页面是给公众看的博客，**不要**在页面上写内部工作流、文件路径、「叫博主更新」这类元信息——那些只写在本文件里。
