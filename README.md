# ARGUS 漏洞公告与利用情报数据

独立于 ARGUS 应用运行的每日采集与版本发布仓库。每日北京时间 **03:23** 运行，也支持 Actions → **Sync data** → **Run workflow** 手动触发。

| 来源 | 采集方式 | 默认单轮预算 |
| --- | --- | --- |
| NVD | fkie-cad 社区 Git 镜像；2018 年起；比较 blob ID，处理变更与删除，最新 CVE 优先补采 | 10,000 文件 |
| CISA KEV | cisagov 官方 Git 镜像；完整目录关联，保留尚未收录详情的 CVE | 1 文件 |
| OSV | 官方导出桶；11 个生态、ETag 检查、持久分页续点；保留 CVE 别名 | 2,000 变更对象，40 列表页 |
| 长亭漏洞库 | 列表发现与详情查询；历史续采，四分之一预算复查到期记录 | 100 详情，7 天复查 |
| Vulners | CVE 详情 API，补充利用情况、评分与原始别名信息 | 100 CVE，7 天复查 |

API 源所需 Actions Secret：`VULNERS_API_KEY`。缺少时该源明确显示 `skipped`，不影响公开源。已配置但余额不足或接口失败时显示 `failed`，保留旧数据并让工作流报告失败。

初次导入是分批过程，`partial` 表示还在补采；可手动多跑几轮。源记录独立保留，不跨源覆盖原始字段。NVD 镜像并非 NIST 官方仓库。

## 数据与使用

Releases 提供 `manifest.json`、`records.jsonl.gz`、`delta.jsonl.gz`、`state.sqlite.gz`，以及存在上游许可文件时的 `resources.tar.gz`。清单包含来源状态、上游版本、记录数与每个资产的 SHA-256。消费者按来源记录 ID / CVE / aliases 关联数据。

先校验清单中的资产大小和 SHA-256。消费者导入完整 `records.jsonl.gz`；仅在本地版本等于 `base_version` 时应用增量。`state.sqlite.gz` 用于采集器恢复，不能丢弃其中的源游标。失败的下载或校验不得当作首次启动。

当前 ARGUS 应用还没有接入这些发布包；本仓库不更改应用的采集开关、数据结构或激活流程。

## 本地验证

需要 Python 3.12、Git；发布与恢复还需已认证的 GitHub CLI。

```sh
python -m pip install -r requirements.txt
python -m unittest discover -s tests -v
python scripts/sync.py --source kev --limit 10 --repository OWNER/argus-intel-data
```

本地数据位于忽略的 `.work/` 与 `dist/`。仓库初始为私有，数据遵循各自上游条款。不要把 API Key 写入配置、日志或数据包。

完整设计及已知边界见 [architecture.md](docs/architecture.md)。
