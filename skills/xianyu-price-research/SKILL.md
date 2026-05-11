---
name: xianyu-price-research
description: 使用本机 Chrome 登录态抓取闲鱼真实搜索结果，导出 CSV/HTML，过滤无效样本，并输出基于真实闲鱼样本的定价建议。适用于二手硬件、电脑配件、迷你主机、SSD、内存等商品的价格回顾与每周复盘。
---

# Xianyu Price Research

适用场景：
- 用户要查询闲鱼真实价格，而不是外围搜索结果
- 用户要给二手商品定价、改价、做周度回顾
- 用户要基于“真实闲鱼样本”给出建议挂牌价、底价、试探价

不适用：
- 只想看京东/淘宝新品价
- 没有本机 Chrome 登录态
- 需要抓取私信、详情页深度字段、成交价（闲鱼通常只能拿到在售价）

## 固定目录

### 工具目录
- 脚本仓库：`/Users/ymzhang/Obsidian-Local/ymzhang/hermes-backup/闲鱼抓取/GooFishCrawlers/`
- Python venv：`/Users/ymzhang/Obsidian-Local/ymzhang/hermes-backup/闲鱼抓取/GooFishCrawlers/.venv/`

### 输出目录
- 周报输出根目录：`/Users/ymzhang/Obsidian-Local/ymzhang/hermes-backup/闲鱼抓取/`
- 原始抓取目录：`/Users/ymzhang/Obsidian-Local/ymzhang/hermes-backup/闲鱼抓取/runtime/`
- 价格报告建议放在：`/Users/ymzhang/Obsidian-Local/ymzhang/hermes-backup/闲鱼抓取/reports/`

## 已验证可用的本机脚本

### 1. 本机 Chrome 登录态测试
`hermes_local_profile_test.py`

用途：
- 使用托管的 Chrome profile 启动本地 Selenium + Chrome
- 验证是否能正常打开闲鱼而不是触发“非法访问”
- 给用户足够时间扫码，并在登录后默认不自动关闭窗口

关键经验：
- 不能每次都删除并重建 profile 副本，否则用户刚扫码保存的登录态会在下一轮被自己清掉
- 现在应默认复用托管 profile：`/Users/ymzhang/Obsidian-Local/ymzhang/hermes-backup/闲鱼抓取/runtime/chrome-profile-copy`
- 登录测试脚本默认不自动关闭；用户扫完码、点完“保持登录”后可手动关窗
- 如确实需要从系统 Chrome 重新复制一份干净副本，显式设置环境变量：`HERMES_REFRESH_PROFILE_COPY=1`

### 2. 闲鱼关键词导出脚本
`goofish_local_search_export.py`

用途：
- 按关键词打开闲鱼搜索页
- 自动滚动加载结果
- 导出真实搜索结果到 CSV / HTML
- 输出字段：`title`, `price_first`, `all_prices`, `want_count`, `location`, `href`, `raw_text`
- 支持 `--batch`，在同一个 Chrome 会话里连续跑多个关键词
- 支持 `--batch-file <关键词文件路径> [滚动轮数]`，从一行一个关键词的文本文件读取关键词；空行和 `#` 开头注释会被忽略

关键经验：
- 不要依赖 `page_source + BeautifulSoup` 去抓商品链接；实测会出现浏览器里已渲染出商品卡片，但 `page_source` 中仍抓不到 `item?id=`，最终导出空 CSV
- 应优先从实时 DOM 抓取：直接用 Selenium `execute_script` 读取 `document.querySelectorAll('a[href*="item?id="]')`
- 搜索页第一次打开时可能只出现“加载中...”骨架页；同一会话里热身回首页后重试，往往第二次即可拿到真实商品卡片
- 因此脚本要内置：同会话重试、热身页、骨架页识别，而不是遇到 0 行就直接判失败

运行示例：

```bash
source "/Users/ymzhang/Obsidian-Local/ymzhang/hermes-backup/闲鱼抓取/GooFishCrawlers/.venv/bin/activate"
cd "/Users/ymzhang/Obsidian-Local/ymzhang/hermes-backup/闲鱼抓取/GooFishCrawlers"
python goofish_local_search_export.py "NUC11PAHi7" 10
```

## 为什么必须用这套方法

闲鱼云端浏览器通常会返回：
- “非法访问”

而本机可用方法是：
- 复制 `~/Library/Application Support/Google/Chrome` 到临时目录
- 用本地 Selenium 调起 Chrome 二次打开闲鱼
- 这样可以复用本机登录态并避开大部分云端检测

## 标准流程

### 第 1 步：确认脚本和环境可用
先检查：

```bash
python3 --version
node --version
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --version
```

然后：

```bash
source "/Users/ymzhang/Obsidian-Local/ymzhang/hermes-backup/闲鱼抓取/GooFishCrawlers/.venv/bin/activate"
cd "/Users/ymzhang/Obsidian-Local/ymzhang/hermes-backup/闲鱼抓取/GooFishCrawlers"
python hermes_local_profile_test.py
```

如果看到闲鱼首页内容而不是“非法访问”，说明环境正常。

### 第 2 步：确定关键词组
原则：
- 同一商品至少抓 2~6 组关键词
- 包括精确型号 + 通俗叫法 + 关键配置

示例：
- 主机：`NUC11PAHi7` / `NUC11PAH i7` / `猎豹峡谷 i7-1165G7`
- 内存：`M471A4G43MB1-CTD` / `三星 32G DDR4 2666 笔记本内存`
- SSD：`970 EVO Plus 500G` / `970 EVO Plus 1T`

### 第 3 步：批量抓取
对每组关键词执行：

```bash
python goofish_local_search_export.py "关键词" 10
```

说明：
- 第二个参数 `10` 表示滚动轮数
- 输出会落到 `runtime/` 下，包含 CSV 和 HTML
- 真正稳定的做法不是“每个关键词都新开一次浏览器”，而是：一次准备托管 profile + 一次启动 Chrome，在同一个 driver 里连续抓完整组关键词
- 如果首轮搜索页只出现骨架（`加载中...`、`1/1`、没有商品卡片），不要立刻判定无结果；应在同一会话里先回到首页热身，再重试同一关键词 1~3 次
- 判断是否抓到结果，优先看运行中 DOM 的 `a[href*="item?id="]`，不要只依赖最终 `page_source`/BeautifulSoup；实测会出现“页面上明明已有商品卡片，但 `page_source` 里没有对应商品链接”的情况，导致 CSV 被错误导出为空

### 第 4 步：清洗样本
必须过滤：
- 回收贴
- 芯片贴
- 不同容量
- 不同型号
- 混卖贴
- 商家全新库存（若用户要的是个人二手参考）
- 明显错误标题或无关结果

保留时要区分：
1. 准系统 / 裸机
2. 带内存/带硬盘整机
3. 全新商家价
4. 个人二手价
5. 批量卖家清仓价

> ⚠️ **准系统 vs 整机是最常见的致命分类错误**：惠普商用机（ProDesk 系列）闲鱼上大量商家挂的是"准系统"（仅机箱+主板+电源+散热，无 CPU/内存/硬盘），与带完整配件的整机价格差距可达 3~5 倍。例如 HP 600G4 MT 准系统 ¥230~¥350，但 i7-8700 整机要 ¥900~¥1200。如果把准系统价当整机价输出，定价会严重低估。
>
> **识别准系统的关键词**（正文/标题中出现任意一个即应标记为准系统）：
> - 明确声明："准系统"、"无CPU"、"无内存"、"无硬盘"、"裸机"
> - 描述结构："机箱+主板+电源"、"机箱、主板、电源"、"不含CPU内存硬盘"
> - 仅配件："只有主板"、"单主板"、"拆机件"
> - 故障/零件："点不亮"、"回收"、"前面板"
>
> **整机判断标准**（同时满足以下三项才算整机）：
> - 有 CPU 型号（如 i5-6500、i7-8700）
> - 有内存容量（如 8G、16G DDR4）
> - 有硬盘（如 256G 固态、1T 机械）
>
> 仅有其中一两项的，归入"部分配置/不确定"类，不可混入整机定价参考。

### 第 5 步：输出定价
必须给三档：
- 快出价
- 正常挂牌价
- 试探价

如果是多个套餐，还要分别给：
- 裸机价
- 加配版本价
- 打包价

## 推荐分析方法

### 对主机类商品
拆成：
- 主机本体价值
- 内存打包加成
- SSD 打包加成

注意：
- 配件单卖价值不能 1:1 全额加回整机
- 整机会有 bundle 折价

### 对 SSD / 内存类商品
优先按：
- 型号完全匹配
- 容量一致
- 健康度 / 通电时间 / 写入量接近
- 成色和是否企业盘/拆机盘/全新库存

## 输出格式要求

最终报告至少包含：
1. 调查时间
2. 使用的关键词
3. 原始 CSV/HTML 文件路径
4. 有效样本摘要
5. 过滤逻辑
6. 建议挂牌价 / 底价 / 试探价
7. 一句话结论

## 周报任务的推荐做法

如果是每周回顾：
- 固定关键词列表
- 推荐将关键词维护在独立文本文件中，例如：`/Users/ymzhang/Obsidian-Local/ymzhang/hermes-backup/闲鱼抓取/xianyu-weekly-keywords.txt`
  - 一行一个关键词
  - 空行和 `#` 开头的注释行会被忽略
  - 更换关键词时优先修改该文件，不要反复改 cron prompt 里的长命令
- 每周抓一轮
- 生成日期版报告到 `reports/`
- 报告中对比“本周 vs 上周”主要价格变化
- 如果运行在 Hermes cron 中，主抓取流程不要用 `background=true` 后台化
- 也不要自己写 shell 循环逐个关键词重启 Chrome
- 正确做法是：直接用 `goofish_local_search_export.py --batch ...`，或优先用 `goofish_local_search_export.py --batch-file /path/to/keywords.txt 10` 从关键词文件读取，在同一个 Chrome 会话里跑完整组关键词，再继续清洗和写报告
- 周报完成前必须确认 12 组 CSV/HTML 是否齐全，并把报告真正落到 `reports/`，不能只留下原始抓取文件

推荐文件名：
- `xianyu-price-review-YYYY-MM-DD.md`

## 注意事项

- Hermes terminal 的 `workdir` 参数可能拒绝包含中文字符的路径（例如 `闲鱼抓取`），报错类似 `Blocked: workdir contains disallowed character '闲'`。遇到时不要改目录结构；改用安全的英文/根目录作为 `workdir`，在 shell 命令内部用 `cd ".../闲鱼抓取/..."` 进入目标目录。

1. 闲鱼拿到的是“在售价”，不是实际成交价。
2. 想要人数可做热度参考，但不能当成交依据。
3. 全新商家价和个人二手价必须分开写。
4. 如果样本太少，不要硬下结论，要明确说“样本不足”。
5. 结果必须优先依据闲鱼实抓样本，外围搜索只能作补充，不能反客为主。

## 失败特征与周报降级策略

### 常见失败特征：搜索页只加载骨架，不出商品卡片

有一种失败不是“非法访问”，而是：
- 搜索页标题正常（例如 `970 EVO Plus 500G_闲鱼`）
- 页面正文只有搜索框、筛选条、`加载中...`
- HTML 里几乎没有 `item?id=` 商品链接
- 导出的 CSV 成功生成，但 `rows=0`
- 首页登录态测试依然正常，说明不是 Chrome/登录态完全失效，而是搜索结果 DOM 没有真正渲染出来

### 遇到这种情况时要额外做的验证

除了 `hermes_local_profile_test.py` 外，再做一次单关键词验证，至少检查：
- `driver.title`
- `driver.current_url`
- `body` 文本是否只剩筛选项和 `加载中...`
- HTML 中是否存在 `item?id=`
- 导出的 CSV 是否为 0 行

如果以上同时成立，应判断为：
- 本地实抓落盘成功
- 但本轮商品结果采集失败
- 不能把 0 行 CSV 当作市场真的没货

### 周报写法要求（当本周 0 样本时）

必须在报告里明确写：
- 本周使用的仍是本机 Chrome 登录态 + 本地 Selenium
- 原始 CSV/HTML 已生成，但本周有效样本数为 0
- 本周结论应写为“样本不足”
- 不要硬写“涨了 / 跌了 / 稳了”
- 如果有上一轮有效实抓，明确标注“暂沿用上周有效实抓建议”

适合直接写进结论的话术：
- `本周抓取成功落盘，但价格样本采集失败。`
- `当前更像是搜索页只加载出页面骨架，商品卡片未进入最终 DOM。`
- `因此本周不建议贸然改价，先沿用上一轮有效实抓建议。`

## 当前已验证成功的案例

- NUC11PAHi7 / NUC11PAH i7 / 猎豹峡谷 i7-1165G7
- 三星 M471A4G43MB1-CTD
- 三星 970 EVO Plus 500G / 1T
- Intel 企业级 SSD：S3700 800G / S3500 800G / S3520 240G
- HP ProDesk 600G4 MT（i7-8700）/ 600G3 SFF（i5-6500）— 准系统 vs 整机分类教训详见 `references/hp-prodesk-quasi-vs-complete.md`
