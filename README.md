# Hermes Skills Export / Hermes 技能导出

This repository contains exported snapshots of two Hermes Agent skills:

- xianyu-price-research
- official-document-format-refresh

Exported from local machine: `~/.hermes/skills/hermes-created/`.

Purpose: keep a portable snapshot of these skills for versioning and sharing.

---

本仓库包含从本地 Hermes Agent 导出的两个技能快照：

- xianyu-price-research（闲鱼比价）
- official-document-format-refresh（公文格式刷新）

来源路径：`~/.hermes/skills/hermes-created/`。

用途：保留可移植的技能快照，便于版本管理与共享。

README updated on: $(date -u +"%Y-%m-%dT%H:%M:%SZ") (UTC)

Contact: ymzhang10

## Skills / 

### xianyu-price-research

English:
- Description: Scrapes real Xianyu () search results using a local Chrome login session, exports CSV/HTML, filters invalid samples, and produces pricing suggestions based on real marketplace samples. Useful for second-hand hardware, computer parts, mini PCs, SSDs, and memory pricing reviews and weekly retrospectives.
- Typical usage scenarios: weekly price review, determine listing price / quick-sell price / probe price for a given model, validate price bands across multiple keywords. Outputs: CSV/HTML in runtime/ and reports in reports/.


