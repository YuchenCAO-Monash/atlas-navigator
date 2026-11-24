# 本地 DeepSeek RAG 防御实验

这是一个使用虚构商城资料的本地客服原型。它通过 Ollama 调用 DeepSeek-R1-Distill-Qwen-7B，并用 ATLAS Navigator 记录防御控制及验证证据。模型从已授权的候选条目中选择来源，应用核对来源后展示审核过的原文。

## 信任边界

| 环节 | 原型控制 |
| --- | --- |
| 文档接入 | 加载固定语料，核对审核版本的 SHA-256。 |
| 检索 | 先按角色过滤，再做关键词匹配，最多返回三个候选。 |
| 模型 | 只向本机 Ollama 发送授权候选，模型只返回来源编号。 |
| 输出 | 按候选范围校验来源，用纯文本显示获准摘录或统一回退提示。 |

[完整威胁模型](rag-threat-model.html)列出 8 项 ATLAS 技术、已有控制及待补控制。实验角色由命令行指定，用于模拟文档授权。待补控制包括可信身份、跨租户隔离验证、资料来源审批和入库敏感信息扫描。

## 运行与复测

使用 Node.js 16.20.2 或更新版本。启动 Ollama，并确认具体模型在本地。如果尚未下载，运行 `ollama pull deepseek-r1:7b`。

在仓库根目录执行：

```powershell
ollama list
node .\rag-lab\rag-assistant.cjs check
node .\rag-lab\rag-assistant.cjs ask public "内部工单升级组是什么？"
node .\rag-lab\rag-assistant.cjs ask staff "内部工单升级组是什么？"
```

普通用户应收到统一回退提示，并显示「模型调用 否」。员工请求可返回 `STAFF-ESCALATION` 的虚构工单资料。离线检查的部分用例用固定模拟响应检查应用逻辑。

完整复测会调用本机模型并刷新报告。可以先备份本次检出的报告：

```powershell
$stamp = Get-Date -Format 'yyyyMMdd-HHmmss'
Copy-Item .\rag-lab\rag-test-report.html (Join-Path $env:TEMP "rag-test-report-$stamp.html")
node .\rag-lab\rag-assistant.cjs check --live
Invoke-Item .\rag-lab\rag-test-report.html
```

## 查看 ATLAS 图层

从仓库根目录进入 `nav-app`，使用项目已有的 Node 16 环境启动 Navigator：

```powershell
Set-Location .\nav-app
npm.cmd install --legacy-peer-deps
npm.cmd run ng -- serve --host 127.0.0.1
```

打开 [本地 Navigator](http://localhost:4200)，在新标签页选择 **Open Existing Layer → Upload from local**，导入 [实测威胁模型图层](atlas-rag-threat-model-tested.json)。在 `AML.T0057` 的备注可以看到 C02 的权限验证结果。

## 实测记录

- [初测报告](rag-test-report-baseline.html)为 **7/8**。M01 未选出预期来源。
- [完整复测报告](rag-test-report.html)为 **8/8**。C02 验证候选集为空且模型调用为 0，M01 和 M02 分别选出普通用户和员工的预期来源。

报告的结论适用于该语料版本、模型和列出的虚构用例。数据来自 [MITRE ATLAS Navigator Data](https://github.com/mitre-atlas/atlas-navigator-data/blob/main/dist/stix-atlas.json) 的 ATLAS 5.6.0 STIX 快照。Copyright 2021-2022 MITRE，依据 [Apache 2.0](https://github.com/mitre-atlas/atlas-navigator-data/blob/main/LICENSE) 提供。项目原许可见 [LICENSE.txt](../LICENSE.txt) 和 [NOTICE.txt](../NOTICE.txt)。
