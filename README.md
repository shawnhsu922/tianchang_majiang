# 天长麻将（Tianchang Mahjong）

## 项目状态

Version：v1.0.0

Status：Ready for Development

文档已全部完成，规则已冻结，可交付 Codex 开发。

---

## 文档进度

| 文件 | 说明 | 状态 |
|------|------|------|
| 00_Project_Charter.md | 项目总纲 | ✅ |
| 01_Product_PRD.md | 产品需求 | ✅ |
| 02_Game_Rules.md | 麻将规则 | ✅ |
| 03_Scoring_Table.md | 花数计算（定稿） | ✅ |
| 04_Rule_Cases.md | 规则测试规范 | ✅ |
| 05_Message_Protocol.md | 消息协议 | ✅ |
| 06_State_Machine.md | 游戏状态机 | ✅ |
| 07_Architecture.md | 系统架构 | ✅ |
| 08_Database.md | 数据库设计 | ✅ |

---

## 项目目标

构建一款基于微信小程序的四人实时联网天长麻将游戏。

目标不是商业化运营，而是完整还原天长地区线下麻将玩法，让四位玩家能够稳定、流畅、准确地完成一局麻将。

项目第一原则：

> 规则准确性 > 网络稳定性 > 用户体验 > 界面美观

---

## 技术路线

### Client
- 微信原生小程序
- JavaScript

### Server
- Node.js 18+ LTS
- TypeScript
- WebSocket（ws 库）
- Express（仅用于健康检查）

### Database
- MySQL 8.x

### 部署
- 腾讯云 CVM
- PM2 进程管理

---

## 项目原则

本项目采用文档驱动开发（Documentation Driven Development）。

所有开发必须严格按照 docs 文件夹中的文档执行。

任何代码修改之前，必须先修改对应文档。

---

## 目录结构

tianchang_majiang/
├── docs/          # 项目文档
├── client/        # 微信小程序客户端
├── server/        # Node.js 服务端
└── test/          # 测试工具

详细目录结构见 07_Architecture.md。

---

## 开发流程

需求讨论
↓
更新文档
↓
补充规则案例
↓
Codex 开发
↓
自动测试（04_Rule_Cases.md P0 全部通过）
↓
人工测试
↓
发布版本

---

## 第一阶段目标（V1.0）

- 微信登录
- 创建房间 / 加入房间
- 四人实时联网
- 完整天长麻将规则
- 完整花数计算和结算
- 断线重连 / 自动托管
- 结算页面

以上之外的功能全部延期。

---

## AI 开发要求

1. 不得修改游戏规则
2. 不得修改花数配置
3. 不得自行新增功能
4. 所有花数必须来自 03_Scoring_Table.md
5. 所有状态流转必须遵守 06_State_Machine.md
6. 每个模块完成后必须通过 04_Rule_Cases.md 对应 P0 测试

详细要求见 INDEX.md。

---

Copyright © Tianchang Mahjong Project
