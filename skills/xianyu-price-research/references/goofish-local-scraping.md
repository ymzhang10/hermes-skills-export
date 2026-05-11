---
name: goofish-local-scraping
description: Use local macOS Chrome profile copy + Selenium to scrape Goofish/闲鱼 search results when cloud/browser tools trigger "非法访问".
---

# Goofish Local Scraping

当需要抓取闲鱼真实搜索结果，而 Hermes 的云端 browser 工具打开 `goofish.com` 只显示“非法访问”时，使用这个方案。

## 触发条件

- 需要闲鱼真实在售样本、价格、标题、链接
- `browser_navigate("https://www.goofish.com")` 能打开，但 `document.body.innerText` 显示：
  - `非法访问`
  - `为了保障您的体验，请使用正常浏览器访问闲鱼`
- 用户在 macOS 本机使用 Chrome，且已登录闲鱼/淘宝体系账号

## 核心思路

不要直接复用正在运行的 Chrome 用户目录；要**复制一份本机 Chrome 用户目录到临时目录**，再让 Selenium 使用复制件启动本地 Chrome。

这样可以：
1. 继承真实登录态
2. 避开云端 browser 的风控环境
3. 避免直接占用用户正在使用的 Chrome profile
4. 保留 HTML/CSV 便于后续分析

## 前提检查

先确认本机环境：

```bash
python3 --version
node --version
npm --version
bun --version || true
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --version
```

确认 Chrome 用户目录存在：

```bash
python3 - <<'PY'
import os
for p in [
    os.path.expanduser('~/Library/Application Support/Google/Chrome'),
    os.path.expanduser('~/Library/Application Support/Google/Chrome/Default'),
]:
    print(('EXISTS' if os.path.exists(p) else 'MISS'), p)
PY
```

## 推荐目录

```text
/Users/ymzhang/Obsidian-Local/ymzhang/hermes-backup/闲鱼抓取/
```

仓库可放：

```text
/Users/ymzhang/Obsidian-Local/ymzhang/hermes-backup/闲鱼抓取/GooFishCrawlers/
```

运行时输出建议放：

```text
/Users/ymzhang/Obsidian-Local/ymzhang/hermes-backup/闲鱼抓取/runtime/
```

## 可用仓库

优先尝试：
- `https://github.com/msyloveldx/GooFishCrawlers`

克隆：

```bash
mkdir -p "/Users/ymzhang/Obsidian-Local/ymzhang/hermes-backup/闲鱼抓取"
cd "/Users/ymzhang/Obsidian-Local/ymzhang/hermes-backup/闲鱼抓取"
git clone https://github.com/msyloveldx/GooFishCrawlers.git
```

## 安装依赖

```bash
python3 -m venv "/Users/ymzhang/Obsidian-Local/ymzhang/hermes-backup/闲鱼抓取/GooFishCrawlers/.venv"
source "/Users/ymzhang/Obsidian-Local/ymzhang/hermes-backup/闲鱼抓取/GooFishCrawlers/.venv/bin/activate"
pip install -r "/Users/ymzhang/Obsidian-Local/ymzhang/hermes-backup/闲鱼抓取/GooFishCrawlers/requirements.txt"
```

## 关键实现：复制本机 Chrome 登录态

新建脚本，例如 `hermes_local_profile_test.py`：

```python
import shutil
import time
from pathlib import Path
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.common.by import By

SRC_USER_DATA = Path.home() / "Library/Application Support/Google/Chrome"
WORK_ROOT = Path.home() / "Obsidian-Local/ymzhang/hermes-backup/闲鱼抓取/runtime"
PROFILE_COPY = WORK_ROOT / "chrome-profile-copy"


def copy_profile():
    WORK_ROOT.mkdir(parents=True, exist_ok=True)
    if PROFILE_COPY.exists():
        shutil.rmtree(PROFILE_COPY)
    ignore_names = shutil.ignore_patterns(
        "Cache", "Code Cache", "GPUCache", "GrShaderCache", "ShaderCache",
        "DawnCache", "BrowserMetrics", "Crashpad", "Safe Browsing*",
        "OptimizationHints", "Subresource Filter", "FileTypePolicies",
        "Singleton*", "RunningChromeVersion"
    )
    shutil.copytree(SRC_USER_DATA, PROFILE_COPY, ignore=ignore_names)
    return PROFILE_COPY


def build_driver(profile_dir: Path):
    options = Options()
    options.add_experimental_option("excludeSwitches", ["enable-automation"])
    options.add_experimental_option("useAutomationExtension", False)
    options.add_argument("--disable-blink-features=AutomationControlled")
    options.add_argument(f"--user-data-dir={profile_dir}")
    options.add_argument("--profile-directory=Default")
    options.add_argument("--window-size=1400,1000")
    options.add_argument("--lang=zh-CN")
    options.binary_location = "/Applications/Google Chrome.app/Contents/MacOS/Google Chrome"
    driver = webdriver.Chrome(options=options)
    driver.execute_cdp_cmd("Page.addScriptToEvaluateOnNewDocument", {
        "source": """
            Object.defineProperty(navigator, 'webdriver', {get: () => undefined});
        """
    })
    return driver


profile_dir = copy_profile()
driver = build_driver(profile_dir)
try:
    driver.get("https://www.goofish.com/")
    time.sleep(8)
    print(driver.find_element(By.TAG_NAME, "body").text[:1000])
finally:
    time.sleep(5)
    driver.quit()
```

## 成功判定

如果成功，页面正文通常不再是“非法访问”，而会出现：
- 搜索
- 订单
- 发闲置
- 各分类
- 猜你喜欢

这说明本机 Chrome 登录态复制方案可用。

## 导出搜索结果

新建脚本 `goofish_local_search_export.py`，思路如下：

1. 使用复制后的 profile 启动本地 Chrome
2. 打开 `https://www.goofish.com/search?q=<关键词>`
3. 滚动若干轮，让列表懒加载
4. 保存 `page_source`
5. 用 BeautifulSoup 从 HTML 中提取 `a[href*="item?id="]`
6. 解析标题、首个价格、想要人数、位置、链接
7. 输出 CSV 和 HTML 归档

最重要的一点：
**不要依赖 Selenium `find_elements('a[href*=item?id]')` 直接取结果。**
在闲鱼页面里，Selenium 取到 0 条的情况很常见；但 `driver.page_source` 里实际已经有完整 `<a href="...item?id=...">` 节点。此时应改用 BeautifulSoup 解析 HTML。

## 建议输出字段

- `title`
- `price_first`
- `all_prices`
- `want_count`
- `location`
- `href`
- `raw_text`

## 建议输出文件

```text
/Users/ymzhang/Obsidian-Local/ymzhang/hermes-backup/闲鱼抓取/runtime/goofish-<keyword>-<timestamp>.csv
/Users/ymzhang/Obsidian-Local/ymzhang/hermes-backup/闲鱼抓取/runtime/goofish-<keyword>-<timestamp>.html
```

## 已验证有效的关键词

- `NUC11PAHi7`
- 也可继续试：
  - `猎豹峡谷 i7-1165G7`
  - `NUC11PAH i7`
  - `M471A4G43MB1-CTD`
  - `970 EVO Plus 500G`
  - `970 EVO Plus 1T`

## 已验证结果

使用本方案时，已经成功抓到：
- 闲鱼真实 item 链接
- 标题
- 价格
- 想要人数
- 你自己的 NUC11PAHi7 挂单

## 常见坑

### 1. 直接用 Hermes browser 工具
现象：
- `browser_navigate("https://www.goofish.com")`
- `browser_console(expression="document.body.innerText")`
返回“非法访问”

处理：
- 改用本地 Selenium + Chrome profile copy

### 2. 直接复用正在运行的 Chrome 用户目录
风险：
- 容易被锁
- 容易碰到 `SingletonSocket` / `SingletonLock` / `RunningChromeVersion` 相关问题

处理：
- 复制 profile
- 忽略：
  - `Singleton*`
  - `RunningChromeVersion`
  - 各类缓存目录

### 3. Selenium 能看到页面，但取不到商品 anchor
现象：
- 页面文字正常
- `find_elements` 返回 0

处理：
- 保存 `driver.page_source`
- 用 BeautifulSoup 解析 HTML

### 4. 闲鱼结果里会混入 i5 / 商家 / 全新 / 引流项
处理：
- 后续分析时必须二次清洗
- 不能直接把搜索页全部结果当成可比样本

## 推荐工作流

1. 先抓一个关键词并导出 CSV
2. 目检 CSV 前 10~20 条是否真实
3. 再批量抓多个关键词
4. 用 Python/表格做清洗：
   - 排除 i5 / i3
   - 排除全新商家机
   - 排除非准系统 / 非同代 / 非同 CPU
5. 最终只保留真正可比样本做估价

## 适用范围

适用于：
- 闲鱼价格调查
- 二手样本收集
- 指定型号市场摸底

不适用于：
- 大规模高频抓取
- 依赖稳定 API 的生产系统
- 长期无人值守监控（除非后续继续加重试、代理、登录态续期）
