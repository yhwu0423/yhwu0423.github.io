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
companies:                    # 大厂全景表（当前 61 家）
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
- **301/302 必须用 `-L` 跟踪落点确认**：落点可能是错的页面——实测案例：`career.honor.com` 裸域名 302 到**社招**页（校招要换成 `…/pb/school.html` 链接）；`moonshot.cn/careers` 302 回首页（说明招聘版块已撤，链接降级为官网首页）；`campus.iflytek.com` 301 到 `iflytek.zhiye.com`。
- **域名会失效**：实测案例 `offershow.net` NXDOMAIN（域名已弃用），改用 `offershow.cn`。老数据里的链接每次更新都要重新全量 curl。
- **Moka 有两个陷阱**（2026-10 实测）：
  1. `app.mokahr.com/campus_apply/{公司名}` 对**不存在的租户也返回 200**，但 `<title>` 是「您访问的页面不存在」——Moka 链接必须额外抓 `<title>` 验证，只看状态码会收进假链接；
  2. `campus_apply/{公司}/{数字id}` 形式的链接无 Cookie 时 302 循环——Moka 深链一律不收录，改用该公司官网/其他 ATS。
- 常见 ATS 域名模式（猜入口时用）：
  - 飞书招聘：`{公司拼音}.jobs.feishu.cn/index` 或 `/campus`
  - 自建校招站：`campus.{公司域名}`、`hr.{公司域名}`、`join.{公司域名}`、`careers.{公司域名}`，另有 `horizon-campus.hotjob.cn`（地平线，用友大易 ATS）这类独立校招域名
  - 北森：`{公司}.zhiye.com/campus`；Moka：`app.mokahr.com/campus_apply/{公司}`（仅作线索，收录前查 title）
- **官网是 SPA 找不到投递入口时**：curl 抓其 `main.*.js`  bundle，`grep -oE 'https://[^"]*(mokahr|feishu|zhiye)[^"]*'` 能挖出隐藏的 ATS 地址（DeepSeek 的 Moka 租户 `high-flyer/140576` 就是这样挖到的）。

### Step 3 ｜ 岗位与 JD 调研
- 每个细分岗位至少配 **2-4 条代表性 JD**；`jds` 里**优先放具体职位详情页深链**，实在没有深链的标「官网在招」指向官方渠道（仅放招聘首页是被用户批评过的做法，避免）。
- 已验证可用的深链模式：
  - 飞书招聘：`{公司拼音}.jobs.feishu.cn/campus/position/{职位ID}/detail`（影石、小鹏、Momenta、沐瞳、面壁等实测有效；`/campus/m/position/...` 移动版亦可）
  - 字节：`jobs.bytedance.com/campus/m/position/detail/{职位ID}`
  - B站：`jobs.bilibili.com/campus/positions/{职位ID}`
  - 阿里：`campus-talent.alibaba.com/campus/position/{职位ID}`
  - 荣耀：`career.honor.com/SU{hash}/pb/school.html?postTypeCode=...`（搜索引擎索引的标题会带具体职位名；**裸 career.honor.com 是社招页，勿用**）
  - OPPO：`careers.oppo.com/university/oppo/campus/post?shareId={id}`（官方分享的岗位列表）
  - 讯飞：`iflytek.zhiye.com/4/jobs?shareId=...`（「飞星计划」2027 届官方分享职位列表；`campus.iflytek.com` 已 301 到 zhiye）
  - 大疆：`careers.dji.com/zh-CN/campus/hot-jobs`（2027 校招热招职位页；其 Moka 链接 302 循环不可用）
  - **牛客职位详情页：`nowcoder.com/jobs/detail/{id}`——SSR 页面，可 curl 抓出 JD 全文验证公司归属与内容**（各厂官方发布的校招职位会同步在这里）
- **牛客 JD 必查三项**（2026-10 教训）：curl 抓正文确认 ① 届别是 2027 届（挖到过「理想+」大模型岗实为 2025 届旧帖）；② 页面上的「已结束」标记——已截止的 JD 可以收录（JD 内容仍是真实官方文本），但 position 名称必须标注「JD 存档，已截止」；③ 公司名与岗位名和 yml 里写的一致。
- 牛客的**搜索页 / 企业主页是 JS 渲染**，curl 拿不到职位 ID；职位 ID 只能靠搜索引擎自然结果的标题/链接捞（查询词带「职位详情」+ 公司 + 岗位名）。
- **第三方聚合站（鼠鼠求职、bebee、watchjobs、领英、猎聘、面灵AI 等）一律不作 jds 链接**，只作为发现官方深链的线索；高校就业网（`career.*.edu.cn` 等）同理——可作公告佐证写进笔记，不作 jd 链接。
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

随后**全量链接审计**（每次更新必做，域名会失效、职位会截止）：

```bash
grep -oE 'https://[^"'"'"' ]+' _data/jobs.yml | sort -u | while read -r u; do
  echo "$(curl -s -o /dev/null -w '%{http_code}' --noproxy '*' --max-time 10 -A 'Mozilla/5.0' "$u')  $u"
done    # 000/404 必须处理；301/302 抽查 -L 落点；mokahr 链接加查 <title>
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
