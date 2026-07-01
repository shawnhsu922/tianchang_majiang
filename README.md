# 天长麻将（Tianchang Mahjong）

## 项目状态

**Version**

v1.0.0

**Status**

Rule Freeze（规则冻结）

**Current Progress**

- ✅ 00_Project_Charter
- ✅ 01_Product_PRD
- ✅ 02_Game_Rules
- ✅ 03_Scoring_Table
- ✅ 04_Rule_Cases
- ⏳ 05_Message_Protocol
- ⏳ 06_State_Machine
- ⏳ 07_Data_Model
- ⏳ 08_API
- ⏳ 09_Architecture

---

## 项目目标

构建一套可直接交付 AI（Codex）开发的棋牌游戏服务器规范。

包括：

- 产品需求（PRD）
- 游戏规则（Rules）
- 计分规则（Scoring）
- 测试规范（Rule Cases）
- 消息协议（Message Protocol）
- 状态机（State Machine）
- 数据模型（Data Model）
- API
- 系统架构（Architecture）# Tianchang Mahjong Mini Program

## 项目简介

本项目是一款基于微信小程序开发的 **天长麻将** 游戏。

目标不是开发一款商业麻将游戏，而是完整还原天长地区线下棋牌室麻将玩法，让四位玩家能够稳定、流畅、准确地进行联网对局。

---

## 当前开发阶段

**Version：V1.0（MVP）**

目前正在进行：

- 产品设计
- 游戏规则整理
- PRD 编写

暂未进入正式编码阶段。

---

## 项目原则

本项目采用 **文档驱动开发（Documentation Driven Development）**。

所有开发必须严格按照 `docs` 文件夹中的文档执行。

**任何代码修改之前，必须先修改对应文档。**

---

## 文档目录

| 文件 | 说明 |
|------|------|
| 00_Project_Charter.md | 项目总纲 |
| 01_Product_PRD.md | 产品需求文档 |
| 02_Game_Rules.md | 麻将规则 |
| 03_Rule_Cases.md | 判例手册 |
| 04_State_Machine.md | 游戏状态机 |
| 05_Score_Engine.md | 计分规则 |
| 06_Network_Protocol.md | 网络协议 |
| 07_Database.md | 数据库设计 |
| 08_UI_Design.md | UI 设计 |
| 09_Development_Guide.md | 开发规范 |
| 10_Test_Cases.md | 测试方案 |

---

## 技术路线（暂定）

### Client

- 微信原生小程序

### Server

- Node.js
- TypeScript
- WebSocket

### Database

- MySQL

---

## 开发流程

所有开发必须遵循以下流程：

需求讨论

↓

更新文档

↓

完善规则

↓

补充测试案例

↓

Codex 开发

↓

自动测试

↓

人工测试

↓

发布版本

---

## 第一阶段目标

V1.0 仅完成：

- 微信登录
- 创建房间
- 加入房间
- 四人实时联网
- 天长麻将规则
- 完整计分系统
- 掉线托管
- 自动重连
- 结算页面

除此之外的功能全部延期。

---

## AI 开发要求

本项目使用 AI 协助开发。

开发过程中必须遵守以下原则：

1. 不得修改游戏规则。
2. 不得自行新增功能。
3. 不得简化规则。
4. 必须严格按照 docs 文档开发。
5. 所有新增代码必须保持模块化。
6. 每完成一个模块必须通过测试。

---

Copyright © Tianchang Mahjong Project
