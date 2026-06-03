# LLM Optimization Guidelines

借助 GPT、Claude 等强模型给出优化路径指导文档，用来指导弱模型（或后续会话中的模型）完成开发需求。

## 背景

在 AI 辅助开发中，强模型（如 Claude Opus、GPT-4o 等）适合做架构分析和优化规划，而弱模型（如 Claude Haiku、GPT-4o-mini 等）适合执行具体的编码任务。但由于弱模型对复杂需求理解能力有限，常常出现：

- **只做脚手架**：创建了文件但没有接入真实执行路径
- **重复造轮子**：在同一系统里保留两套做相同事情的代码
- **字符串合约漂移**：改了 key 名但没有同步所有消费者
- **验证流于形式**：没有证据就断定"已完成"
- **fallback 不诚实**：把无效结果包装成正常输出

本文档抽象了一组通用规则，帮助强模型产出结构化的优化指导文档，也帮助弱模型按照明确的标准完成实现。

## 文档结构

- **`CLAUDE.md`** — 核心行为准则，包含 5 大类 17 条规则，覆盖从思考到实现到验证的全流程

## 使用方式

1. **强模型**：阅读项目代码，根据 `CLAUDE.md` 中的规则框架，在 `optimize_guideline/` 目录下产出具体的优化指导文档
2. **弱模型**：按照指导文档的要求执行，并用 `CLAUDE.md` 中的验收标准自查
3. **验证**：用第 5 章的规则检查实现是否真正完成了目标

## 核心规则概览

| # | 规则 | 一句话 |
|---|------|--------|
| 1 | Think Before Coding | 先明确假设，不猜 |
| 2 | Simplicity First | 最少代码解决问题 |
| 3 | Surgical Changes | 只改必须改的 |
| 4 | Goal-Driven Execution | 定义可验证的成功标准 |
| 5.x | Implementation Rules | 13 条落地规则（见 CLAUDE.md） |

## 致谢

本项目基于 [multica-ai/andrej-karpathy-skills](https://github.com/multica-ai/andrej-karpathy-skills) 的 CLAUDE.md 行为准则扩展而来，新增了「优化指导文档落地实施」相关的 13 条规则。

## License

MIT
