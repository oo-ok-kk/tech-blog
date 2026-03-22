---
title: 从 PDF 解析到 Docker 部署：基于 LlamaIndex + DeepSeek 的轻量级 RAG 工程实践
date: 2026-03-22 20:36:00
tags:
  - AIWorkflow
  - RAG
  - DeepSeek
  - LlamaIndex
  - Docker
  - Python
categories: 技术实践
---

在 RAG (检索增强生成) Demo 满天飞的今天，作为一名 **AI Workflow Engineer**，我更关注如何解决实际落地中的三个痛点：**解析精度、环境隔离、隐私安全**。

最近我开源了一个名为 **Private-AI-Knowledge-Base** 的项目，旨在提供一个标准化的私有知识库落地模版。

## 🏗️ 核心架构设计

项目采用 **LlamaIndex** 作为编排框架，结合 **DeepSeek-V3** 的强大推理能力，构建了一个闭环的私有知识流：

* **Data Ingestion**: 集成 `PyMuPDFReader`。在对比了多种解析引擎后，该方案对复杂排版 PDF 的分块准确度显著提升，有效缓解了检索阶段的"幻觉"问题。
* **Vector Storage**: 使用 `ChromaDB` 实现本地持久化，确保数据在容器重启后依然驻留。
* **Hybrid Deployment**: 采用"本地 Embedding (BGE) + 云端 LLM (DeepSeek)"的混合模式，兼顾了本地响应效率与模型推理上限。

## 🛠️ 工程化亮点

1.  **环境沙盒化 (Docker)**：利用 Docker 实现了完整的服务解耦。通过自定义 `Dockerfile` 和 `start.command` 调度脚本，一键拉起 FastAPI 后端与 Streamlit 前端。
2.  **彻底脱敏设计**：全链路采用环境变量管理 API Key，通过 `.gitignore` 严格隔离私有数据与虚拟环境，符合生产级的安全标准。
3.  **极简交互**：基于 Streamlit 构建的 Web UI，支持实时流式响应，提供了秒级的"对话文档"体验。

## 📍 项目地址

欢迎各位开发者交流、指正或贡献代码！

👉 **GitHub**: [oo-ok-kk/Private-AI-Knowledge-Base](https://github.com/oo-ok-kk/Private-AI-Knowledge-Base)

---

#AIWorkflow #RAG #DeepSeek #LlamaIndex #Docker #Python
