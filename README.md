# Hermes Skills Export / Hermes 技能导出

This repository contains exported snapshots of two Hermes Agent skills:

- xianyu-price-research
- official-document-format-refresh

Exported from local machine: `~/.hermes/skills/hermes-created/`.

Purpose: keep a portable snapshot of these skills for versioning and sharing.

---

## Skills / 技能说明

### xianyu-price-research

English:
- Description: Scrapes real Xianyu (闲鱼) search results using a local Chrome login session, exports CSV/HTML, filters invalid samples, and produces pricing suggestions based on real marketplace samples. Useful for second-hand hardware, computer parts, mini PCs, SSDs, and memory pricing reviews and weekly retrospectives.
- Typical usage scenarios: weekly price review, determine listing price / quick-sell price / probe price for a given model, validate price bands across multiple keywords. Outputs: CSV/HTML in runtime/ and reports in reports/.

中文：
- 描述：使用本机 Chrome 登录态抓取闲鱼真实搜索结果，导出 CSV/HTML，过滤无效样本，并基于真实样本输出定价建议。适用于二手硬件、电脑配件、迷你主机、SSD、内存等商品的价格回顾与每周复盘。
- 使用场景：每周价格回顾、为某款商品确定挂牌价/快速出手价/试探价，根据多个关键词验证价格区间。输出文件位于 runtime/（原始 CSV/HTML）和 reports/（定价报告）。

---

### official-document-format-refresh

English:
- Description: Refreshes legacy official documents (.doc/.docx) to comply with modern Chinese governmental document formatting standards (post-2012), focusing strictly on formatting changes without altering original text. Produces a new .docx and a text-diff report confirming no content edits.
- Typical usage scenarios: convert legacy policy drafts or reports into standard official-document layout; prepare editable .docx for official submission while preserving original content.

中文：
- 描述：将已有机关公文（.doc/.docx）按 2012 年起执行的机关公文格式规范进行格式刷新，强调仅更新格式、不改正文内容。产出新的 .docx 文件，并生成文本差异比对报告以确认未改正文。
- 使用场景：旧公文整理为规范格式、将非规范草稿转为可提交的机关公文版式、保留原文不改内容的格式化输出。

---

README updated on: $(date -u +"%Y-%m-%dT%H:%M:%SZ") (UTC)
Contact: ymzhang10
