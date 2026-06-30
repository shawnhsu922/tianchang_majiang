# Tianchang Mahjong Mini Program

# Documentation Index

Version：1.0.0

Last Updated：2026-06-30

---

# 项目文档导航

本项目采用 **Documentation Driven Development（文档驱动开发）**。

所有开发必须严格按照文档进行。

不得直接修改代码。

---

# 阅读顺序（必须遵守）

第一次进入项目：

README.md

↓

00_Project_Charter.md

↓

01_Product_PRD.md

↓

02_Game_Rules.md

↓

03_Rule_Cases.md

↓

04_State_Machine.md

↓

05_Score_Engine.md

↓

06_Network_Protocol.md

↓

07_Database.md

↓

08_UI_Design.md

↓

09_Development_Guide.md

↓

10_Test_Cases.md

---

# 当前项目状态

| 文档 | 状态 |
|------|------|
| README | ✅ |
| 00_Project_Charter | ✅ |
| 01_Product_PRD | ✅ |
| 02_Game_Rules | ⏳ |
| 03_Rule_Cases | ⏳ |
| 04_State_Machine | ⏳ |
| 05_Score_Engine | ⏳ |
| 06_Network_Protocol | ⏳ |
| 07_Database | ⏳ |
| 08_UI_Design | ⏳ |
| 09_Development_Guide | ⏳ |
| 10_Test_Cases | ⏳ |

---

# 当前开发阶段

Phase 1

项目设计（Project Design）

当前任务：

> 编写 Game Rules（游戏规则）

---

# 文档职责

README

项目介绍。

---

00_Project_Charter

项目目标。

项目原则。

技术路线。

---

01_Product_PRD

产品需求。

页面。

流程。

功能。

---

02_Game_Rules

麻将规则。

所有规则唯一来源（Single Source of Truth）。

---

03_Rule_Cases

规则测试案例。

每一个规则至少一个测试案例。

---

04_State_Machine

游戏状态流转。

每个阶段允许的操作。

---

05_Score_Engine

计分系统。

所有花数计算。

---

06_Network_Protocol

WebSocket。

消息定义。

同步流程。

---

07_Database

数据库设计。

---

08_UI_Design

页面设计。

UI规范。

---

09_Development_Guide

开发规范。

代码规范。

目录规范。

Git规范。

---

10_Test_Cases

集成测试。

性能测试。

异常测试。

---

# AI 开发要求

ChatGPT

负责：

- 产品设计
- 游戏规则
- 技术架构
- 文档维护

Codex

负责：

- 编码实现
- Bug 修复
- 单元测试
- 按照文档开发

任何 AI：

不得修改游戏规则。

不得自行增加功能。

不得删除已有规则。

必须严格遵守 docs 文件夹中的文档。

---

# 下一步

当前完成：

✅ README

✅ Project Charter

✅ Product PRD

下一步：

➡ 编写 Game Rules
